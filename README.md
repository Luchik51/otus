# OTUS. Инфраструктурная платформа на основе Kubernetes. 25 декабря 2024 года — 9 мая 2025
# Проектная работа

## Задание: подготовка MVP инфраструктурной платформы для приложения-примера
### Требовнания. MVP платформы обязательно включает в себя:
Kubernetes (managed или self-hosted)
Мониторинг кластера и приложения с алертами и дашбордами
Централизованное логирование для кластера и приложений 
Обязательно в виде кода в репозитории:
Публичный
CI/CD пайплайн для приложения (Gitlab CI, Github Actions, etc)
Автоматизация создания и настройки кластера
Развертывание и настройки сервисов платформы
Настройки мониторинга, алерты, дашборды

**МVP** — это аббревиатура, которая расшифровывается как Minimum Viable Product (минимально жизнеспособный продукт).  
Это начальная версия продукта, которая содержит только основные функции, необходимые для запуска на рынке и получения обратной связи от пользователей. Цель MVP — быстрее вывести продукт, минимизировать затраты на разработку и определить наиболее важные функции, которые ценят пользователи.


В качестве MPV будет:
managed Kubernetes кластер в Yandex Cloud
Приложение Nextcloud (состоящее из Web-сервера Apache, сервера баз данных MySQL, кэшировние авторизаций Redis)
Мониторинг и логирование Loki-stack (Loki (хранение логов) + prometheus (мониторинг) + grafana (визуализация) + alermanager (уведомления) + promtail (сбор логов))

Как организовать создание кубернетес кластера в яндекс клоуд и разместить приложения:
Предварительно надо надо 3 секрета разместить в github:
В настройках репозитория (Settings → Secrets → Actions) добавьте:

YC_SA_KEY – содержимое key.json

YC_FOLDER_ID – ID вашего фолдера в Yandex Cloud

YC_CLOUD_ID – ID облака (можно найти в настройках облака)

Про содержимое key.json оно делаеться так:
нам надо создать сервисный аккаунт с правами достаточными для создания кубернетес кластера (k8s.clusters.agent и vpc.publicAdmin)
И запустить команду:
```
$RES_SA_ID=ID
yc iam key create --service-account-id $RES_SA_ID --output key.json
#где $RES_SA_ID = id этого сервисного аккаунта.

## Получаем адрес внешний
yc vpc address create --external-ipv4 zone=ru-central1-a
yc vpc address list
```



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
helm show values grafana/grafana > values-orig-grafana.yaml
helm pull grafana/loki-stack  --untar
helm pull grafana/grafana --untar
helm pull grafana/loki --untar
helm pull grafana/promtail --untar
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

Evicted:
kubectl describe pod loki-prometheus-node-exporter-cz6q7 -n logging

Недостаток памяти (MemoryPressure)
Недостаток CPU (CPUPressure)
Дефицит дискового пространства (DiskPressure)
Taints и Tolerations на control-plane узле, из-за которых поды могут быть вытеснены.


Глянуть:
Volumes:
  nextcloud-main:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
Pod was rejected: The node had condition: [DiskPressure].

Удаление всех Evicted подов в namespace nextcloud:
kubectl delete pod -n nextcloud --field-selector=status.phase=Failed


Grafana Dashboards: - sd-k8s4-grafana.patio-minsk.by
kubectl apply -f .\Grafana-dashboards\ConfigMap.yaml
kubectl rollout restart deployment.apps/loki-grafana -n logging
kubectl exec -it loki-grafana-55b975b95f-mcgw5 -n logging -c grafana -- ls /var/lib/grafana/dashboards/default
Не сработало. Нашел вариант:
https://community.grafana.com/t/how-to-upload-dashboards-as-json-files-to-kubernetes-via-helm/59731
Интересное:
To update current configmap use below command.
kubectl replace -f custom-dashboards.yaml -n monitoring
https://github.com/23ewrdtf/notes/blob/master/Grafana/Readme.MD



https://habr.com/ru/articles/772702/ - хорошая стратья по настройке визуализации логов от Loki в Grafana
{app="nextcloud"} |= `GET /avatar/` - авторизации
{app="nextcloud"} |= `GET /logout` - выход
{app="nextcloud"} |= `GET /login?direct=1&user=` - не удачнве попытки авторизации
{app="nextcloud"} |= `207 148` - правка файлов


забрать себе конфиг мап: 
kubectl get configmap loki-grafana-dashboards-default -n logging -o yaml

