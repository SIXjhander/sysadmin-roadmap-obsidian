# Kubernetes

> Система оркестрации контейнеров

## 🎯 Почему это важно

Kubernetes (K8s) - это индустриальный стандарт для оркестрации контейнеров. Если [[Docker]] решает проблему "как запустить приложение", то Kubernetes решает проблему "как управлять сотнями приложений в продакшене".

## 📋 Необходимые знания перед изучением

**ОБЯЗАТЕЛЬНО нужно знать:**
- [[Docker]] - понимание контейнеров и образов
- [[Linux]] - работа в терминале, процессы
- [[Основы сетей]] - DNS, Load Balancing, сетевые порты
- [[YAML]] - синтаксис манифестов K8s

**Желательно знать:**
- [[Системы мониторинга]] - для observability в K8s
- [[CI-CD]] - для автоматизации деплоя в K8s
- [[Git]] - GitOps практики

## 📊 Уровни владения

### [[Middle SysAdmin]] уровень (базовый K8s)

#### Архитектура Kubernetes
- **Control Plane компоненты**
  - API Server - точка входа в кластер
  - etcd - хранилище состояния кластера
  - Scheduler - размещение подов на нодах
  - Controller Manager - управление контроллерами

- **Worker Node компоненты**
  - kubelet - агент на каждой ноде
  - kube-proxy - сетевой прокси
  - Container runtime (Docker/containerd/CRI-O)

#### Основные объекты K8s

**Pods**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx:1.21
    ports:
    - containerPort: 80
```

**Deployments**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
```

**Services**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
  type: LoadBalancer
```

#### kubectl - основной инструмент
```bash
# Информация о кластере
kubectl cluster-info
kubectl get nodes

# Работа с подами
kubectl get pods
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl exec -it <pod-name> -- /bin/bash

# Работа с deployments
kubectl get deployments
kubectl create deployment nginx --image=nginx
kubectl scale deployment nginx --replicas=5
kubectl rollout status deployment/nginx
kubectl rollout undo deployment/nginx

# Работа с services
kubectl get services
kubectl expose deployment nginx --port=80 --type=LoadBalancer

# Применение манифестов
kubectl apply -f deployment.yaml
kubectl delete -f deployment.yaml
```

#### ConfigMaps и Secrets
```yaml
# ConfigMap
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  database_url: "postgres://localhost:5432"
  log_level: "info"

---
# Secret
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  password: cGFzc3dvcmQxMjM=  # base64 encoded
```

#### Volumes
- emptyDir - временное хранилище
- hostPath - монтирование с ноды (не рекомендуется для прода)
- PersistentVolume и PersistentVolumeClaim
- StorageClass для динамического provisioning

#### Namespaces
```bash
kubectl create namespace development
kubectl get pods -n development
kubectl config set-context --current --namespace=development
```

### [[Senior SysAdmin]] уровень (продвинутый K8s)

#### Production-ready кластеры

**High Availability**
- Multi-master setup
- etcd cluster (3 или 5 нод)
- Load balancer перед API server
- Backup и restore etcd

**Сетевые решения (CNI)**
- Calico - популярный выбор с network policies
- Flannel - простой, для небольших кластеров
- Cilium - продвинутый, с eBPF
- Weave Net
- Требует: глубокого понимания [[Основы сетей]]

**Ingress Controllers**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
spec:
  rules:
  - host: app.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app-service
            port:
              number: 80
```
- nginx-ingress
- Traefik
- HAProxy Ingress
- Связано с: [[Nginx]], [[Load Balancing]]

**Storage**
- Dynamic provisioning
- CSI (Container Storage Interface) drivers
- StatefulSets для stateful приложений
- Backup стратегии

#### Helm - пакетный менеджер
```bash
# Установка Helm chart
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install my-nginx bitnami/nginx

# Создание своего chart
helm create mychart
helm package mychart
helm install myapp ./mychart
```

**Структура Helm chart:**
```
mychart/
  Chart.yaml          # Метаданные chart
  values.yaml         # Значения по умолчанию
  templates/          # Шаблоны K8s манифестов
    deployment.yaml
    service.yaml
    ingress.yaml
```

#### Operators
- Custom Resource Definitions (CRDs)
- Создание собственных операторов
- Operator Framework
- Примеры: Prometheus Operator, MySQL Operator

#### Безопасность

**RBAC (Role-Based Access Control)**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```

**Network Policies**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```

**Security best practices:**
- Pod Security Standards (PSS)
- Не запускать контейнеры как root
- Read-only root filesystem
- Security contexts
- Image scanning
- Secrets encryption at rest
- Связано с: [[Основы информационной безопасности]]

#### Мониторинг и логирование

**Prometheus + Grafana**
- Сбор метрик из кластера
- kube-state-metrics
- node-exporter
- Alertmanager
- Связано с: [[Системы мониторинга]], [[Метрики и алертинг]]

