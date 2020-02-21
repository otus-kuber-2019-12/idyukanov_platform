 - Запустил локальное окружение на minikube
```bash
minikube start --vm-driver=none
```
 - Добавил проверку типа readinessProbe в web-pod.yaml на порт 80 и страницы /index.html
```
readinessProbe:
  httpGet:
    path: /index.html
    port: 80
```
 - Запустил под. Под поднялся в статус Running но без статуса READY
```
NAME   READY   STATUS    RESTARTS   AGE
web    0/1     Running   0          37s

Conditions:
  Type              Status
  Initialized       True 
  Ready             False 
  ContainersReady   False 
  PodScheduled      True 
```
 - Сделал kubectl describe pods web. В ивентах есть варнинг, что проверка readinessProbe не прошла, так как ничего не слушает в контейнере этот порт.
```
Warning  Unhealthy  8s (x7 over 68s)  kubelet, minikube  Readiness probe failed: Get http://172.17.0.6:80/index.html: dial tcp 172.17.0.6:80: connect: connection refused
```
 - Добавил вторую проверку после readinessProbe, тип livenessProbe
```
livenessProbe:
   tcpSocket: { port: 8000 }
```
 - Применение нового манифеста завершается с ошибкой
```
The Pod "web" is invalid: spec: Forbidden: pod updates may not change fields other than `spec.containers[*].image`, `spec.initContainers[*].image`, `spec.activeDeadlineSeconds` or `spec.tolerations` (only additions to existing tolerations)
```
 - Применил конфиг с помощью ключа --force
```
kubectl apply -f kubernetes-intro/web-pod.yaml --force
```
 - Создал новый манифест web-deploy.yaml
 - Удалили старый под kubectl delete pod web
 - Применил новый манифест kubectl apply -f web-deploy.yaml
 - Под в состоянии not running
```
  Type           Status  Reason
  ----           ------  ------
  Available      False   MinimumReplicasUnavailable
```
 - Поправил манифест, изменил readinessProbe Port на 8000 и увеличил количество реплик на 3
 - Применил измененный конфиг kubectl apply -f webdeploy.yaml
 - Проверил Conditions в kubectl describe deployments.apps web
```
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
```
 - Попролбуем разные варианты деплоя
```
    maxUnavailable: 100%
    maxSurge: 0
```
```
    maxUnavailable: 0
    maxSurge: 100%
```
```
    maxUnavailable: 1
    maxSurge: 0
```

 - Создан манифест web-svc-cip.yaml
```
[root@id-otus-101 kubernetes-networks]# kubectl get service
NAME          TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
web-svc-cip   ClusterIP   10.96.132.158   <none>        80/TCP    11s
```
 - Сделал curl http://10.96.132.158/index.html получаем
```
[root@id-otus-101 kubernetes-networks]# curl http://10.96.132.158/index.html
<html>
<head/>
<body>
<!-- IMAGE BEGINS HERE -->
<font size="-3">
<pre><font color=white>0111010....
```
 - Сделал ping 10.96.132.158, пинга нет
 - Сделал arp -an
 - Сделал ip addr show
 - Нигде нету ClusterIP
 - Сделал iptables --list -nv -t nat, ClusterIP есть в Chain KUBE-SERVICES
```
[root@id-otus-101 kubernetes-networks]# iptables --list -nv -t nat
Chain KUBE-SERVICES (2 references)
 pkts bytes target     prot opt in     out     source               destination
    1    60 KUBE-SVC-WKCOG6KH24K26XRJ  tcp  --  *      *       0.0.0.0/0            10.96.132.158        /* default/web-svc-cip: cluster IP */ tcp dpt:80
```
 - Включим kube-proxy мод ipvs
```
kubectl --namespace kube-system edit configmap/kube-proxy

...
metricsBindAddress: 127.0.0.1:10249
mode: "ipvs"
nodePortAddresses: null
...
```
 - Удалил под kube-proxy, что бы он пересоздался
```
kubectl delete pods kube-proxy-jh4lc -n kube-system
```
 - Очистил iptable с помощью файла iptables.cleanup 
```
*nat
-A POSTROUTING -s 172.17.0.0/16 ! -o docker0 -j MASQUERADE
COMMIT
*filter
COMMIT
*mangle
COMMIT
```
 - iptables-restore /tmp/iptables.cleanup
 - Поставил ipvsadm и сделал ipvsadm --list -n