kubectl get all -n logging
helm delete loki -n logging
kubectl delete pvc --all -n logging
kubectl apply -f .\grafana-dashboards\ConfigMap.yaml
helm upgrade --install loki grafana/loki-stack -f loki-stack-values1.yaml --namespace logging --create-namespace


Alermanager - sd-k8s4-alertmanager.patio-minsk.by
https://1cloud.ru/help/monitoring_system_helps/receive_alerts_in_telegram
Successfully created new token: 2ef978.
https://www.youtube.com/watch?v=nz5xMoY1d6c - все в одном. Полезное видео.
А это репозиторий из видео - https://github.com/digitalstudium/grafana-docker-stack/blob/alertmanager/configs/prometheus/alert_rules.yml - там есть правила, но они для docker - для кубернетеса надо запросы в prometheus писать.
https://habr.com/ru/companies/agima/articles/524654/ - prometheus + redis + alermanager (slack + email) + blackbox (HTTP target статус сайта)
https://medium.com/@bavicnative/alerting-incident-management-in-kubernetes-configuring-alerts-with-alertmanager-f744500c4b9b - примеры правил. Оказывается, настраиваютсья через values prometheus (serverFiles: alerting_rules.yml:)


Бот: Otus_Project_2025
Otus_Project_2025_bot
Done! Congratulations on your new bot. You will find it at t.me/Otus_Project_2025_bot. You can now add a description, about section and profile picture for your bot, see /help for a list of commands. By the way, when you've finished creating your cool bot, ping our Bot Support if you want a better username for it. Just make sure the bot is fully operational before you do this.

Use this token to access the HTTP API:
7953385693:AAHPVau07SF7I-E3iXsL8N3EC5vl9zZpQns
Keep your token secure and store it safely, it can be used by anyone to control your bot.

For a description of the Bot API, see this page: https://core.telegram.org/bots/api
Шаблон: https://api.telegram.org/botINSERT_BOT_ID_HERE/getUpdates
Мой: https://api.telegram.org/bot7953385693:AAHPVau07SF7I-E3iXsL8N3EC5vl9zZpQns/getUpdates

Не смотрел (возможно, полезно): 
https://dev.to/ruanbekker/how-to-setup-alerting-with-loki-kmj
https://dbadbadba.com/blog/finocchiaro-loki

Метрики:
up{job="kubernetes-service-endpoints"} == 0 - почему-то только один сервер.
up{job="kubernetes-nodes"} == 1 - работают
up{job="kubernetes-nodes"} == 0 - не работают
kubectl exec -it loki-prometheus-server-0 -n logging -- curl -s http://localhost:9090/api/v1/rules

Проверка связанности: https://sd-k8s4-prometheus.patio-minsk.by/api/v1/alertmanagers
Проверка наличия правил: https://sd-k8s4-prometheus.patio-minsk.by/api/v1/alerts
kubectl rollout restart statefulset.apps/loki-prometheus-server -n logging
kubectl rollout restart statefulset.apps/loki-alertmanager -n logging
Оказываеться не перезапускались поды = не обновлялась конфигурация.
kubectl logs loki-prometheus-server-0 -c prometheus-server -n logging
kubectl get ConfigMap loki-prometheus-server -n logging -o yaml

# Удаляем Pod с автоматическим пересозданием
kubectl delete pod loki-prometheus-server-0 -n logging --grace-period=0 --force
kubectl delete pod loki-prometheus-server-1 -n logging --grace-period=0 --force
# Полный сброс StatefulSet (если нужно)
kubectl rollout restart statefulset loki-prometheus-server -n logging

Тестовое сообщение (проверил - работает):
curl -X POST \
  "https://api.telegram.org/bot7953385693:AAHPVau07SF7I-E3iXsL8N3EC5vl9zZpQns/sendMessage" \
  -d "chat_id=630538347&text=Test+message"

kubectl delete statefulset.apps/loki-alertmanager -n logging
kubectl delete pvc storage-loki-alertmanager-0 storage-loki-alertmanager-1 -n logging

kubectl delete pod/loki-alertmanager-0 -n logging --grace-period=0 --force

Уведомления через alermanager логов с Loki - пока не понятно. Надо какая-то связь Loki + Alertmanage, как у прометеуса.
Нашел что-то:
# Needed for Alerting: https://grafana.com/docs/loki/latest/rules/
# This is just a simple example, for more details: https://grafana.com/docs/loki/latest/configuration/#ruler_config
#  ruler:
#    storage:
#      type: local
#      local:
#        directory: /rules
#    rule_path: /tmp/scratch
#    alertmanager_url: http://alertmanager.svc.namespace:9093
#    ring:
#      kvstore:
#        store: inmemory
#    enable_api: true
Не делал.