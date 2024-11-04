############1#############
Создайте манифест, описывающий pod с distroless образом для
создания контейнера, например kyos0109/nginx-distroless и
примените его в кластере. Приложите манифест к результатам ДЗ

```
kubectl apply -f pod_nginx-distroless.yaml
```

############2############
С помощью команды kubectl debug создайте эфемерный
контейнер для отладки этого пода. Отладочный контейнер должен
иметь доступ к пространству имен pid для основного контейнера
пода.

```
kubectl debug -it nginx-distroless --image=dockersec/tcpdump --target nginx-distroless -- sh
```

############3############
Получите доступ к файловой системе отлаживаемого контейнера
из эфемерного. Приложите к результатам ДЗ вывод команды ls –la
для директории /etc/nginx

```
# cd proc/1/root
# ls -la etc/nginx
total 48
drwxr-xr-x 3 root root 4096 Oct  5  2020 .
drwxr-xr-x 1 root root 4096 Nov  1 19:38 ..
drwxr-xr-x 2 root root 4096 Oct  5  2020 conf.d
-rw-r--r-- 1 root root 1007 Apr 21  2020 fastcgi_params
-rw-r--r-- 1 root root 2837 Apr 21  2020 koi-utf
-rw-r--r-- 1 root root 2223 Apr 21  2020 koi-win
-rw-r--r-- 1 root root 5231 Apr 21  2020 mime.types
lrwxrwxrwx 1 root root   22 Apr 21  2020 modules -> /usr/lib/nginx/modules
-rw-r--r-- 1 root root  643 Apr 21  2020 nginx.conf
-rw-r--r-- 1 root root  636 Apr 21  2020 scgi_params
-rw-r--r-- 1 root root  664 Apr 21  2020 uwsgi_params
-rw-r--r-- 1 root root 3610 Apr 21  2020 win-utf
```

############4############
Запустите в отладочном контейнере команду tcpdump -nn -i any -e
port 80 (или другой порт, если у вас приложение на нем)

```
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on any, link-type LINUX_SLL2 (Linux cooked v2), snapshot length 262144 bytes
19:42:36.309868 lo    In  ifindex 1 00:00:00:00:00:00 ethertype IPv4 (0x0800), length 80: 127.0.0.1.60918 > 127.0.0.1.80: Flags [S], seq 2086611236, win 65495, options [mss 65495,sackOK,TS val 36733118 ecr 0,nop,wscale 7], length 0
19:42:36.309880 lo    In  ifindex 1 00:00:00:00:00:00 ethertype IPv4 (0x0800), length 80: 127.0.0.1.80 > 127.0.0.1.60918: Flags [S.], seq 3934930931, ack 2086611237, win 65483, options [mss 65495,sackOK,TS val 36733118 ecr 36733118,nop,wscale 7], length 0
19:42:36.309888 lo    In  ifindex 1 00:00:00:00:00:00 ethertype IPv4 (0x0800), length 72: 127.0.0.1.60918 > 127.0.0.1.80: Flags [.], ack 1, win 512, options [nop,nop,TS val 36733118 ecr 36733118], length 0
19:42:47.628811 lo    In  ifindex 1 00:00:00:00:00:00 ethertype IPv4 (0x0800), length 77: 127.0.0.1.60918 > 127.0.0.1.80: Flags [P.], seq 1:6, ack 1, win 512, options [nop,nop,TS val 36744437 ecr 36733118], length 5: HTTP
19:42:47.628838 lo    In  ifindex 1 00:00:00:00:00:00 ethertype IPv4 (0x0800), length 72: 127.0.0.1.80 > 127.0.0.1.60918: Flags [.], ack 6, win 512, options [nop,nop,TS val 36744437 ecr 36744437], length 0
19:42:47.628994 lo    In  ifindex 1 00:00:00:00:00:00 ethertype IPv4 (0x0800), length 381: 127.0.0.1.80 > 127.0.0.1.60918: Flags [P.], seq 1:310, ack 6, win 512, options [nop,nop,TS val 36744437 ecr 36744437], length 309: HTTP: HTTP/1.1 400 Bad Request
19:42:47.628999 lo    In  ifindex 1 00:00:00:00:00:00 ethertype IPv4 (0x0800), length 72: 127.0.0.1.60918 > 127.0.0.1.80: Flags [.], ack 310, win 510, options [nop,nop,TS val 36744437 ecr 36744437], length 0
19:42:47.629017 lo    In  ifindex 1 00:00:00:00:00:00 ethertype IPv4 (0x0800), length 72: 127.0.0.1.80 > 127.0.0.1.60918: Flags [F.], seq 310, ack 6, win 512, options [nop,nop,TS val 36744437 ecr 36744437], length 0
19:42:47.672654 lo    In  ifindex 1 00:00:00:00:00:00 ethertype IPv4 (0x0800), length 72: 127.0.0.1.60918 > 127.0.0.1.80: Flags [.], ack 311, win 512, options [nop,nop,TS val 36744481 ecr 36744437], length 0
19:42:48.129823 lo    In  ifindex 1 00:00:00:00:00:00 ethertype IPv4 (0x0800), length 72: 127.0.0.1.60918 > 127.0.0.1.80: Flags [F.], seq 6, ack 311, win 512, options [nop,nop,TS val 36744938 ecr 36744437], length 0
19:42:48.129841 lo    In  ifindex 1 00:00:00:00:00:00 ethertype IPv4 (0x0800), length 72: 127.0.0.1.80 > 127.0.0.1.60918: Flags [.], ack 7, win 512, options [nop,nop,TS val 36744938 ecr 36744938], length 0
```

