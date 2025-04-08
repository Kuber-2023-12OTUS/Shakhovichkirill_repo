# Подходы к развертыванию и обновлению production-grade кластера

## 1. Основное ДЗ

- Развернуты 4 ВМ в YC для kubernetes и обновления его с помощью kubeadm

- На всех ВМ установлен containderd, kubeadm, kubelet, kubectl. Подготовка ВМ осуществлялась playbook prepare-play.yaml. Версия kubeadm, kubelet, kubectl 1.31.0-1.1.

```
ansible-playbook -bi k8s.ini prepare-play.yaml
```

- Далее на мастер ноде проинициализирован кластер Kubernetes версии 1.31. Затем в качестве сетевого плагина был установлен Flannel. Присоеденины 3 воркер ноды. Данные дейтсвия применялись с использованием playbook kubeadm-play.yaml. Вывод информации о кластере в файле kubernetes_v.1.31.

```
ansible-playbook -bi k8s.ini kubeadm-play.yaml
```

- Выполните обновление master ноды до последней актуальной версии k8s с помощью kubeadm. Версия 1.32. Подготовка ВМ перед обновлением проводилась с помощтб playbook prepare-upgrade-play.yaml.

```
ansible-playbook -bi k8s.ini prepare-upgrade-play.yaml
```
Далее на мастер ноде выполняем проверку плана обновления кластера и его обновление
```
kubeadm upgrade plan
kubeadm upgrade apply v1.32.3
```

Выводим мастер ноду из планирования ноду
```
kubectl drain k8s-m-0 --ignore-daemonsets
```

Далее обновляем kubelet, kubectl, kubeadm на мастер ноде
```
apt-get update && sudo apt-get install -y kubelet='1.32.3-1.1' kubectl='1.32.3-1.1' 'kubeadm=1.32.3-1.1' && apt-mark hold kubelet kubectl kubeadm
```

Перезапуск kubelet на мастер ноде
```
systemctl daemon-reload && systemctl restart kubelet.service
```

Возвращаем мастер ноду
```
kubectl uncordon k8s-m-0
```
Затем обновляем локальную конфигурацию kubelet на воркер нодах
```
kubeadm upgrade node
```

Далее выполняем команды на каждой воркер ноде последовательно.
```
kubectl drain {{ worker }} --ignore-daemonsets #on master

apt-get update && sudo apt-get install -y kubelet='1.32.3-1.1' kubectl='1.32.3-1.1' 'kubeadm=1.32.3-1.1' && apt-mark hold kubelet kubectl kubeadm

systemctl daemon-reload && systemctl restart kubelet.service

kubectl uncordon {{ worker }}
```

- Результат выполнения задания в файле kubernetes_v.1.31

## 2. Задание *

- Создать кластер с использованием kubespray

```
git clone https://github.com/kubernetes-sigs/kubespray.git

cp invemtrory/sample inventory/otus_cluster

ansible-playbook -bi inventory/otus_cluster/inventory.ini cluster.yaml
```

- Вывод информации о кластере в файле kubectl_get_nodes, inventory файл inventory.ini