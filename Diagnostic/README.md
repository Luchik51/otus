## Тут хранится все, что были использовано, для диагностики при создании MVP  

**Yandex Cloud**
Далее по этой подсказке действовал
https://yandex.cloud/ru/docs/managed-kubernetes/tutorials/new-kubernetes-project#bash_2
там еще надо сначала создать сеть, подсети (вроде автоматом создаються, но это не точно)
 И все это можно в теории можно сделать через github action, но наверное надо этому сервисному аккаунту тогда права админа давать .. что-то очень много изменений надо внести.
Желательно создать 3 аккаунта

Квоты: https://yandex.cloud/ru/docs/compute/concepts/limits
Резервация адреса: https://yandex.cloud/ru/docs/vpc/operations/get-static-ip
yc vpc address create --external-ipv4 zone=ru-central1-a
yc vpc address list


Nextcloud values:
https://raw.githubusercontent.com/nextcloud/all-in-one/main/nextcloud-aio-helm-chart/values.yaml

Nextcloud install (не получилось - он простейший, для теста и есть проблема со storageclass - тут нужен NFS с поддержкой RWM)
helm repo add nextcloud-aio https://nextcloud.github.io/all-in-one/
helm upgrade --install nextcloud-aio nextcloud-aio/nextcloud-aio-helm-chart -f values.yaml
# скачанный уже
helm upgrade --install nextcloud-aio ./nextcloud-aio-helm-chart -f values.yaml
