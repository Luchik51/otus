# otus

Как организовать создание кубернетес кластера в яндекс клоуд и разместить приложения:
Предварительно надо надо 3 секрета разместить в github:
В настройках репозитория (Settings → Secrets → Actions) добавьте:

YC_SA_KEY – содержимое key.json

YC_FOLDER_ID – ID вашего фолдера в Yandex Cloud

YC_CLOUD_ID – ID облака (можно найти в настройках облака)

Про содержимое key.json оно делаеться так:
нам надо создать сервисный аккаунт с правами достаточными для создания кубернетес кластера (k8s.clusters.agent и vpc.publicAdmin
)
И запустить команду:
yc iam key create --service-account-id $RES_SA_ID --output key.json
где $RES_SA_ID = id этого сервисного аккаунта.

Далее по этой подсказке действовал
https://yandex.cloud/ru/docs/managed-kubernetes/tutorials/new-kubernetes-project#bash_2
там еще надо сначала создать сеть, подсети (вроде автоматом создаються, но это не точно)
 И все это можно в теории можно сделать через github action, но наверное надо этому сервисному аккаунту тогда права админа давать .. что-то очень много изменений надо внести.
Желательно создать 3 аккаунта


Nextcloud values:
https://raw.githubusercontent.com/nextcloud/all-in-one/main/nextcloud-aio-helm-chart/values.yaml

Nextcloud install
helm repo add nextcloud-aio https://nextcloud.github.io/all-in-one/
helm upgrade --install nextcloud-aio nextcloud-aio/nextcloud-aio-helm-chart -f values.yaml


helm search repo nextcloud-aio/nextcloud --versions
helm show values nextcloud-aio/nextcloud-aio-helm-chart  > values-orig.yaml
helm pull nextcloud-aio/nextcloud-aio-helm-chart  --untar
helm list
helm upgrade --install nextcloud-aio ./nextcloud-aio-helm-chart --values values.yaml