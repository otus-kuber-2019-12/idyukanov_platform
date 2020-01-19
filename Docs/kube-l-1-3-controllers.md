## В процессе сделано:
 - Запущено окружение с помощью локального кластера kind
 - Создан манифест frontend-replicaset.yaml, добавлены образ с предыдущего задания и env
 - При применении манифеста завершилось ошибкой
```bash
error: error validating "kubernetes-controllers/frontend-replicaset.yaml": error validating data: ValidationError(ReplicaSet.spec): missing required field "selector" in io.k8s.api.apps.v1.ReplicaSetSpec; if you choose to ignore these errors, turn validation off with --validate=false
```
 - В манифест frontend-replicaset.yaml добавлена секция
```yaml
selector:
    matchLabels:
    app: frontend
```
 - Манифест frontend-replicaset.yaml применен
 - Увелечино количество реплик frontend с помощью команды
```bash
kubectl scale replicaset frontend --replicas=3
[idyukanov@id-otus-101 idyukanov_platform]$ kubectl get rs frontend
NAME       DESIRED   CURRENT   READY   AGE
frontend   3         3         3       177m
```
 - Проверил, что поды восстанавливаются после их удаления
 - Применил повторно манифест frontend-replicaset.yaml, кол-во реплик уменьшилось до 1
 - Изменил replicas: в frontend-replicaset.yaml на 3, применил, кол-во реплик изменилось до 3
 - Обновили версию образа в frontend-replicaset.yaml до boroda747/frontend:v0.0.2, применил, поды не обновились. Осталась старые версии с boroda747/frontend:v0.0.1.
```bash
kubectl get pods -l app=frontend -o=jsonpath='{.items[0:3].spec.containers[0].image}'
boroda747/frontend:v0.0.1 boroda747/frontend:v0.0.1 boroda747/frontend:v0.0.1
```
 - ReplicaSet не умеет рестартовать поды при измении манифеста, она следит только за их количеством, нужно использовать deployment.
 - Сделаны манифесты paymentservice-replicaset.yaml и paymentservice-deployment.yaml.
 - Обновил образ в paymentservice-deployment.yaml на boroda747/paymentservice:v0.0.2. Применил манифест. Все поды обновились на новый образ.
 - Сделал откат на предыдущую версию с помощью
```bash
kubectl rollout undo deployment paymentservice --to-revision=1
```
 - Сделан манифест paymentservice-deployment-bg.yaml со сценарием резвертывания blue-green
 - Сделан манифест paymentservice-deployment-reverse.yaml со сценарием резвертывания Rolling Update
 - Добавлен манифест frontend-deployment.yaml с блоком и применен
```yaml
        readinessProbe:
        initialDelaySeconds: 10
        httpGet:
            path: "/_healthz"
            port: 8080
            httpHeaders:
            - name: "Cookie"
            value: "shop_session-id=x-readiness-probe"
```
 - В подах frontend появилось описание Readiness
```bash
Readiness:      http-get http://:8080/_healthz delay=10s timeout=1s period=10s #success=1 #failure=3
```
 - Изменил url на http://:8080/_health, под не поднялся из за ошибки probe
```bash
Warning  Unhealthy  4s (x3 over 24s)  kubelet, kind-worker  Readiness probe failed: HTTP probe failed with statuscode: 404
```
    Вернул старый url
 - Добавлен манфиест node-exporter-daemonset.yaml и применен
 - Проверил с помощью kubectl port-forward node-exporter 9100:9100 наличие метрик
 - Поды node-exporter развернулись только на воркер нодах. Добавил в node-exporter-daemonset.yaml секцию
```yaml
tolerations:
- key: node-role.kubernetes.io/master
    effect: NoSchedule
```
    Что бы планировщик смог планировать поды на мастере
 - Проверил, что все поды развернулись на всех нодах
```bash
[idyukanov@id-otus-101 idyukanov_platform]$ kubectl get pods -l app=node-exporter -o wide
NAME                  READY   STATUS    RESTARTS   AGE   IP            NODE                  NOMINATED NODE   READINESS GATES
node-exporter-4m8qx   1/1     Running   0          84m   10.244.5.16   kind-worker2          <none>           <none>
node-exporter-6hr5s   1/1     Running   0          85m   10.244.4.17   kind-worker3          <none>           <none>
node-exporter-ccsbs   1/1     Running   0          85m   10.244.0.4    kind-control-plane    <none>           <none>
node-exporter-knwhf   1/1     Running   0          85m   10.244.3.15   kind-worker           <none>           <none>
node-exporter-m2d47   1/1     Running   0          85m   10.244.1.2    kind-control-plane2   <none>           <none>
node-exporter-z2n9l   1/1     Running   0          85m   10.244.2.2    kind-control-plane3   <none>           <none>
```


## Как запустить проект:
 - kubectl scale replicaset frontend --replicas=3

## Как проверить работоспособность:
 - Удаление всех подов frontend для проверки 
```bash
kubectl delete pods -l app=frontend | kubectl get pods -l app=frontend -w
```
 - Проверка образа в rs kubectl get replicaset frontend -o=jsonpath='{.spec.template.spec.containers[0].image}'
 - Проверка образа в контейнерах kubectl get pods -l app=frontend -o=jsonpath='{.items[0:3].spec.containers[0].image}'
 - История версий kubectl rollout history deployment paymentservice
 - Откат kubectl rollout undo deployment paymentservice --to-revision=1