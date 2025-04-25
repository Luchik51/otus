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
# скачанный уже
helm upgrade --install nextcloud-aio ./nextcloud-aio-helm-chart -f values.yaml

---
helm repo add nextcloud https://nextcloud.github.io/helm/
helm repo update
helm search repo nextcloud
helm search repo nextcloud/nextcloud --versions
helm pull nextcloud/nextcloud  --untar
helm show values nextcloud/nextcloud > values-orig-nc.yaml
helm upgrade --install nextcloud nextcloud/nextcloud -f values-nc.yaml -n nextcloud --create-namespace
helm delete nextcloud -n nextcloud
kubectl delete pvc --all -n nextcloud
kubectl delete all --all -n nextcloud
kubectl delete namespace nextcloud
или только Redis
kubectl delete pvc -n nextcloud -l app.kubernetes.io/instance=nextcloud-redis
---

helm search repo nextcloud-aio/nextcloud --versions
helm show values nextcloud-aio/nextcloud-aio-helm-chart  > values-orig.yaml
helm pull nextcloud-aio/nextcloud-aio-helm-chart  --untar
helm list
helm list -n kube-system
helm upgrade --install nextcloud-aio ./nextcloud-aio-helm-chart --values values.yaml
kubectl get endpoints -n nextcloud
helm delete nextcloud-aio
helm delete yandex-s3 -n kube-system

kubectl taint nodes sd-k8s4-cp node-role=infra:NoSchedule-

Yandex
Квоты: https://yandex.cloud/ru/docs/compute/concepts/limits
Резервация адреса: https://yandex.cloud/ru/docs/vpc/operations/get-static-ip
yc vpc address create --external-ipv4 zone=ru-central1-a
yc vpc address list



---- Первый тестовый запуск Nextcloud ----
1. Complete your nextcloud deployment by running:

  NOTE: It may take a few minutes for the LoadBalancer IP to be available.
        Watch the status with: 'kubectl get svc --namespace nextcloud -w nextcloud'

  export APP_HOST=$(kubectl get svc --namespace nextcloud nextcloud --template "{{ range (index .status.loadBalancer.ingress 0) }}{{ . }}{{ end }}")
  export APP_PASSWORD=$(kubectl get secret --namespace nextcloud nextcloud -o jsonpath="{.data.nextcloud-password}" | base64 --decode)

  ## PLEASE UPDATE THE EXTERNAL DATABASE CONNECTION PARAMETERS IN THE FOLLOWING COMMAND AS NEEDED ##

  helm upgrade nextcloud nextcloud/nextcloud \
    --set nextcloud.password=$APP_PASSWORD,nextcloud.host=$APP_HOST,service.type=LoadBalancer,mariadb.enabled=false,externalDatabase.user=nextcloud,externalDatabase.database=nextcloud,externalDatabase.host=YOUR_EXTERNAL_DATABASE_HOST
---- Конец первый тестовый запуск Nextcloud ----

---- Второй (с базой данных) тестовый запуск Nextcloud ----
PS D:\DevOps\git\otus> helm upgrade --install nextcloud nextcloud/nextcloud -f values-nc.yaml -n nextcloud --create-namespace
Error: UPGRADE FAILED: execution error at (nextcloud/charts/mariadb/templates/secrets.yaml:8:21): 
PASSWORDS ERROR: You must provide your current passwords when upgrading the release.
                 Note that even after reinstallation, old credentials may be needed as they may be kept in persistent volume claims.
                 Further information can be obtained at https://docs.bitnami.com/general/how-to/troubleshoot-helm-chart-issues/#credential-errors-while-upgrading-chart-releases

    'auth.rootPassword' must not be empty, please add '--set auth.rootPassword=$MARIADB_ROOT_PASSWORD' to the command. To get the current value:

        export MARIADB_ROOT_PASSWORD=$(kubectl get secret --namespace "nextcloud" nextcloud-mariadb -o jsonpath="{.data.mariadb-root-password}" | base64 -d)

Я указал переменную для базы данных, но можно и как секрет сгенерировать.
---- Конец второй (с базой данных) тестовый запуск Nextcloud ----


---- Первый тестовый запуск Loki Stack ----
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
helm show values grafana/loki-stack > values-orig-grafana-loki-stack.yaml
helm pull grafana/loki-stack  --untar
helm upgrade --install loki grafana/loki-stack -f loki-stack-values.yaml --namespace logging --create-namespace
# helm upgrade --install grafana grafana/grafana -f loki-stack-values.yaml --namespace logging --create-namespace
helm upgrade --install loki grafana/loki-stack  --set grafana.enabled=true,prometheus.enabled=true,prometheus.alertmanager.persistentVolume.enabled=false,prometheus.server.persistentVolume.enabled=false,loki.persistence.enabled=true,loki.persistence.storageClassName=local-path,loki.persistence.size=5Gi --namespace logging --create-namespace
Отсюда: https://techno-tim.github.io/posts/grafana-loki-kubernetes/
Более простой вариат: https://piotrminkowski.com/2023/07/20/logging-in-kubernetes-with-loki/ ## но с примером использования - как смотреть логи. (не понял)
Еще простой вариант с примером: https://cylab.be/blog/197/deploy-loki-on-kubernetes-and-monitor-the-logs-of-your-pods
helm upgrade --install loki grafana/loki-stack --namespace logging --create-namespace --set grafana.enabled=true

Тоже простой. Надо начинать с малого: https://habr.com/ru/articles/766102/
helm upgrade --install loki grafana/loki-stack -f loki-stack-values1.yaml --namespace logging --create-namespace

kubectl -n logging port-forward service/loki-grafana 8080:80
http://localhost:8080


(container_memory_usage_bytes{namespace="logging"} / 1048576)
{{container}}

Но вот алертинг .. как? ни разу не делал.
helm delete loki -n logging
kubectl delete pvc --all -n logging
kubectl get all -n logging
kubectl get pvc -n logging
---- Конец первого тестовый запуск Loki Stack ----

Поставить:
kubectl taint nodes sd-k8s4-cp node-role.kubernetes.io/control-plane:NoSchedule
Удалить:
kubectl taint nodes sd-k8s4-cp node-role.kubernetes.io/control-plane:NoSchedule-