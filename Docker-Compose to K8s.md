
# 🚀 Адаптация Docker Compose для Kubernetes и Helm

## 📘 Оглавление

1. [Введение](#введение)

- [Сопоставление сущностей Docker Compose и Kubernetes](#сопоставление-сущностей-docker-compose-и-kubernetes)

2. [Методы конвертации](#методы-конвертации)

- [Вручную](#вручную)

- [С помощью Kompose](#с-помощью-kompose)

3. [Базовая структура Helm Chart](#базовая-структура-helm-chart)

4. [Перенос переменных и параметров](#перенос-переменных-и-параметров)

5. [Настройка сетевых ресурсов](#настройка-сетевых-ресурсов)

6. [Работа с Volume](#работа-с-volume)

7. [Проверка и деплой](#проверка-и-деплой)

8. [Дополнительно: развитие проекта](#дополнительно-развитие-проекта)

9. [Пример: конвертация Redmine + MySQL](#пример-конвертация-redmine--mysql)

- [Исходный docker-compose.yml](#исходный-docker-composeyml)

- [Kubernetes-манифесты](#kubernetes-манифесты)

- [Helm Chart структура](#helm-chart-структура)

  

---

  

## Введение

Переход от **Compose → Kubernetes → Helm** обычно включает:

- анализ `docker-compose.yml`,

- Конвертацию сущностей в манифесты,

- Параметризацию и упаковку в Helm Chart.

---

### Сопоставление сущностей Docker Compose и Kubernetes

| **Docker Compose**        | **Назначение**                    | **Kubernetes эквивалент**                                     | **Комментарий / Особенности**                                    |
| ------------------------- | --------------------------------- | ------------------------------------------------------------- | ---------------------------------------------------------------- |
| `services:`               | Определение контейнеров           | `Deployment`, `StatefulSet`, `DaemonSet`, `Job`               | Основной блок. Обычно — Deployment; если stateful — StatefulSet. |
| `image:`                  | Используемый контейнер            | `spec.template.spec.containers.image`                         | Прямое соответствие.                                             |
| `build:`                  | Инструкции для сборки образа      | —                                                             | В K8s образ должен быть собран заранее (через CI/CD).            |
| `command:`                | Переопределяет ENTRYPOINT         | `command:`                                                    | Полное соответствие.                                             |
| `entrypoint:`             | Аргументы запуска                 | `args:`                                                       | Аналогично.                                                      |
| `container_name:`         | Имя контейнера                    | —                                                             | Контейнеры в Pod получают уникальные имена автоматически.        |
| `depends_on:`             | Порядок запуска                   | —                                                             | Заменяется readinessProbe, initContainers или StartupProbe.      |
| `ports:`                  | Проброс портов                    | `Service` (ClusterIP, NodePort, LoadBalancer)                 | Преобразуется в Service.                                         |
| `expose:`                 | Внутренние порты                  | `Service` (ClusterIP)                                         | Только внутри кластера.                                          |
| `volumes:`                | Монтирование каталогов            | `PersistentVolume` + `PersistentVolumeClaim` + `volumeMounts` | Полное соответствие.                                             |
| `volumes_from:`           | Наследует volume                  | —                                                             | Можно реализовать через общий PVC.                               |
| `configs:`                | Внешние конфиги                   | `ConfigMap`                                                   | Аналогично.                                                      |
| `secrets:`                | Секреты                           | `Secret`                                                      | Полное соответствие.                                             |
| `environment:`            | Переменные окружения              | `env:` или `envFrom:`                                         | Полное соответствие.                                             |
| `env_file:`               | Файл переменных                   | `ConfigMap` / `Secret`                                        | Импортируется через Helm values.                                 |
| `networks:`               | Логические сети                   | DNS + Namespace                                               | Kubernetes обеспечивает сетевую связность.                       |
| `dns:` / `dns_search:`    | Настройки DNS                     | `dnsConfig`                                                   | Частично поддерживается.                                         |
| `extra_hosts:`            | `/etc/hosts` записи               | `hostAliases`                                                 | Полное соответствие.                                             |
| `restart:`                | Поведение при сбое                | `restartPolicy`                                               | Обычно `Always`.                                                 |
| `logging:`                | Настройка логов                   | Sidecar / FluentBit / Loki / stdout                           | Kubernetes сам логи не управляет.                                |
| `healthcheck:`            | Проверка здоровья                 | `livenessProbe`, `readinessProbe`, `startupProbe`             | Полное соответствие.                                             |
| `deploy.replicas:`        | Количество экземпляров            | `spec.replicas`                                               | Прямое соответствие.                                             |
| `deploy.resources:`       | Лимиты CPU/RAM                    | `resources:`                                                  | Полное соответствие.                                             |
| `deploy.restart_policy:`  | Перезапуск контейнеров            | `restartPolicy`                                               | Частичное соответствие.                                          |
| `deploy.placement:`       | Размещение по узлам               | `nodeSelector`, `affinity`, `taints`                          | Через labels и affinity.                                         |
| `deploy.update_config:`   | Настройка обновлений              | `strategy:` (RollingUpdate)                                   | Kubernetes управляет rollout.                                    |
| `deploy.rollback_config:` | Настройки отката                  | `helm rollback` / `kubectl rollout undo`                      | Аналог.                                                          |
| `deploy.mode:`            | Режим запуска (replicated/global) | `Deployment` / `DaemonSet`                                    | Полное соответствие.                                             |
| `init:`                   | Init-процесс                      | `initContainers`                                              | Более гибкий аналог.                                             |
| `labels:`                 | Метки контейнера                  | `metadata.labels` / `annotations`                             | Полное соответствие.                                             |
| `hostname:`               | Имя хоста                         | `hostname:`                                                   | Полное соответствие.                                             |
| `ipc:` / `pid:` / `uts:`  | Разделяемые namespace             | `shareProcessNamespace`                                       | Частично поддерживается.                                         |
| `cap_add` / `cap_drop`    | Linux capabilities                | `securityContext.capabilities`                                | Полное соответствие.                                             |
| `privileged:`             | Привилегированный режим           | `securityContext.privileged: true`                            | Аналогично.                                                      |
| `user:`                   | UID контейнера                    | `securityContext.runAsUser`                                   | Полное соответствие.                                             |
| `working_dir:`            | Рабочая директория                | `workingDir:`                                                 | Полное соответствие.                                             |
| `tmpfs:`                  | Временная ФС                      | `emptyDir.medium: Memory`                                     | Аналог tmpfs.                                                    |
| `devices:`                | Проброс устройств                 | `hostPath` volumeMount                                        | Требует привилегий.                                              |
| `ulimits:`                | Системные лимиты                  | `securityContext` + `sysctl`                                  | Частично.                                                        |
| `runtime:`                | Docker runtime                    | `runtimeClassName`                                            | Аналог.                                                          |
| `profiles:`               | Наборы сервисов                   | Helm values override / subcharts                              | Через разные values-файлы.                                       |
| `extends:`                | Наследование сервисов             | Helm templates / subcharts                                    | Реализуется шаблонами.                                           |

---

## Методы конвертации

### Вручную:
Шаги:
1. Создать `Deployment` и `Service` для каждого контейнера.
2. Вынести переменные в `ConfigMap` или `Secret`.
3. Добавить `PersistentVolumeClaim` для хранения данных.
4. Настроить `Ingress` при необходимости внешнего доступа.
5. Проверить совместимость портов и сетей.
### С помощью Kompose
Утилита [`kompose`](https://kompose.io/) конвертирует `docker-compose.yml` в Kubernetes YAML.

```bash
kompose convert -f docker-compose.yml -o k8s/
```

После этого надо доработать YAML так как kompose оставляет много своих артефактов и аннотаций и обернуть в Helm Chart.

---

## 🏗️ Базовая структура Helm Chart

```
myapp/
├─ Chart.yaml
├─ values.yaml
├─ templates/
│ ├─ deployment.yaml
│ ├─ service.yaml
│ ├─ configmap.yaml
│ ├─ secret.yaml
│ ├─ pvc.yaml
│ └─ ingress.yaml
└─ README.md
```

### Пример `Chart.yaml`

```yaml
apiVersion: v2
name: redmine
version: 0.1.0
description: Redmine with MySQL
```

---

##  Перенос переменных и параметров

Переменные из `environment` рекомендуется выносить в `values.yaml`.
```yaml
env:
REDMINE_DB_MYSQL: db
REDMINE_DB_PASSWORD: example
REDMINE_SECRET_KEY_BASE: supersecretkey
```
А затем подставлять в helm шаблоне:

```yaml
env:
- name: REDMINE_DB_MYSQL
value: {{ .Values.env.REDMINE_DB_MYSQL }}
- name: REDMINE_DB_PASSWORD
valueFrom:
secretKeyRef:
name: redmine-secret
key: db-password
```
---
## Настройка сетевых ресурсов

| Compose            | Kubernetes            | Пример                         |
| ------------------ | --------------------- | ------------------------------ |
| `ports: 8080:3000` | `Service`             | Создает ClusterIP или NodePort |
| `expose:`          | `containerPort`       | Внутренний порт                |
| `networks:`        | `Ingress` / `Service` | Управление связями             |

---
## Работа с Volume

Compose:

```yaml
volumes:
- ./data:/var/lib/mysql
```

Kubernetes:

```yaml
volumes:
- name: mysql-data
persistentVolumeClaim:
claimName: mysql-pvc
```

```yaml
volumeMounts:
- name: mysql-data
mountPath: /var/lib/mysql
```

---
## Проверка и деплой

```bash
helm lint ./redmine
helm install redmine ./redmine -f values.yaml
kubectl get pods,svc,pvc
```
---
## Дополнительно:
- Добавить `Ingress` для внешнего доступа.
- Настроить `HorizontalPodAutoscaler`.
---

  

## Пример: конвертация Redmine + MySQL

Исходный docker-compose.yml:
```yaml
services:
  redmine:
    image: redmine
    restart: always
    ports:
      - 8080:3000
    environment:
      REDMINE_DB_MYSQL: db
      REDMINE_DB_PASSWORD: example
      REDMINE_SECRET_KEY_BASE: supersecretkey

  db:
    image: mysql:8.0
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: example
      MYSQL_DATABASE: redmine
```

Kubernetes-манифесты:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: root-password
        - name: MYSQL_DATABASE
          value: redmine
        ports:
        - containerPort: 3306
        volumeMounts:
        - name: mysql-data
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-data
        persistentVolumeClaim:
          claimName: mysql-pvc
---
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  ports:
  - port: 3306
  selector:
    app: mysql
```

redmine-deployment.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redmine
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redmine
  template:
    metadata:
      labels:
        app: redmine
    spec:
      containers:
      - name: redmine
        image: redmine
        env:
        - name: REDMINE_DB_MYSQL
          value: mysql
        - name: REDMINE_DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: root-password
        - name: REDMINE_SECRET_KEY_BASE
          valueFrom:
            secretKeyRef:
              name: redmine-secret
              key: secret-key-base
        ports:
        - containerPort: 3000
---
apiVersion: v1
kind: Service
metadata:
  name: redmine
spec:
  type: NodePort
  ports:
  - port: 3000
    targetPort: 3000
    nodePort: 30800
  selector:
    app: redmine
```

Helm Chart структура
```
redmine-chart/
 ├─ Chart.yaml
 ├─ values.yaml
 ├─ templates/
 │   ├─ deployment-redmine.yaml
 │   ├─ deployment-mysql.yaml
 │   ├─ service-redmine.yaml
 │   ├─ service-mysql.yaml
 │   ├─ secret.yaml
 │   └─ pvc.yaml

```

values.yaml:
```yaml
redmine:
  image: redmine
  port: 3000

mysql:
  image: mysql:8.0
  rootPassword: example
  database: redmine
  storage: 1Gi
```
