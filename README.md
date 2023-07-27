# Выполнено ДЗ № 1 (prepare+intro)

- Основное ДЗ
- Задание со *

## В процессе сделано:
1. Подготовка c веткой prepare, установка minikube, выяснение, почему восстанавливаются ресурсы в kube-system: coredns восстанавливается, т.к. он есть в deployment, ресурсы типа apiserver, etcd, scheduler - как понимаю за ними следит сама мастер-нода
2. Запуск web-сервера на nginx через web-pod.yaml на основе своего образа с DockerHub
3. Запуск микросервиса frontend из Hipster Shop. Изначально не стартует, т.к. в манифесте отсутствуют переменные в блоке env для контейнера, видимо они являются обязательными для корректной работы (все из примера манифеста на гитхабе или частично)

## Как запустить проект:
1. Тесты из обновленной методички для ветки prepare прошли успешно
2. Выполняется kubectl apply -f web-pod.yaml и затем проброс порта kubectl port-forward --address 0.0.0.0 pod/web 8000:8000
3. Выполняется kubectl apply -f frontend-pod-healthy.yaml

## Как проверить работоспособность:
1. - 
2. После проброса порта можно зайти по ссылке http://<ip>:8080/index.html
3. После применения обновленного манифества для frontend появляется под frontend в статусе Running


# Выполнено ДЗ № 2 (controllers)

- Основное ДЗ
- Задание со *

## В процессе сделано:
1. Создание frontend-replicaset.yaml. При изменении образа не происходит обновления подов, т.к. репликасет следит в первую очередь за количеством подов, если количество удовлетворяет условиям, то обновления не происходит, для обновления при внесении изменений нужно использовать деплоймент.
2. Созданы образы для paymentservice, paymentservice-replicaset.yaml и paymentservice-deployment.yaml, с деплоймент проверены обновление и откат
3. Созданы paymentservice-deployment-bg.yaml и paymentservice-deployment-reverse.yaml для реализации других стратегий развертывания.
4. Создан frontend-deployment.yaml с добавлением readinessProbe по примеру.
5. Создан node-exporter-daemonset.yaml для развертывания DaemonSet с NodeExporter, реализован запуск на всех нодах, включая мастер-ноды, через добавление tolerations.

## Как запустить проект:
1. Выполняется kubectl apply -f frontend-replicaset.yaml.
2. Выполняется kubectl apply -f paymentservice-deployment.yaml, при изменении тега образа с v0.0.1 на v0.0.2 и повторном запуске команды происходит обновление.
3.1. Выполняется kubectl apply -f paymentservice-deployment-bg.yaml, при изменении тега образа и повторном запуске команды происходит обновление по сценарию: сначала развертывание трех новых подов, потом удаление трех старых.
3.2. Выполняется kubectl apply -f paymentservice-deployment-reverse.yaml, при изменении тега образа и повторном запуске команды происходит обновление по сценарию: сначала удаление одного старого пода, потом запуск одного нового и т.д.
4. Выполняется kubectl apply -f frontend-deployment.yaml.
5. Выполняется kubectl apply -f node-exporter-daemonset.yaml.

## Как проверить работоспособность:
1. Запускаются 3 пода приложения frontend
2. Запускаются 3 пода приложения payment
3. Поды развертываются по заявленным выше сценариям
4. В описании подов после развертывания видно readinessProbe, при нарушении синтаксиса описания пробы и, например, попытке развертывания из образа с другим тегом, развертывание остановится на попытке запуска первого пода, которое не будет завершаться успешно. Проверить можно командой kubectl rollout status deployment/frontend, выполнить откат командой kubectl rollout undo deployment/frontend.
5. Выполнив запуск, можно увидеть, что запущено по одному поду на каждой ноде, при пробросе порта kubectl port-forward <имя любого pod в DaemonSet> 9100:9100 метрики будут в localhost:9100/metrics.


# Выполнено ДЗ № 3 (networks)

- Основное ДЗ
- Задание со * (частично)

## В процессе сделано:
1. Добавлен корректный web-pod.yaml с пробами. Вопрос почему констуркция в пробе с проверкой командой "ps aux | grep my_web_server_process" не имеет смысла - т.к. всегда будет положительный ответ из-за наличия самого процесса grep в результате.
2. Добавлен деплоймент web-deploy.yaml с тремя репликами и пробами.
3. Добавлен web-svc-cip.yaml для создания Service типа ClusterIP
4. Настроен IPVS, установлен MetalLB по методичке и с помощью манифеста metallb-config.yaml, создан Service типа LoadBalancer через web-svc-lb.yaml, добавлен маршрут в ОС.
5. Создан Ingress через nginx-lb.yaml и с использованием MetalLb, приложение web подключено к Ingress через web-svc-headless.yaml, создано правило через web-ingress.yaml.
6. Попробовал сделать Ingress для Dashboard из официального манифеста плюс dashboard.yaml

## Как запустить проект:
1. Выполняется kubectl apply -f web-pod.yaml
2. Выполняется kubectl apply -f web-deploy.yaml
3. Выполняется kubectl apply -f web-svc-cip.yaml
4. IPVS по методичке, MetalLB аналогчино, далее запуск через kubectl apply -f metallb-config.yaml, kubectl apply -f web-svc-lb.yaml
5. Применяются манифесты nginx-lb.yaml, web-svc-headless.yaml, web-ingress.yaml
6. После настройки Dashboard применить dashboard.yaml

## Как проверить работоспособность:
1. Корректный статус пода после запуска.
2. Корректный статус запуска деплоймента и подов.
3. Появляется Service web-svc-cip с Cluster-IP.
4. В kube-ipvs0 появляется ip-адрес, после установки MetalLB корректные статусы всех объектов, после применения web-svc-lb.yaml и добавления маршрута, страница отвечает через EXTERNAL-IP - curl http://172.17.255.1.
5. Service ingress-nginx получил EXTERNAL-IP, приложения отвечает через curl http://172.17.255.2/index.html.
6. Вроде бы отвечает страница к приложению через /dashboard - curl http://172.17.255.2/dashboard.

# Выполнено ДЗ № 4 (volumes)

- Основное ДЗ
- Задание со * 

## В процессе сделано:
1. В кластере kind созданы minio-statefulset.yaml и minio-headlessservice.yaml
2. Добавлен my-secret.yaml с секретом и обновленный с учетом секрета minio-statefulset-secret.yaml
3. Созданы PV типа hostPath, PVC, pod, выполнена запись в файл в смонтированную директорию внутри пода, затем удаление пода, создание второго к тому же PVC и проверка сохранения информации в файле

## Как запустить проект:
1. Выполняется kubectl apply -f minio-statefulset.yaml и kubectl apply -f minio-headlessservice.yaml
2. Применить манифест с секретом и обновленный манифест с statefulset Minio
3. Применить my-pv.yaml, my-pvc.yaml, my-pod.yaml, записать в файл внутри пода текст, удалить под и создать второй my-pod-2.yaml

## Как проверить работоспособность:
1. В кластере есть pv, pvs, statefulset, pod с Minio
2. Корректная работа аналогично пункту 1, плюс появился secret
3. После создания первого пода выполняется kubectl exec -it my-pod -- /bin/bash, затем echo "Hello Kub-ik" > /app/data/data.txt, после удаления первого пода и создания второго внутри него выполняется проверка текста в файле cat /app/data/data.txt
