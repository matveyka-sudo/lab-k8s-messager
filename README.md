# Лабораторная работа: Развертывание микросервисного мессенджера в Kubernetes

Данный проект демонстрирует процесс контейнеризации и развертывания облачного приложения (мессенджера) в кластере Kubernetes с использованием методологии GitOps и инструмента Argo CD.

##  Архитектура системы

Приложение разделено на микросервисы, что обеспечивает масштабируемость и отказоустойчивость:
- **Frontend**: Клиентское веб-приложение.
- **BFF (Backend for Frontend)**: Агрегатор запросов для фронтенда.
- **Message Service**: Микросервис управления сообщениями.
- **User Service**: Микросервис управления пользователями.
- **PostgreSQL**: Реляционная база данных.
- **MinIO**: S3-совместимое объектное хранилище (локальная замена внешнего S3 CSI).
- **Goose**: Инструмент автоматизации миграций БД, запускаемый через Kubernetes Jobs.

## Технологический стек
- **Kubernetes** (OrbStack / Docker Desktop)
- **Kustomize**: управление манифестами для разных окружений.
- **Argo CD**: реализация GitOps цикла.
- **Postgres & MinIO**: инфраструктурные сервисы.

## Инструкция по запуску

### 1. Клонирование репозитория
```bash
git clone [https://github.com/matveyka-sudo/lab-k8s-messager.git](https://github.com/matveyka-sudo/lab-k8s-messager.git)
cd lab-k8s-messager
```

### 2. Установка argo cd
``` bash
kubectl create namespace argocd
kubectl apply -n argocd -f [https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml](https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml)
```

### 3. Развёртка через argo cd
Создайте приложение в Argo CD, которое будет отслеживать папку с оверлеем:
``` bash
argocd app create messager-dev \
--repo [https://github.com/matveyka-sudo/lab-k8s-messager.git](https://github.com/matveyka-sudo/lab-k8s-messager.git) \
--path k8s/overlays/dev \
--dest-server [https://kubernetes.default.svc](https://kubernetes.default.svc) \
--dest-namespace default \
--sync-policy auto
```

### 4. Применение конфигурации
Нажмите Sync в интерфейсе Argo CD.
Для корректного применения миграций и обновления Job рекомендуется использовать опции Prune и Replace.

### 5.Проверка работоспособности
После успешной синхронизации (статус Synced / Healthy в Argo CD):
### Проверка, что все компоненты запущены
``` bash
kubectl get pods
```

### Проверка, что миграции выполнены (COMPLETIONS 1/1)
``` bash
kubectl get jobs
```

### Доступ к приложению через браузер
``` bash
kubectl port-forward svc/dev-frontend 8080:80
```


