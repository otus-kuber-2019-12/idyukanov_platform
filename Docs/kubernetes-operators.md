# Выполнено ДЗ №

 - [x] Основное ДЗ
 - [x] Задание со *

## В процессе сделано:
 - Запустил локальное окружение на kind
```bash
kind create cluster
```
 - Создал и применил его deploy/cr.yml, выпала ошибка, что в kind версия otus.homework/v1 не существует.
```bash
error: unable to recognize "deploy/cr.yml": no matches for kind "MySQL" in version "otus.homework/v1"
```
 - Создал и применил CRD deploy/crd.yml
```bash
kubectl apply -f deploy/crd.yml
customresourcedefinition.apiextensions.k8s.io/mysqls.otus.homework created
```
 - Заново применил deploy/cr.yml.
```
kubectl apply -f deploy/cr.yml
mysql.otus.homework/mysql-instance created
```
 - Добавил параметры validation в deploy/crd.yml (spec), убрал в cr.yml 'usless_data: "useless info"'

 - Задание со *
 - Создал build/mysql-operator.py, добавил шаблоны jinja в build/template/
 - Собрал контейнер build/Dockerfile и поместил его на dockerhub 

 - Добавил в /deploy service-account.yml role.yml role-binding.yml deploy-operator.yml.
    В deploy-operator.yml изменил image на свой: "boroda747/mysql-op:v1" и применил
 - Применил заново deploy/cr.yml (старый удалил)
 - Проверил создание pvc
```
kubectl get pvc
NAME                        STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
backup-mysql-instance-pvc   Bound    pvc-f3502b75-4522-409e-a25f-67c07600a10c   1Gi        RWO            standard       40m
mysql-instance-pvc          Bound    pvc-1e54c7e6-99b0-485b-b4ac-6d7377f4478d   1Gi        RWO            standard       17m
```
 - Заполнил базу otus-database созданного mysql
    kubectl exec -it $MYSQLPOD -- mysql -potuspassword -e "select * from test;" otusdatabase
```
+----+-------------+
| id | name        |
+----+-------------+
|  1 | some data   |
|  2 | some data-2 |
+----+-------------+
```

 - Удалил mysql-instance
 - Проверил pvc и job
```
kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM                               STORAGECLASS   REASON   AGE

kubectl get jobs.batch
NAME                         COMPLETIONS   DURATION   AGE
backup-mysql-instance-job    1/1           2s         22s
```
 - Применил заново deploy/cr.yml
```
kubectl get jobs.batch
NAME                         COMPLETIONS   DURATION   AGE
backup-mysql-instance-job    1/1           3s         2m14s
restore-mysql-instance-job   1/1           34s        111s

kubectl exec -it $MYSQLPOD -- mysql -potuspassword -e "select * from test;" otusdatabase
+----+-------------+
| id | name        |
+----+-------------+
|  1 | some data   |
|  2 | some data-2 |
+----+-------------+
```
## PR checklist:
 - [x] Выставлен label с номером домашнего задания