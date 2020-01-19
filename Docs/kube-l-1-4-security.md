## В процессе сделано:
 - Запущено окружение с помощью локального кластера kind
 - Создан манифест и применен task01/01-u-adm-bob.yaml с пользователем bob и применена роль cluster-admin.
 - Создан манифест и применен task01/02-u-dave.yaml с пользователем dave без роли.
 - Создан манифест и применен task02/01-ns-prometheus.yaml для создания namespace с именем prometheus.
 - Создан манифест и применен task02/02-s-carol.yaml с пользователем carol и в этом же yaml создаем роль cluster-watch с разрешениями на get, list, watch. Применяем роль к carol.
 - Создан манифест и применен task03/01-ns-dev.yaml для создания namespace с именем dev.
 - Создан манифест и применен task03/02-s-jane.yaml с пользователем jane и применена роль admin.
 - Создан манифест и применен task03/03-s-ken.yaml с пользователем ken и применена роль view.

## Как запустить проект:
 - kubectl apply -f ...

## Как проверить работоспособность:
 - Создать файл конфигурации kubectl в своей home ~/.kube/name.yaml, изменить user и key: на bob, dave, carol, jane или ken.
 - Подключить файл конфигурации kubectl config use-context name.yaml
 - Попробовать попробовать сделать kubectl get *, exec * и тд
 - Проверить неймспейсы, выполнить: kubectl get ns dev -o json