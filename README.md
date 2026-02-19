# Домашнее задание к занятию «Запуск приложений в K8S»

## ` Дмитрий Климов `

## Задание 1. Создать Deployment и обеспечить доступ к репликам приложения из другого Pod

  1. Создать Deployment приложения, состоящего из двух контейнеров — nginx и multitool. Решить возникшую ошибку.
  2. После запуска увеличить количество реплик работающего приложения до 2.
  3. Продемонстрировать количество подов до и после масштабирования.
  4. Создать Service, который обеспечит доступ до реплик приложений из п.1.
  5. Создать отдельный Pod с приложением multitool и убедиться с помощью curl, что из пода есть доступ до приложений из п.1.

## Ответ:

## Домашнее задание: Запуск приложений в K8S

### Чеклист готовности
Среда настроена (`kubectl` доступен через `microk8s kubectl`).

---

### Шаг 1: Создание Deployment и обеспечение доступа к репликам

Создаем файл `multi-app-deployment.yaml`. Мы используем `multitool` с командой `sleep`, чтобы контейнер не завершился немедленно.

```yaml
# multi-app-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: multi-container-app
  labels:
    app: diagnostic-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: diagnostic-app
  template:
    metadata:
      labels:
        app: diagnostic-app
    spec:
      containers:
      - name: nginx-server
        image: nginx:latest
        ports:
        - containerPort: 80
      - name: diagnostic-tool
        image: praqma/network-multitool:latest
        command: ["/bin/sh", "-c"]
        args: ["while true; do sleep 3600; done"]
```
### A. Развертывание (Начальное состояние: 1 реплика)

```bash
microk8s kubectl apply -f multi-app-deployment.yaml
```
### B. Демонстрация подов до масштабирования

```bash
microk8s kubectl get pods -l app=diagnostic-app
```
### C. Масштабирование до 2 реплик

```bash
microk8s kubectl scale deployment multi-container-app --replicas=2
```
### D. Демонстрация подов после масштабирования

```bash
microk8s kubectl get pods -l app=diagnostic-app
```
### Шаг 2: Создание Service

### ` Создаем файл multi-app-service.yaml. `

```yaml
# multi-app-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: multi-app-service
spec:
  selector:
    app: diagnostic-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP
```
### ` Применяем Service: `

```bash
microk8s kubectl apply -f multi-app-service.yaml
```
### ` Проверяем, что ClusterIP назначен: `

```bash
microk8s kubectl get svc multi-app-service
```
### Шаг 3: Проверка доступа из другого Pod

### ` A. Создание Pod-отладчика `

```bash
microk8s kubectl run debugger-pod --image=praqma/network-multitool:latest --restart=Never -- sleep 3600
```

<img width="1920" height="1080" alt="Снимок экрана (2674)" src="https://github.com/user-attachments/assets/2c0c418a-769b-4ee8-9cc6-9d97ff53dc18" />

### ` B. Проверка связи (Шаг диагностики) `

<img width="1920" height="1080" alt="Снимок экрана (2675)" src="https://github.com/user-attachments/assets/1164606e-c942-4d41-ba28-11ecd6b72169" />

<img width="1920" height="1080" alt="Снимок экрана (2676)" src="https://github.com/user-attachments/assets/f024018f-825d-468f-96e9-d8d8e921326b" />

### 1. Войти в под:

```bash
microk8s kubectl exec -it debugger-pod -- bash
```
```bash
curl http://10.152.183.142:80
```

<img width="1920" height="1080" alt="Снимок экрана (2677)" src="https://github.com/user-attachments/assets/9633becf-5e1f-4226-8829-06194ec55378" />


















