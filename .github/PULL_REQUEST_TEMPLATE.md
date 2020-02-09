# Выполнено ДЗ №

 - [x] Основное ДЗ
 - [x] Задание со *

## В процессе сделано:
 - Запустил локальное окружение на kind
```bash
kind create cluster
```
 - Применил конфигурацию minio-statefulset.yaml для установки MinIO в кластер kind
```bash
kubectl apply  -f https://raw.githubusercontent.com/express42/otus-platform-snippets/master/Module-02/Kuberenetes-volumes/minio-statefulset.yaml
```
 - Применил конфигурацию minio-headless-service.yaml
```bash
kubectl apply -f https://raw.githubusercontent.com/express42/otus-platformsnippets/master/Module-02/Kuberenetes-volumes/minio-headless-service.yaml
```
 - Проверил создание существование пода, pvc, pv и тд
```bash
kubectl get pvc
NAME           STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
data-minio-0   Bound    pvc-711551a0-a9ed-4a6b-99f7-f33704c84259   10Gi       RWO            standard       6m18s

kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                  STORAGECLASS   REASON   AGE
pvc-711551a0-a9ed-4a6b-99f7-f33704c84259   10Gi       RWO            Delete           Bound    default/data-minio-0   standard                6m22s

kubectl get statefulsets
NAME    READY   AGE
minio   1/1     10m
```
- Задание со *
- Сделал конфигурацию minio-secret.yaml для access_key и secret_key в base64
```bash
# echo -n 'minioaccesskey' | base64
bWluaW9hY2Nlc3NrZXk=
# echo -n 'miniosecretkey' | base64
bWluaW9zZWNyZXRrZXk=
```
- Добавил в конфигурацию minio-statefulset.yaml valueFrom:secretKeyRef, что бы брать секреты
```yaml
        env:
        - name: MINIO_ACCESS_KEY
          valueFrom:
            secretKeyRef:
              name: mysecret
              key: access_key
        - name: MINIO_SECRET_KEY
          valueFrom:
            secretKeyRef:
              name: mysecret
              key: secret_key
```
## PR checklist:
 - [x] Выставлен label с номером домашнего задания