```
TCP  10.96.132.158:80 rr
  -> 172.17.0.6:8000              Masq    1      0          0         
  -> 172.17.0.7:8000              Masq    1      0          0         
  -> 172.17.0.8:8000              Masq    1      0          0 
```
 - Теперь ping есть
```
ping 10.96.132.158
PING 10.96.132.158 (10.96.132.158) 56(84) bytes of data.
64 bytes from 10.96.132.158: icmp_seq=1 ttl=64 time=0.041 ms
```
 - Сделаем ip addr show
```
kube-ipvs0: <BROADCAST,NOARP> mtu 1500 qdisc noop state DOWN group default 
    link/ether ae:d0:2a:c4:82:3b brd ff:ff:ff:ff:ff:ff
    inet 10.96.132.158/32 brd 10.96.132.158 scope global kube-ipvs0
       valid_lft forever preferred_lft forever
```

 - Установил MetalLB 
```
kubectl apply -f https://raw.githubusercontent.com/google/metallb/v0.8.0/manifests/metallb.yaml
```
 - Создал конфиг metallb-config.yaml для конфигурации metallb и применил его.
 - Создан конфиг web-svc-lb.yaml из web-svc-cip.yaml, изменен type: LoadBalancer и name: web-svc-lb и применил.
```
[root@id-otus-101 kubernetes-networks]# kubectl apply -f web-svc-lb.yaml 
service/web-svc-lb created
[root@id-otus-101 kubernetes-networks]# kubectl --namespace metallb-system logs controller-7845b997db-hj7wp | grep "service.go"
{"caller":"service.go:33","event":"clearAssignment","msg":"not a LoadBalancer","reason":"notLoadBalancer","service":"default/web-svc-cip","ts":"2020-01-26T16:02:35.600276162Z"}
{"caller":"service.go:33","event":"clearAssignment","msg":"not a LoadBalancer","reason":"notLoadBalancer","service":"default/kubernetes","ts":"2020-01-26T16:02:35.600485579Z"}
{"caller":"service.go:33","event":"clearAssignment","msg":"not a LoadBalancer","reason":"notLoadBalancer","service":"kube-system/kube-dns","ts":"2020-01-26T16:02:35.600670737Z"}
{"caller":"service.go:33","event":"clearAssignment","msg":"not a LoadBalancer","reason":"notLoadBalancer","service":"kubernetes-dashboard/kubernetes-dashboard","ts":"2020-01-26T16:02:35.600867418Z"}
{"caller":"service.go:33","event":"clearAssignment","msg":"not a LoadBalancer","reason":"notLoadBalancer","service":"kubernetes-dashboard/dashboard-metrics-scraper","ts":"2020-01-26T16:02:35.601318602Z"}
{"caller":"service.go:98","event":"ipAllocated","ip":"172.17.255.1","msg":"IP address assigned by controller","service":"default/web-svc-lb","ts":"2020-01-26T16:33:25.89838598Z"}
```
 - Посмотрим kubectl describe svc web-svc-lb
```
Name:                     web-svc-lb
Namespace:                default
Labels:                   <none>
Annotations:              kubectl.kubernetes.io/last-applied-configuration:
                            {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"name":"web-svc-lb","namespace":"default"},"spec":{"ports":[{"port":80,"p...
Selector:                 app=web
Type:                     LoadBalancer
IP:                       10.96.38.80
LoadBalancer Ingress:     172.17.255.1
Port:                     <unset>  80/TCP
TargetPort:               8000/TCP
NodePort:                 <unset>  32441/TCP
Endpoints:                172.17.0.6:8000,172.17.0.7:8000,172.17.0.8:8000
Session Affinity:         None
External Traffic Policy:  Cluster
Events:
  Type    Reason       Age    From                Message
  ----    ------       ----   ----                -------
  Normal  IPAllocated  5m32s  metallb-controller  Assigned IP "172.17.255.1"
```
 - Так как у меня minikube запущен на локальном докере --vm-driver=none, прокидывать маршрут мне не пришлось
 - Проверим curl http://172.17.255.1/index.html
 ```
[root@id-otus-101 kubernetes-networks]# curl http://172.17.255.1/index.html
<html>
<head/>
<body>
<!-- IMAGE BEGINS HERE -->
<font size="-3">
<pre><font color=white>0111010011111011110........
 ```
  - Доп. задание со *. Сделал два конфига coredns-svc-lb-tcp.yaml и coredns-svc-lb-udp.yaml
  - Добавил в "annotations:metallb.universe.tf/allow-shared-ip: dns" для совместного использования одного ip
  - Конфиги отлючаются портами TCP и UDP.
  - Применил конфиг
