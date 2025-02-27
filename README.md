# hello-k8s

## Описание проекта

Мое решение разворачивает веб-приложение в Kubernetes с использованием:

- [Kind](https://kind.sigs.k8s.io/) — для создания локального Kubernetes-кластера с одной control-plane нодой.
- [Ansible](https://www.ansible.com/) — для автоматизированного развертывания кластера, установки Ingress-контроллера и деплоя приложения.
- [Helm](https://helm.sh/) — для удобного управления релизами и развертывания приложения в кластере.

## Предварительные требования

Перед началом работы установите: [Docker](https://www.docker.com/), [Kind](https://kind.sigs.k8s.io/), [Ansible](https://www.ansible.com/), [Helm](https://helm.sh/)

## Запуск и использование

### 1. Клонирование репозитория
```sh
git clone https://github.com/jeorji/hello-k8s.git 
cd hello-k8s
```

### 2. Установка зависимостей Ansible
```sh
ansible-galaxy install -r ansible/requirements.yml
```

### 3. Развертывание кластера
```sh
ansible-playbook ansible/playbook.yml -e cluster_state=present
```

### 4. Проверка доступности приложения

Так как в Kind настроены порты, а Ingress-контроллер работает в режиме `NodePort`, можно проверить доступность приложения:

```sh
curl -H "Host: hello.local" localhost
```

Пример ответа:
```html
<!DOCTYPE html>
<html>
<head>
    <title>Hello, Kubernetes!</title>
</head>
<body>
    <h1>Hello, Kubernetes!</h1>
</body>
</html>
```

### 5. Проверка состояния подов
```sh
kubectl get all -n hello-k8s
```
Пример вывода:
```sh
NAME                                     READY   STATUS    RESTARTS   AGE
pod/hello-k8s-backend-6bb84f4988-s6nqn   1/1     Running   0          26m

NAME                        TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
service/hello-k8s-backend   ClusterIP   10.96.246.39   <none>        80/TCP    26m

NAME                                READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/hello-k8s-backend   1/1     1            1           26m

NAME                                           DESIRED   CURRENT   READY   AGE
replicaset.apps/hello-k8s-backend-6bb84f4988   1         1         1       26m
```

### 6. Просмотр логов пода приложения
```sh
kubectl logs -l app=hello-k8s-backend -n hello-k8s
```
Пример вывода:
```sh
2025/02/27 19:40:50 [notice] 1#1: start worker process 36
10.244.0.6 - - [27/Feb/2025:19:41:17 +0000] "GET / HTTP/1.1" 200 131 "-" "curl/8.4.0" "172.22.0.1"
10.244.0.6 - - [27/Feb/2025:20:05:31 +0000] "GET / HTTP/1.1" 200 131 "-" "curl/8.4.0" "172.22.0.1"
```

### 7. Проверка логов Ingress-контроллера
```sh
kubectl logs -l app.kubernetes.io/name=ingress-nginx -n ingress-nginx
```
Пример вывода:
```sh
I0227 19:41:27.532588      12 event.go:377] Event(v1.ObjectReference{Kind:"Ingress", Namespace:"hello-k8s", Name:"nginx-ingress"}): type: 'Normal' reason: 'Sync' Scheduled for sync
172.22.0.1 - - [27/Feb/2025:20:05:31 +0000] "GET / HTTP/1.1" 200 131 "-" "curl/8.4.0" 74 0.006 [hello-k8s-hello-k8s-backend-80] [] 10.244.0.8:80 131 0.006 200
```

### 8. Проверка состояния Ingress-ресурса
```sh
kubectl get ingress -n hello-k8s
kubectl describe ingress nginx-ingress -n hello-k8s
```

### 9. Удаление кластера
```sh
ansible-playbook ansible/playbook.yml -e cluster_state=absent
```

### 10. PROFIT ???
