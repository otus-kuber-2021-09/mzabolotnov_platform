# mzabolotnov_platform
mzabolotnov Platform repository
<details>
<summary> <b>HW №8 Мониторинг сервиса в кластере k8s (kubernetes-monitoring) </b></summary>

ДЗ выполнено согласно kubernetes-monitoring/HW_monitoring_191527_c0f871-227996-1a026d.pdf

Задание выполняем в кластере minikube

<details>
<summary><b>Установка prometheus-operator</b></summary>

Prometeus-operator устанавливаем с помощью Helm [Документация](https://github.com/prometheus-community/helm-charts)

Установку производим в namespace monitoring

```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
kubectl create ns monitoring
helm upgrade --install prometheus-operator prometheus-community/prometheus-operator -n monitoring
```
</details>

<details>
<summary><b>Установка nginx, nginx-prometheus-exporter</b></summary>

Берем стандартный образ nginx:latest. Подсовываем ему настройки с помощью ConfigMap (описано в deployment.yaml).
Подсаживаем в каждый pod c nginx SideCar-контейнер [nginx-prometheus-exporter](https://github.com/nginxinc/nginx-prometheus-exporter), который собирает метрики с http://localhost:8080/basic_status.
Порт 8080 и /basic_status задаются в настройках nginx. (см. ConfigMap deployment.yamk)

создаем pod c nginx и nginx-prometheus-exporter

```
kubectl apply -f kubernetes-monitoring/deployment.yaml
```
содаем серсис для nginx и nginx-prometheus-exporter

```
kubectl apply -f kubernetes-monitoring/service.yaml
```

создаем servicemonitor согласно [документации](https://github.com/prometheus-operator/prometheus-operator/blob/main/Documentation/user-guides/getting-started.md)

```
kubectl apply -f kubernetes-monitoring/servicemonitor.yaml
```
</details>

<details>
<summary><b>prometheus и grafana</b></summary>

В результате имеем Prometheus
![Screenshot_2021-11-28_20-49-12](https://user-images.githubusercontent.com/80415069/143781706-2e9808e5-f4be-40b5-9206-f111e6643b8e.png)
[Dashbord для Grafana](https://github.com/nginxinc/nginx-prometheus-exporter/tree/master/grafana)
![Screenshot_2021-11-28_20-47-05](https://user-images.githubusercontent.com/80415069/143781729-cae00c93-1137-4b96-9f66-2d79d7a68582.png)

</details>

</details>

<details>
<summary> <b>HW №6 Шаблонизация манифестов Kubernetes. (kubernetes-templating) </b></summary>

- [x] Основное ДЗ

Задание выполнялось согласно kubernetes-templating/hw-4088-53c0bc.pdf

Задание выполнялось в кластере Kubernetes Yandex Cloud

Кластер поднимается с использованием terraform. Исходные файлы расположены в папке kubernetes-templating/terraform
```
teraform init
terraform apply
```

#### <b>Задание 1. Установка ingress-controler в YC</b>
Установку ingress-controler в YC производим согласно инструкции
[Установка ingress-controler](https://cloud.yandex.ru/docs/managed-kubernetes/solutions/ingress-cert-manager)
#### <b>Задание 2. Установка cert-menager</b>
Для установки cert-menager добавляем репозиторий
```
    helm repo add jetstack https://charts.jetstack.io
```
Создаем namespace
```
    kubectl create ns cert-manager
```
Для установки cert-menager согласно ссылке [Установка CRD](https://cloud.yandex.ru/docs/managed-kubernetes/solutions/ingress-cert-manager) устанавливаем некоторые CRD ресурсы
```
    kubectl apply --validate=false -n cert-manager -f https://github.com/jetstack/cert-manager/releases/download/v0.16.1/cert-manager.crds.yaml
```
или
```
    kubectl apply --validate=false -n cert-manager -f kubernetes-templating/cert-manager/cert-manager.crds.yaml
```
Устанавливаем cert-menager из helm-чарта
```
    helm upgrade --install cert-manager jetstack/cert-manager --wait --namespace=cert-manager --version=1.6.0
```
Для корректной работы cert-menager необходимо создание ClusterIssuer (или Issuer).
Манифесты ресурсов:
    kubernetes-templating/cert-manager/cluster_issuer_stage.yaml - для отладки
    kubernetes-templating/cert-manager/cluster_issuer.yaml - окончательный вариант (рабочий)
#### <b>Задание 3. Установка chartmuseum</b>
Создан файл kubernetes-templating/chartmuseum/values.yaml
[Исходник файла values.yaml](https://github.com/helm/charts/blob/master/stable/chartmuseum/values.yaml)
В файл внесены настройки ingress с валидацией tsl-сертификата от Let's Encrypt
Устанавливаем chartmuseum
```
    kubectl create ns chartmuseum
    helm upgrade --install chartmuseum stable/chartmuseum --wait --namespace=chartmuseum --version=2.14.2 -f chartmuseum/values.yaml
```
 В результате
```
    curl https://chartmuseum.51.250.0.41.nip.io
```
должно получиться, что-то типа этого
```
<!DOCTYPE html>
<html>
<head>
<title>Welcome to ChartMuseum!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to ChartMuseum!</h1>
<p>If you see this page, the ChartMuseum web server is successfully installed and
working.</p>

<p>For online documentation and support please refer to the
<a href="https://github.com/helm/chartmuseum">GitHub project</a>.<br/>

<p><em>Thank you for using ChartMuseum.</em></p>
</body>
</html>
```
#### <b>Задание 4. Установка Harbor</b>
Добавляем репозиторий
```
helm repo add harbor https://helm.goharbor.io
```
```
helm upgrade --install harbor harbor/harbor -f kubernetes-templating/harbor/values.yaml
```
в результате harbor доступен по адресу

https://mikhza.twilightparadox.com с валидным сертификатом

#### <b>Задание 5. Создание своего helm-чарта</b>

Используем demo-приложение от Google
https://github.com/GoogleCloudPlatform/microservices-demo

создаем пустой чарт

```
helm create kubernetes-templating/hipster-shop
```
удаляем файл kubernetes-templating/hipster-shop/values.yaml и очищаем папку kubernetes-templating/hipster-shop/templates
Копируем [файл](https://github.com/express42/otus-platform-snippets/blob/master/Module-04/05-Templating/manifests/all-hipster-shop.yaml) в папку kubernetes-templating/hipster-shop/templates

Устанавливаем приложение

```
kubectl create ns hipster-shop
helm upgrade --install hipster-shop kubernetes-templating/hipster-shop --namespace hipster-shop
```
Используя NodePort проверяем работоспособность. Адрес ноды находим используя команду

```
kubectl get nodes -o wide
```
создаем helm-чарт для компонента frontend

```
helm create kubernetes-templating/frontend
```
Переносим из kubernetes-templating/hipster-shop/templates/all-hipster-shop.yaml части касаемые сервиса frontend в отдельные файлы
kubernetes-templating/frontend/templates/deployment.yaml
kubernetes-templating/frontend/templates/service.yaml
Создаем ingress манифест
kubernetes-templating/frontend/templates/ingress.yaml
Переустанавливаем hipster-shop
```
helm upgrade --install hipster-shop kubernetes-templating/hipster-shop --namespace hipster-shop
```
Устанавливаем frontend
```
helm upgrade --install frontend frontend --namespace hipster-shop
```
В итоге имеем
```
$ kubectl get ingress -A
Warning: extensions/v1beta1 Ingress is deprecated in v1.14+, unavailable in v1.22+; use networking.k8s.io/v1 Ingress
NAMESPACE      NAME                      CLASS    HOSTS                            ADDRESS       PORTS     AGE
chartmuseum    chartmuseum-chartmuseum   <none>   chartmuseum.51.250.0.41.nip.io   51.250.0.41   80, 443   26h
default        harbor-ingress            <none>   mikhza.twilightparadox.com       51.250.0.41   80, 443   24h
hipster-shop   frontend                  <none>   shop.51.250.0.41.nip.io          51.250.0.41   80        8h
```
Ура! Приложение работает.

Параметризуем приложение frontend

Меняем файл kubernetes-templating/frontend/values.yaml
```
image:
  tag: v0.1.3

replicas: 1

service:
  port: 80
  targetPort: 8080
  NodePort: 30010
  type: NodePort
```
Меняем файлы

kubernetes-templating/frontend/templates/deployment.yaml
kubernetes-templating/frontend/templates/service.yaml
kubernetes-templating/frontend/templates/ingress.yaml

И вновь запускаем upgrade frontend
```
helm upgrade --install frontend frontend --namespace hipster-shop
```
Архивируем наши чарты

```
helm package frontend
helm package hipster-shop
```
Получаем два архива
kubernetes-templating/frontend-0.1.0.tgz
kubernetes-templating/hipster-shop-0.1.0.tgz
Помещаем наши архивы в репозиторий harbor через web-интерфейс

Добавляем наш репозиторий с двумя чартами
```
helm repo add templating https://mikhza.twilightparadox.com/chartrepo/kubernetes-templating/
```
#### <b>Задание 5. Используем kubecfg для шаблонизации похожих ресурсов</b>
скачиваем kubecfg https://github.com/bitnami/kubecfg/releases
Делаем файл исполняемым и помещаем по пути PATH

Создаем файл kubernetes-templating/kubecfg/services.jsonnet для поднятия двух deploymet и service похожих компонентов
paymentservice и shippingservice.
Скачиваем [готовую библиотеку компонент](https://github.com/bitnami-labs/kube-libsonnet/raw/52ba963ca44f7a4960aeae9ee0fbee44726e481f/kube.libsonnet)
меняем в библиотеке
```
  Deployment(name): $._Object("apps/v1", "Deployment", name) {

```
Выпиливаем компоненты paymentservice и shippingservice из файла kubernetes-templating/hipster-shop/templates/all-hipster-shop.yaml

Делаем upgrade hipster-shop

```
helm upgrade --install hipster-shop ../hipster-shop --namespace hipster-shop --set frontend.service.NodePort=30010
```
И поднимаем отдельно компоненты
```
kubecfg update services.jsonnet -n hipster-shop
```
Заработало со второго раза. Проверяется помещением в корзину товара. Не должно быть ошибок.

#### <b>Задание 6. Используем kustomize</b>

выпиливаем один сервис из общего файла kubernetes-templating/hipster-shop/templates/all-hipster-shop.yaml
Сервис currencyservice
В директории kubernetes-templating/kustomize приведены базовые русурсы kubernetes-templating/kustomize/base
И их кастомизация на окружения dev и prod

</details>

<details>
<summary> <b>HW №5 Security. (kubernetes-security) </b></summary>

- [x] Основное ДЗ
Используется кластер kind

Задание выполнялось согласно kubernetes-security/HW/Kuber_Security_HW-5522-83b386.pdf

## <b>Задание 1</b>
- Создан Service Account bob, ему дана роль admin в рамках всего кластера
- Создан Service Account dave без доступа к кластеру
  Итоговые манифесты, которые решают данную задачу, расположены kubernetes-security/task01

## <b>Задание 2</b>
- Создан Namespace prometheus
- Создан Service Account carol в этом Namespace
- Дан всем Service Account в Namespace prometheus возможность
  делать get, list, watch в отношении Pods всего кластера
  Итоговые манифесты, которые решают данную задачу, расположены kubernetes-security/task02

## <b>Задание 3</b>
- Создать Namespace dev
- Создать Service Account jane в Namespace dev
- Дать jane роль admin в рамках Namespace dev
- Создать Service Account ken в Namespace dev
- Дать ken роль view в рамках Namespace dev
  Итоговые манифесты, которые решают данную задачу, расположены kubernetes-security/task03


</details>

<details>
<summary> <b>HW №4 Volumes, Storages, StatefulSet. (kubernetes-volumes) </b></summary>

- [x] Основное ДЗ

В kind развернут под  с MinIO с использованием манифеста
<pre> 
kubectl apply -f kubernetes-volumes/miniostatefulset.yaml
</pre>
В результате запустится под и создастся StatefulSet minio

Для того, чтобы наш StatefulSet был доступен изнутри кластера,
создадим Headless Service kubernetes-volumes/minio-headlessservice.yaml
</details>
<details>
<summary> <b>HW №3 Сетевая подсистема Kubernetes. (kubernetes-network) </b></summary>

 Задание выполнялось согласно kubernetes-networks/Network_HW-23186-07a062.pdf

- [x] Основное ДЗ

Контейнер с простым web-приложением, работающий на порту 8000 запущен в кластере minikube.
Доступ к приложению осуществляем в трех вариантах
1. ClusterIP
2. LoadBalancer
3. nginx-ingress
Манифест приложения kubernetes-networks/web-deploy.yaml
kubernetes-networks/web-svc-cip.yaml - service ClusterIP для приложения web
kubernetes-networks/web-svc-lb.yaml - service LoadBalancer для приложения web
kubernetes-networks/web-svc-headless.yaml - service для приложения wed через ingress.
При применении 
<pre>
kubectl apply -f kubernetes-networks/web-ingress.yaml
</pre>
Ошибка:
<pre>
Error from server (InternalError): error when creating "web-ingress.yaml": Internal error occurred: failed calling webhook "validate.nginx.ingress.kubernetes.io": an error on the server ("") has prevented the request from succeeding
</pre>
Решение ошибки:
https://stackoverflow.com/questions/61365202/nginx-ingress-service-ingress-nginx-controller-admission-not-found
</details>
<details>
<summary> <b>HW №2 Контролеры. (kubernetes-controllers) </b> </summary>
В ДЗ сделано.
1. Запущен кластер kind. Настройки кластера kind-config.yaml 
2. Создан манифест kubernetes-controller/frontend-replicaset.yaml, с помощью которого приложение запускается в кластере kind
3. Произведена попытка обновлени приложения при изменении образа контейнера. Приложение не обновляется, потому что ReplicaSet не   подходит для этой цели.
4. Создан манифест frontend-deployment.yaml. С использованием Deployment приложение обновилось.
5. Далее создан манифесты paymentservice-deployment-bg.yaml, paymentservice-deployment-reverse.yaml, где параметрами maxSurge, maxUnavailable созданы аналоги blue/green и reverse-rolling update
6. При помощи манифеста kubernetes-controllers/node-exporter-daemonset.yaml node-exporter запущен на всех нодах кластера.
</details>
<details>
<summary> <b>HW №1 Настройка локального окружения. Запуск первого контейнера. Работа с kubectl. (kubernetes-intro)</b> </summary>
Поды восстанавливаются, потому что состояние кластера контролируется Contoller Manager, представляющий собой набор контролеров
Примеры из нашей задачи:
Controlled By:  ReplicaSet/coredns-74ff55c5b
Controlled By:  Node/minikube

В ДЗ сделано.
1. Создан kubernetes-intro/web/Dockerfile. Собран образ nginx, работающим на 8000 порту. Образ пушим на DockerHub
2. Создан манифест kubernetes-intro/web-pod.yaml, с помощью которого приложение запускается в кластере minikube
3. Собран образ из https://github.com/GoogleCloudPlatform/microservices-demo/blob/master/src/frontend/Dockerfile.
4. Запущено приложение Frontend из ук. в п.3 образа в кламтере minikube. Pod имеет статус Error
5. Из командной строки создан манифест kubernetes-intro/frontend-pod.yaml
6. * Манифест исправлен kubernetes-intro/frontend-pod-healthy.yaml. Pod запускается без ошибок.
</details>