############5############
Выполните несколько сетевых обращений к nginx в отлаживаемом
поде любым удобным вам способом. Убедитесь что tcpdump
отображает сетевые пакеты этих подключений. Приложите
результат работы tcpdump к результатам ДЗ. Вывод в пункте 4.

############6############
С помощью kubectl debug создайте отладочный под для ноды, на
которой запущен ваш под с distroless nginx

```
kubectl debug node/minikube -it --image=ubuntu
```

############7############
Получите доступ к файловой системе ноды, и затем доступ к логам
пода с distrolles nginx. Приложите сами логи, и команду их
получения к результатам ДЗ.

```
root@minikube:/host# cat var/lib/docker/containers/53526ecb81e5e1bdb7b91e70060743b6929809f280eb9b286ddd2bcaeb2b7b3f/53526ecb81e5e1bdb7b91e70060743b6929809f280eb9b286ddd2bcaeb2b7b3f-json.log
{"log":"127.0.0.1 - - [02/Nov/2024:03:42:48 +0800] \"\\xFF\\xF4\\xFF\\xFD\\x06\" 400 157 \"-\" \"-\" \"-\"\n","stream":"stdout","time":"2024-11-01T19:42:48.131071637Z"}
{"log":"127.0.0.1 - - [02/Nov/2024:03:54:14 +0800] \"GET / HTTP/1.1\" 200 612 \"-\" \"curl/7.81.0\" \"-\"\n","stream":"stdout","time":"2024-11-01T19:54:14.283027622Z"}
{"log":"127.0.0.1 - - [02/Nov/2024:04:30:52 +0800] \"GET /aasasas HTTP/1.1\" 404 153 \"-\" \"curl/7.81.0\" \"-\"\n","stream":"stdout","time":"2024-11-01T20:30:52.213248083Z"}
{"log":"2024/11/02 04:30:52 [error] 7#7: *3 open() \"/usr/share/nginx/html/aasasas\" failed (2: No such file or directory), client: 127.0.0.1, server: localhost, request: \"GET /aasasas HTTP/1.1\", host: \"127.0.0.1:8080\"\n","stream":"stderr","time":"2024-11-01T20:30:52.213248505Z"}
```

############*############
Выполните команду strace для корневого процесса nginx в
рассматриваемом ранее поде. Опишите в результатах ДЗ какие
операции необходимо сделать, для успешного выполнения
команды, и приложите ее вывод к результатам ДЗ также.

```
kubectl debug nginx-distroless -it --image=nicolaka/netshoot --target nginx-distroless -- /bin/bash
```
master process
```
nginx-distroless:~# strace -p 1
strace: Process 1 attached
rt_sigsuspend([], 8
```
worker process and curl with other host
```
nginx-distroless:~# strace -p 7
strace: Process 7 attached
epoll_wait(8, [{events=EPOLLIN, data={u32=3659104272, u64=140595118571536}}], 512, -1) = 1
accept4(6, {sa_family=AF_INET, sin_port=htons(48470), sin_addr=inet_addr("127.0.0.1")}, [112 => 16], SOCK_NONBLOCK) = 3
epoll_ctl(8, EPOLL_CTL_ADD, 3, {events=EPOLLIN|EPOLLRDHUP|EPOLLET, data={u32=3659104737, u64=140595118572001}}) = 0
epoll_wait(8, [{events=EPOLLIN, data={u32=3659104737, u64=140595118572001}}], 512, 60000) = 1
recvfrom(3, "GET /aasasas HTTP/1.1\r\nHost: 127"..., 1024, 0, NULL, NULL) = 85
openat(AT_FDCWD, "/usr/share/nginx/html/aasasas", O_RDONLY|O_NONBLOCK) = -1 ENOENT (No such file or directory)
gettid()                                = 7
write(4, "2024/11/02 15:34:20 [error] 7#7:"..., 209) = 209
writev(3, [{iov_base="HTTP/1.1 404 Not Found\r\nServer: "..., iov_len=155}, {iov_base="<html>\r\n<head><title>404 Not Fou"..., iov_len=100}, {iov_base="<hr><center>nginx/1.18.0</center"..., iov_len=53}], 3) = 308
write(5, "127.0.0.1 - - [02/Nov/2024:15:34"..., 97) = 97
setsockopt(3, SOL_TCP, TCP_NODELAY, [1], 4) = 0
epoll_wait(8, [{events=EPOLLIN|EPOLLRDHUP, data={u32=3659104737, u64=140595118572001}}], 512, 65000) = 1
recvfrom(3, "", 1024, 0, NULL, NULL)    = 0
close(3)                                = 0
epoll_wait(8,
```