**Logging**
- EFK Stack (Elasticsearch, Fluentd, Kibana)
- Loki (от Grafana Labs)
- Centralized logging
- Связано с: [[Логирование]]

**Distributed Tracing**
- Jaeger
- Zipkin
- OpenTelemetry

#### Service Mesh
- **Istio** - самый популярный
  - Traffic management
  - Security (mTLS)
  - Observability
- **Linkerd** - легковесная альтернатива
- **Consul** (от HashiCorp)

#### Autoscaling

**Horizontal Pod Autoscaler (HPA)**
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

**Cluster Autoscaler**
- Автоматическое масштабирование нод

**Vertical Pod Autoscaler (VPA)**
- Автоматическая настройка ресурсов

#### CI/CD для Kubernetes
- GitOps с ArgoCD или Flux
- Интеграция с [[CI-CD]] пайплайнами
- Blue-green deployments
- Canary deployments
- Rolling updates стратегии

#### Multi-cluster management
- Federation
- Cluster API
- Rancher
- Multi-region deployments

## 🔗 Связанные технологии

### Обязательные для работы с K8s
- [[Docker]] - контейнеризация
- [[Системы мониторинга]] - мониторинг кластера
- [[CI-CD]] - автоматизация деплоя

### Дополняющие технологии
- [[Terraform]] - provisioning кластеров (IaC)
- [[Ansible]] - конфигурирование нод
- [[Load Balancing]] - понимание балансировки нагрузки

### Облачные управляемые K8s
- [[AWS]] - EKS (Elastic Kubernetes Service)
- [[Azure]] - AKS (Azure Kubernetes Service)
- [[Google Cloud Platform]] - GKE (Google Kubernetes Engine)

## 📚 Ресурсы для изучения

### Официальная документация
- kubernetes.io/docs - отличная документация
- Kubernetes The Hard Way - для глубокого понимания

### Книги
- "Kubernetes Up & Running" by Kelsey Hightower
- "Kubernetes in Action" by Marko Luksa
- "Production Kubernetes" by Josh Rosso

### Курсы и сертификации
- CKA (Certified Kubernetes Administrator)
- CKAD (Certified Kubernetes Application Developer)
- CKS (Certified Kubernetes Security Specialist)

### Практика
- Minikube - локальный кластер
- kind (Kubernetes in Docker)
- k3s - легковесный K8s
- Play with Kubernetes - онлайн песочница

## 💡 Best Practices

### Манифесты
1. **Используй labels и selectors** правильно
2. **Всегда указывай resource requests и limits**
3. **Используй readiness и liveness probes**
4. **Не используй latest tag**
5. **Используй namespaces для изоляции**

### Ресурсы
```yaml
resources:
  requests:
    memory: "64Mi"
    cpu: "250m"
  limits:
    memory: "128Mi"
    cpu: "500m"
```

### Health checks
```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 3
  periodSeconds: 3

readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5
```

## 🎯 Практические задачи

### Middle уровень
- [ ] Установить Minikube локально
- [ ] Создать Deployment с 3 репликами
- [ ] Настроить Service для доступа к Deployment
- [ ] Создать ConfigMap и использовать в Pod
- [ ] Настроить Ingress для приложения
- [ ] Выполнить rolling update

### Senior уровень
- [ ] Развернуть HA кластер (3 master ноды)
- [ ] Настроить Prometheus + Grafana
- [ ] Реализовать GitOps с ArgoCD
- [ ] Настроить Network Policies
- [ ] Создать свой Helm chart
- [ ] Настроить HPA и Cluster Autoscaler
- [ ] Реализовать canary deployment
- [ ] Настроить Service Mesh (Istio)

## 🚨 Типичные ошибки

1. **Не устанавливать resource limits** - может привести к noisy neighbor
2. **Не использовать health checks** - K8s не знает о состоянии приложения
3. **Хранить secrets в Git** - используй sealed secrets или vault
4. **Не планировать capacity** - приводит к проблемам с масштабированием
5. **Игнорировать RBAC** - проблемы с безопасностью
6. **Использовать hostPath в проде** - проблемы с persistence

## 📈 Когда использовать Kubernetes

### ✅ Хорошие случаи
- Микросервисная архитектура
- Необходимость автоскейлинга
- Multi-cloud или hybrid cloud
- Высокие требования к availability
- Много разных приложений

### ❌ Избыточно для
- Простые монолитные приложения
- Очень маленькая команда (<3 человек)
- Нет нужды в оркестрации
- Простые статические сайты

Для простых случаев может быть достаточно [[Docker]] + [[Docker Compose]].

---

**Связанные уровни:**
- 📈 [[Middle SysAdmin]] - базовое знакомство с K8s
- 🚀 [[Senior SysAdmin]] - production-ready K8s

**Требует знания:**
- [[Docker]] - обязательно
- [[Linux]] - обязательно
- [[Основы сетей]] - обязательно

#технология #kubernetes #k8s #оркестрация #devops #container-orchestration
