# Выполнено ДЗ №

 - [ ] Основное ДЗ
 - [ ] Задание со *

## В процессе сделано:
 - Под kube-apiserver-minikube описан в /etc/kubernetes/manifests/kube-apiserver.yaml в папке манифестов Kubelet. Kubelet сам управляет этим Подом и поднимает, если он упал.
 - Под coredns скорее всего поднимает scheduler, имеет опцию restartPolicy: Always, так как не нашел его в /etc/kubernetes/manifests/
 - Создан образ app-web с web-сервером (python) отдает папку /app, слушает 8000, uid 1001, помещен в открытый репозиторий boroda747/app-web:v1  
 - Написан манифест web-pod.yaml
 - В манифест web-pod.yaml добавлен initContainer, который тянет index.html для web-pod.yaml. Добавлен volume app
 - Создан образ frontend из предоставленного репозитория, помещен в boroda747/frontend:v1
 - Запуск kubectl run ... frontend, pod не стартуер, ошибка "panic: environment variable "PRODUCT_CATALOG_SERVICE_ADDR" not set", не хватает environments. Написан frontend-pod.yaml
 - Написан frontend-pod-healthy.yaml, с добавлением нужных environments.

## Как запустить проект:
 - Запуск kubectl apply -f web-pod.yaml
 - Проверка kubectl port-forward --address 0.0.0.0 pod/web 8000:8000. 
 - Запуск kubectl run frontend --image boroda747/frontend:v1 --restart=Never или kubectl apply -f frontend-pod.yaml
 - Применение правильного конфига kubectl apply -f frontend-pod-healthy.yaml

## Как проверить работоспособность:
 - Посмотреть kubectl get pods, что контейнер в статусе running
 - Перейти по ссылке http://localhost:8000 или http://ip_host:8000

## PR checklist:
 - [ ] Выставлен label с номером домашнего задания