```
[root@id-otus-101 kubernetes-networks]# kubectl describe service --namespace kube-system coredns-svc-lb-tcp 
Name:                     coredns-svc-lb-tcp
Namespace:                kube-system
Labels:                   <none>
Annotations:              kubectl.kubernetes.io/last-applied-configuration:
                            {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{"metallb.universe.tf/allow-shared-ip":"dns"},"name":"coredns-svc-lb-tcp","n...
                          metallb.universe.tf/allow-shared-ip: dns
Selector:                 k8s-app=kube-dns
Type:                     LoadBalancer
IP:                       10.96.9.125
IP:                       172.17.255.4
LoadBalancer Ingress:     172.17.255.4
Port:                     <unset>  53/TCP
TargetPort:               53/TCP
NodePort:                 <unset>  30123/TCP
Endpoints:                172.17.0.4:53,172.17.0.5:53
Session Affinity:         None
External Traffic Policy:  Cluster
Events:
  Type    Reason       Age   From                Message
  ----    ------       ----  ----                -------
  Normal  IPAllocated  24m   metallb-controller  Assigned IP "172.17.255.4"
```
```
[root@id-otus-101 kubernetes-networks]# kubectl describe service --namespace kube-system coredns-svc-lb-udp 
Name:                     coredns-svc-lb-udp
Namespace:                kube-system
Labels:                   <none>
Annotations:              kubectl.kubernetes.io/last-applied-configuration:
                            {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{"metallb.universe.tf/allow-shared-ip":"dns"},"name":"coredns-svc-lb-udp","n...
                          metallb.universe.tf/allow-shared-ip: dns
Selector:                 k8s-app=kube-dns
Type:                     LoadBalancer
IP:                       10.96.172.134
IP:                       172.17.255.4
LoadBalancer Ingress:     172.17.255.4
Port:                     <unset>  53/UDP
TargetPort:               53/UDP
NodePort:                 <unset>  31027/UDP
Endpoints:                172.17.0.4:53,172.17.0.5:53
Session Affinity:         None
External Traffic Policy:  Cluster
Events:
  Type    Reason       Age   From                Message
  ----    ------       ----  ----                -------
  Normal  IPAllocated  32m   metallb-controller  Assigned IP "172.17.255.4"
```

 - Проверил с помощью nslookup
```
[root@id-otus-101 kubernetes-networks]# nslookup web-svc-lb.default.svc.cluster.local 172.17.255.4
Server:         172.17.255.4
Address:        172.17.255.4#53

Name:   web-svc-lb.default.svc.cluster.local
Address: 10.96.38.80
```

 - Устанавлием ингресс 
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/mandatory.yaml
```
 - Сделал конфиг для ingress nginx-lb.yaml и применил его.
 - Сервис получил IP 172.17.255.2 
 - Сделал curl
```
[root@id-otus-101 kubernetes-networks]# curl http://172.17.255.2
<html>
<head><title>404 Not Found</title></head>
<body>
<center><h1>404 Not Found</h1></center>
<hr><center>nginx/1.17.7</center>
</body>
</html>
```
 - Создан конфиг web-svc-headless.yaml, добавлен clusterIP: None, изменено имя name: web-svc и применил его
 - Убедимся, что сервис web-svc не получил ClusterIP
```
[root@id-otus-101 kubernetes-networks]# kubectl get service -o wide
NAME          TYPE           CLUSTER-IP      EXTERNAL-IP    PORT(S)        AGE     SELECTOR
web-svc       ClusterIP      None            <none>         80/TCP         7m48s   app=web
```
 - Создан web-ingress.yaml и применен.

## PR checklist:
 - [x] Выставлен label с номером домашнего задания