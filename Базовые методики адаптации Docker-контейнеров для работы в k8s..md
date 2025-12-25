# Базовые методики адаптации Docker-контейнеров для работы в k8s.
# Оглавление

 [[Базовые методики адаптации Docker-контейнеров для работы в k8s.#Базовые методики адаптации Docker-контейнеров для работы в k8s.|Базовые методики адаптации Docker-контейнеров для работы в k8s.]]
	 [[Базовые методики адаптации Docker-контейнеров для работы в k8s.##1. Адаптация контейнерного образа (Dockerfile-level)|1. Адаптация контейнерного образа (Dockerfile-level)]]
		 [[Базовые методики адаптации Docker-контейнеров для работы в k8s.###1.1. Один процесс — один контейнер|1.1. Один процесс — один контейнер]]
		 [[Базовые методики адаптации Docker-контейнеров для работы в k8s.###1.2. Корректная работа PID 1|1.2. Корректная работа PID 1]]
		 [[Базовые методики адаптации Docker-контейнеров для работы в k8s.###1.3. Отказ от root-пользователя|1.3. Отказ от root-пользователя]]
		 [[Базовые методики адаптации Docker-контейнеров для работы в k8s.###1.4 Для надежного развертывания и отката в Kubernetes используется концепция неизменяемых развертываний.|1.4 Для надежного развертывания и отката в Kubernetes используется концепция неизменяемых развертываний.]]
	[[Базовые методики адаптации Docker-контейнеров для работы в k8s.##2. Конфигурация и переменные окружения|2. Конфигурация и переменные окружения]]
		 [[Базовые методики адаптации Docker-контейнеров для работы в k8s.###2.1. Декларативная конфигурация|2.1. Декларативная конфигурация]]
		 [[Базовые методики адаптации Docker-контейнеров для работы в k8s.###2.2. Использование ConfigMap и Secret|2.2. Использование ConfigMap и Secret]]
	[[Базовые методики адаптации Docker-контейнеров для работы в k8s.##3. Работа с файловой системой и данными|3. Работа с файловой системой и данными]]
		[[Базовые методики адаптации Docker-контейнеров для работы в k8s.###3.1. Эфемерность контейнера|3.1. Эфемерность контейнера]]
		[[Базовые методики адаптации Docker-контейнеров для работы в k8s.###3.2. Volume-ориентированная архитектура|3.2. Volume-ориентированная архитектура]]
	[[Базовые методики адаптации Docker-контейнеров для работы в k8s.##4. Сетевое взаимодействие|4. Сетевое взаимодействие]]
		[[Базовые методики адаптации Docker-контейнеров для работы в k8s.###4.1. Отказ от жёсткой привязки к IP|4.1. Отказ от жёсткой привязки к IP]]
		[[Базовые методики адаптации Docker-контейнеров для работы в k8s.###4.2. Один порт — один сервис|4.2. Один порт — один сервис]]
	[[Базовые методики адаптации Docker-контейнеров для работы в k8s.##5. Health-checks и жизненный цикл|5. Health-checks и жизненный цикл]]
		[[Базовые методики адаптации Docker-контейнеров для работы в k8s.###5.1. Readiness и Liveness probes|5.1. Readiness и Liveness probes]]
		[[Базовые методики адаптации Docker-контейнеров для работы в k8s.###5.2. Startup-поведение|5.2. Startup-поведение]]
	[[Базовые методики адаптации Docker-контейнеров для работы в k8s.##6. Управление ресурсами|6. Управление ресурсами]]
		[[Базовые методики адаптации Docker-контейнеров для работы в k8s.###6.1. Предсказуемое потребление ресурсов|6.1. Предсказуемое потребление ресурсов]]
		[[Базовые методики адаптации Docker-контейнеров для работы в k8s.###6.2. Реакция на OOM и throttling|6.2. Реакция на OOM и throttling]]
	[[Базовые методики адаптации Docker-контейнеров для работы в k8s.##7. Логирование и наблюдаемость|7. Логирование и наблюдаемость]]
		[[Базовые методики адаптации Docker-контейнеров для работы в k8s.###7.1. Логи в stdout/stderr|7.1. Логи в stdout/stderr]]
		[[Базовые методики адаптации Docker-контейнеров для работы в k8s.###7.2. Метрики и трассировка|7.2. Метрики и трассировка]]
	[[Базовые методики адаптации Docker-контейнеров для работы в k8s.##8. Kubernetes-native подход|8. Kubernetes-native подход]]
		[[Базовые методики адаптации Docker-контейнеров для работы в k8s.###8.1. Stateless-архитектура|8.1. Stateless-архитектура]]
		[[Базовые методики адаптации Docker-контейнеров для работы в k8s.###8.2. Разделение ответственности|8.2. Разделение ответственности]]
	[[Базовые методики адаптации Docker-контейнеров для работы в k8s.##9. Методики переноса существующих Docker-контейнеров|9. Методики переноса существующих Docker-контейнеров]]
[[Базовые методики адаптации Docker-контейнеров для работы в k8s.#Сопоставление сущностей Docker Compose и Kubernetes|Сопоставление сущностей Docker Compose и Kubernetes]]
[[Базовые методики адаптации Docker-контейнеров для работы в k8s.#Адаптация Docker Compose (Redmine + MySQL) для Kubernetes|Адаптация Docker Compose (Redmine + MySQL) для Kubernetes]]
	[[Базовые методики адаптации Docker-контейнеров для работы в k8s.##1. Исходный docker-compose.yml:|1. Исходный docker-compose.yml:]]
	[[Базовые методики адаптации Docker-контейнеров для работы в k8s.##2. Неявные допущения Docker Compose|2. Неявные допущения Docker Compose]]
	[[Базовые методики адаптации Docker-контейнеров для работы в k8s.##3. Декомпозиция Docker Compose на Kubernetes-ресурсы|3. Декомпозиция Docker Compose на Kubernetes-ресурсы]]
	[[Базовые методики адаптации Docker-контейнеров для работы в k8s.##4. Адаптация MySQL (db)|4. Адаптация MySQL (db)]]
		[[Базовые методики адаптации Docker-контейнеров для работы в k8s.###4.1. Stateful-характер сервиса `StatefulSet`.|4.1. Stateful-характер сервиса `StatefulSet`.]]
		[[Базовые методики адаптации Docker-контейнеров для работы в k8s.###4.2. Хранилище данных|4.2. Хранилище данных]]
		[[Базовые методики адаптации Docker-контейнеров для работы в k8s.###4.3. Конфигурация и секреты|4.3. Конфигурация и секреты]]
		[[Базовые методики адаптации Docker-контейнеров для работы в k8s.###4.4. Сетевое взаимодействие|4.4. Сетевое взаимодействие]]
		[[Базовые методики адаптации Docker-контейнеров для работы в k8s.###4.5. Порты|4.5. Порты]]
	[[Базовые методики адаптации Docker-контейнеров для работы в k8s.##5. Адаптация Redmine|5. Адаптация Redmine]]
		[[Базовые методики адаптации Docker-контейнеров для работы в k8s.###5.1. Тип нагрузки|5.1. Тип нагрузки]]
		[[Базовые методики адаптации Docker-контейнеров для работы в k8s.###5.2. Конфигурация приложения|5.2. Конфигурация приложения]]
		[[Базовые методики адаптации Docker-контейнеров для работы в k8s.###5.3. Подключение к базе данных|5.3. Подключение к базе данных]]
		[[Базовые методики адаптации Docker-контейнеров для работы в k8s.###5.4. Порты и доступ извне|5.4. Порты и доступ извне]]
	[[Базовые методики адаптации Docker-контейнеров для работы в k8s.##6. Управление жизненным циклом|6. Управление жизненным циклом]]
		[[Базовые методики адаптации Docker-контейнеров для работы в k8s.###6.1. Readiness Probe|6.1. Readiness Probe]]
		[[Базовые методики адаптации Docker-контейнеров для работы в k8s.###6.2. Liveness Probe|6.2. Liveness Probe]]
		[[Базовые методики адаптации Docker-контейнеров для работы в k8s.###6.3. Корректное завершение|6.3. Корректное завершение]]
	[[Базовые методики адаптации Docker-контейнеров для работы в k8s.##7. Управление ресурсами|7. Управление ресурсами]]
		[[Базовые методики адаптации Docker-контейнеров для работы в k8s.###7.1. Ограничения ресурсов|7.1. Ограничения ресурсов]]
	[[Базовые методики адаптации Docker-контейнеров для работы в k8s.##8. Логирование|8. Логирование]]
		[[Базовые методики адаптации Docker-контейнеров для работы в k8s.###8.1. Redmine|8.1. Redmine]]
		[[Базовые методики адаптации Docker-контейнеров для работы в k8s.###8.2. MySQL|8.2. MySQL]]
	[[Базовые методики адаптации Docker-контейнеров для работы в k8s.##9. Итоговая архитектура в Kubernetes|9. Итоговая архитектура в Kubernetes]]
		[[Базовые методики адаптации Docker-контейнеров для работы в k8s.###9.1. Набор ресурсов|9.1. Набор ресурсов]]
[[Базовые методики адаптации Docker-контейнеров для работы в k8s.#Итоговые манифесты Kubernetes для Redmine|Итоговые манифесты Kubernetes для Redmine]]
	[[Базовые методики адаптации Docker-контейнеров для работы в k8s.##1. Namespace (опционально)|1. Namespace (опционально)]]
	[[Базовые методики адаптации Docker-контейнеров для работы в k8s.##2. ConfigMap для MySQL и Redmine|2. ConfigMap для MySQL и Redmine]]
	[[Базовые методики адаптации Docker-контейнеров для работы в k8s.##3. Secrets|3. Secrets]]
	[[Базовые методики адаптации Docker-контейнеров для работы в k8s.##4. PersistentVolumeClaim для MySQL|4. PersistentVolumeClaim для MySQL]]
	[[Базовые методики адаптации Docker-контейнеров для работы в k8s.##5. Service для MySQL|5. Service для MySQL]]
	[[Базовые методики адаптации Docker-контейнеров для работы в k8s.##6. StatefulSet для MySQL|6. StatefulSet для MySQL]]
	[[Базовые методики адаптации Docker-контейнеров для работы в k8s.##7. Service для Redmine|7. Service для Redmine]]
	[[Базовые методики адаптации Docker-контейнеров для работы в k8s.##8. Deployment для Redmine|8. Deployment для Redmine]]
	[[Базовые методики адаптации Docker-контейнеров для работы в k8s.##9.  Ingress для внешнего доступа (Опционально)|9.  Ingress для внешнего доступа (Опционально)]]
	[[Базовые методики адаптации Docker-контейнеров для работы в k8s.##10. Улучшения для Production|10. Улучшения для Production]]
		[[Базовые методики адаптации Docker-контейнеров для работы в k8s.###10.1 initContainer (ожидание MySQL) добавить в Deployment Redmine (Kubernetes-native замена `depends_on`.)|10.1 initContainer (ожидание MySQL) добавить в Deployment Redmine (Kubernetes-native замена `depends_on`.)]]
		[[Базовые методики адаптации Docker-контейнеров для работы в k8s.###10.2 Добавление TLS в Ingress|10.2 Добавление TLS в Ingress]]
		[[Базовые методики адаптации Docker-контейнеров для работы в k8s.###10.3 Пробы для MySQL|10.3 Пробы для MySQL]]
		[[Базовые методики адаптации Docker-контейнеров для работы в k8s.###10.4 HPA для Redmine|10.4 HPA для Redmine]]
		[[Базовые методики адаптации Docker-контейнеров для работы в k8s.###10.5 PodSecurityContext / non-root|10.5 PodSecurityContext / non-root]]
[[Базовые методики адаптации Docker-контейнеров для работы в k8s.#Итоговая структура чарта|Итоговая структура чарта]]
	[[Базовые методики адаптации Docker-контейнеров для работы в k8s.##Как развернуть проект локально.|Как развернуть проект локально.]]
[[Базовые методики адаптации Docker-контейнеров для работы в k8s.#BONUS Конвертация Docker-compose с помощью приложения Kompose|BONUS Конвертация Docker-compose с помощью приложения Kompose]]
	[[Базовые методики адаптации Docker-контейнеров для работы в k8s.##Исходный docker-compose.yml|Исходный docker-compose.yml]]
	[[Базовые методики адаптации Docker-контейнеров для работы в k8s.##Подход к конвертации|Подход к конвертации]]
	[[Базовые методики адаптации Docker-контейнеров для работы в k8s.##Подготовленный docker-compose.yml|Подготовленный docker-compose.yml]]
	[[Базовые методики адаптации Docker-контейнеров для работы в k8s.##Установка Kompose|Установка Kompose]]
	[[Базовые методики адаптации Docker-контейнеров для работы в k8s.##Конвертация в Helm Chart|Конвертация в Helm Chart]]
	[[Базовые методики адаптации Docker-контейнеров для работы в k8s.##Полученная структура Helm Chart|Полученная структура Helm Chart]]
	[[Базовые методики адаптации Docker-контейнеров для работы в k8s.##Проверка Helm Chart|Проверка Helm Chart]]
	[[Базовые методики адаптации Docker-контейнеров для работы в k8s.##Развертывание в Kubernetes|Развертывание в Kubernetes]]
	[[Базовые методики адаптации Docker-контейнеров для работы в k8s.##Проверка ресурсов|Проверка ресурсов]]
	[[Базовые методики адаптации Docker-контейнеров для работы в k8s.##Итог|Итог]]
[[Базовые методики адаптации Docker-контейнеров для работы в k8s.#Полезные ссылки|Полезные ссылки]]
# Базовые методики адаптации Docker-контейнеров для работы в k8s.

Kubernetes (K8s) является оркестратором контейнеров, изначально разработанным для управления Docker-контейнерами. Однако Docker-образы, созданные для изолированной работы, часто требуют адаптации для эффективного и надежного функционирования в распределенной среде Kubernetes. Ниже перечислены базовые, общепринятые методики адаптации Docker-контейнеров и контейнеризованных приложений для корректной и эффективной работы в Kubernetes (K8s).

Адаптация Docker-контейнеров для Kubernetes — это приведение контейнера и приложения к принципам:
- декларативности
- эфемерности
- отказоустойчивости
- управляемости оркестратором

Чем ближе контейнер следует Kubernetes-native практикам, тем выше стабильность, масштабируемость и предсказуемость его работы в кластере.

## 1. Адаптация контейнерного образа (Dockerfile-level)

### 1.1. Один процесс — один контейнер

Kubernetes управляет контейнерами как подом и ожидает, что каждый контейнер будет выполнять один основной процесс
- Исключается запуск нескольких демонов через `supervisord`, `init.sh` и т.п.
- Побочные задачи (cron, миграции, sidecar-логика) выносятся в отдельные контейнеры или Jobs
- Рефакторинг монолитных образов, запускающих несколько процессов (например, веб-сервер и фоновая задача), в несколько специализированных контейнеров
- Хорошая практика выносить базы данных на отдельные сервера и подключать к ним контейнеры из Kubernetes
### 1.2. Корректная работа PID 1

Контейнерный процесс должен:
- корректно обрабатывать SIGTERM и SIGINT
- завершаться за разумное время
- использование `exec` в `ENTRYPOINT`/`CMD`
### 1.3. Отказ от root-пользователя

Для Kubernetes (и Pod Security Standards):
- контейнеры запускаются от непривилегированного пользователя
- задаются `USER` в Dockerfile и совместимые `fsGroup`
- возможно регулировать это в манифестах Kubernetes - настройка `securityContext` в Pod: `runAsNonRoot: true`, `runAsUser`, `readOnlyRootFilesystem: true`, отказ от привилегий (`capabilities.drop: ["ALL"]`).
### 1.4 Для надежного развертывания и отката в Kubernetes используется концепция неизменяемых развертываний.

- Отказ от использования плавающих тегов (например, `:latest`) в рабочих средах
- Каждый новый билд образа получает уникальный тег. Манифесты Kubernetes ссылаются на конкретный тег
---
## 2. Конфигурация и переменные окружения

Жестко закодированная конфигурация в Docker-образе неприемлема для динамичной среды K8s. Кконфигурации (адреса БД, флаги функций, уровни логирования) должны быть вынесены в переменные окружения (`environment`) или специальные объекты Kubernetes: ConfigMap и Secret.
### 2.1. Декларативная конфигурация

Контейнер должен получать конфигурацию извне:
- через переменные окружения
- через конфигурационные файлы, монтируемые как volume
### 2.2. Использование ConfigMap и Secret

В Kubernetes:
- ConfigMap — для некритичных параметров
- Secret — для чувствительных данных

Контейнер должен уметь:
- читать конфигурацию из env или файлов
- корректно реагировать на отсутствие значений по умолчанию
---
## 3. Работа с файловой системой и данными

Контейнеры в Kubernetes являются эфемерными и могут быть пересозданы или перемещены в любой момент.
### 3.1. Эфемерность контейнера

Файловая система контейнера считается временной:
- любые данные внутри контейнера могут быть потеряны при рестарте
- состояние выносится во внешние хранилища
### 3.2. Volume-ориентированная архитектура

Контейнер адаптируется к работе с:
- PersistentVolume (PV/PVC)
- временными volume (`emptyDir`)
- read-only монтированием конфигурации
---
## 4. Сетевое взаимодействие

### 4.1. Отказ от жёсткой привязки к IP

Контейнер не должен:
- полагаться на статические IP
- хранить адреса сервисов в конфигурации

Используются:
- DNS-имена сервисов Kubernetes
- переменные окружения, автоматически создаваемые K8s
### 4.2. Один порт — один сервис

Контейнер явно декларирует:
- какие порты он слушает
- протоколы (TCP/UDP, HTTP/gRPC)

---
## 5. Health-checks и жизненный цикл

В отличие от простого Docker, Kubernetes должен автоматически определять, работает ли контейнер корректно.
### 5.1. Readiness и Liveness probes

- **Liveness Probe:** Определяет, когда контейнер необходимо перезапустить
- **Readiness Probe:** Определяет, когда контейнер готов принимать трафик (например, завершил загрузку кэша)

Контейнер должен предоставлять:
- реализация в приложении HTTP-эндпоинтов (`/healthz`, `/readyz`) или команд проверки состояния
- endpoint или команду для проверки готовности (readiness)
- механизм обнаружения зависших состояний (liveness)

Это требует:
- встроенных health-endpoint’ов
- либо утилит командной проверки
### 5.2. Startup-поведение

Для медленно стартующих приложений:
- корректная инициализация без таймаутов
- поддержка `startupProbe`

---
## 6. Управление ресурсами

Kubernetes требует явного указания потребностей контейнера в ресурсах CPU и памяти для эффективного планирования. Для этого требуется тестирование контейнера под нагрузкой для определения базового потребления (requests) и максимально допустимого использования (limits).
### 6.1. Предсказуемое потребление ресурсов

Контейнер должен:
- корректно работать при ограничениях CPU и памяти
- не полагаться на «неограниченные» ресурсы хоста
### 6.2. Реакция на OOM и throttling

Приложение адаптируется к:
- возможным рестартам при OOMKill
- замедлению при CPU throttling

---
## 7. Логирование и наблюдаемость

Kubernetes и связанные системы мониторинга (например, EFK-стек) собирают логи из стандартных потоков вывода контейнера.
### 7.1. Логи в stdout/stderr

Контейнер:
- не пишет логи в файлы по умолчанию
- использует stdout/stderr для интеграции с logging-стеком Kubernetes
### 7.2. Метрики и трассировка

При необходимости:
- экспонирование метрик (Prometheus-совместимых)
- поддержка distributed tracing без привязки к хосту

---
## 8. Kubernetes-native подход

### 8.1. Stateless-архитектура

Контейнер проектируется как:
- масштабируемый;
- безопасный для горизонтального масштабирования
- не хранящий состояние локально

### 8.2. Разделение ответственности

Вместо «умного» контейнера:
- orchestration-логика переносится в Kubernetes (Deployment, Job, CronJob)
- зависимости описываются декларативно, а не скриптами внутри контейнера

---
## 9. Методики переноса существующих Docker-контейнеров

На практике применяются три базовые стратегии:

1. **Минимальная адаптация** — доработка Dockerfile и entrypoint без изменения архитектуры
2. **Рефакторинг контейнера** — переработка образа под stateless и cloud-native модель
3. **Реархитектура приложения** — изменение логики приложения под принципы Kubernetes

---
# Сопоставление сущностей Docker Compose и Kubernetes

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
# Адаптация Docker Compose (Redmine + MySQL) для Kubernetes 

## 1. Исходный docker-compose.yml:

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

Данный Docker Compose описывает два сервиса:

**redmine**
- Stateless web-приложение
- Работает на порту `3000`
- Экспонируется наружу через `8080:3000`
- Зависит от MySQL
- Использует переменные окружения для конфигурации

**База данных db (MySQL 8.0)**
- Stateful сервис
- Хранит данные внутри контейнера
- Использует root-пароль
- Неявно предполагает постоянное хранилище

## 2. Неявные допущения Docker Compose

Docker Compose опирается на допущения, **несовместимые с Kubernetes без адаптации**

| Допущение                  | K8s                                            |
| -------------------------- | ---------------------------------------------- |
| `restart: always`          | В Kubernetes рестарт управляется контроллерами |
| Локальное хранилище MySQL  | Pod эфемерен                                   |
| Статическое имя `db`       | Требует Service                                |
| Пароли в environment       | Нужны Secrets                                  |
| Проброс портов `8080:3000` | В K8s используется Service / Ingress           |
| Отсутствие probes          | Kubernetes не знает состояние приложения       |
## 3. Декомпозиция Docker Compose на Kubernetes-ресурсы

| Compose          | Kubernetes            |
| ---------------- | --------------------- |
| service: redmine | Deployment + Service  |
| service: db      | StatefulSet + Service |
| environment      | ConfigMap + Secret    |
| ports            | Service               |
| volumes (неявно) | PersistentVolumeClaim |
## 4. Адаптация MySQL (db)

### 4.1. Stateful-характер сервиса `StatefulSet`.
- хранит состояние
- требует стабильного volume
- не масштабируется горизонтально без дополнительной логики.
### 4.2. Хранилище данных
- данные MySQL **обязательно выносятся** в `PersistentVolumeClaim`;
- путь `/var/lib/mysql` монтируется из PVC.
### 4.3. Конфигурация и секреты

Переменные окружения из Compose:
`MYSQL_ROOT_PASSWORD: example MYSQL_DATABASE: redmine`

Адаптация:
- `MYSQL_ROOT_PASSWORD` → **Secret** - пароль это чувствительные данные
- `MYSQL_DATABASE` → **ConfigMap** - имя БД это конфигурационный параметр
### 4.4. Сетевое взаимодействие

В Docker Compose контейнер доступен как `db`.
В Kubernetes:
- создаётся `Service` с DNS-именем, например: `mysql.default.svc.cluster.local`
- Redmine подключается по DNS, а не по IP
### 4.5. Порты

`ports:   - 3306`
- **порты не публикуются наружу**, если это не требуется;
- `Service` типа `ClusterIP`.
## 5. Адаптация Redmine

### 5.1. Тип нагрузки

Redmine:
- stateless web-приложение
- может быть масштабировано горизонтально
- не хранит критическое состояние локально

**Выбор ресурса:** `Deployment`
### 5.2. Конфигурация приложения

Переменные окружения:

`REDMINE_DB_MYSQL: db REDMINE_DB_PASSWORD: example REDMINE_SECRET_KEY_BASE: supersecretkey`

| Переменная                | Kubernetes-ресурс |
| ------------------------- | ----------------- |
| `REDMINE_DB_MYSQL`        | ConfigMap         |
| `REDMINE_DB_PASSWORD`     | Secret            |
| `REDMINE_SECRET_KEY_BASE` | Secret            |

Контейнер **не должен содержать секреты в образе**.
### 5.3. Подключение к базе данных

Redmine больше не зависит от порядка старта контейнеров. Для корректной обработки требуется init контейнер который будет ждать запуск базы данных.

### 5.4. Порты и доступ извне

`ports:   - 8080:3000`

1. Контейнер слушает порт `3000`
2. Создаётся `Service`:
- `ClusterIP` — для внутреннего доступа
- `NodePort` или `Ingress` — для внешнего

Проброс портов контейнера **не используется**.
## 6. Управление жизненным циклом

### 6.1. Readiness Probe

Redmine должен:
- принимать HTTP-запросы;
- быть готовым к работе только после подключения к БД

Добавляется:
- `readinessProbe` на HTTP endpoint (например `/`)
### 6.2. Liveness Probe

Позволяет Kubernetes:
- перезапускать контейнер при зависании
- отличать ошибку приложения от временной недоступности БД
### 6.3. Корректное завершение

При удалении Pod:
- Redmine получает `SIGTERM`
- завершает активные HTTP-запросы
- Kubernetes ждёт `terminationGracePeriodSeconds`
## 7. Управление ресурсами

### 7.1. Ограничения ресурсов

В Compose ресурсы не ограничены.

В Kubernetes:
- задаются `requests` и `limits` для CPU и памяти
- предотвращается неконтролируемое потребление ресурсов
## 8. Логирование

### 8.1. Redmine
- Логи выводятся в stdout/stderr
- Kubernetes собирает их централизованно
### 8.2. MySQL
- Используется стандартный logging драйвер контейнера
- Отказ от файловых логов внутри контейнера
## 9. Итоговая архитектура в Kubernetes

### 9.1. Набор ресурсов

- `ConfigMap` — конфигурация Redmine и MySQL
- `Secret` — пароли и ключи
- `StatefulSet` — MySQL
- `PersistentVolumeClaim` — данные MySQL
- `Deployment` — Redmine
- `Service (ClusterIP)` — MySQL
- `Service / Ingress` — Redmine


# Итоговые манифесты Kubernetes для Redmine

## 1. Namespace (опционально)

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: redmine
```

## 2. ConfigMap для MySQL и Redmine

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: redmine-config
  namespace: redmine
data:
  MYSQL_DATABASE: redmine
  REDMINE_DB_MYSQL: mysql
```

## 3. Secrets

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: redmine-secrets
  namespace: redmine
type: Opaque
stringData:
  MYSQL_ROOT_PASSWORD: example
  REDMINE_DB_PASSWORD: example
  REDMINE_SECRET_KEY_BASE: supersecretkey
```

> В production-среде секреты должны создаваться вне Git (Vault, External Secrets и т.п.).

## 4. PersistentVolumeClaim для MySQL

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pvc
  namespace: redmine
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```

## 5. Service для MySQL

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql
  namespace: redmine
spec:
  type: ClusterIP
  ports:
    - port: 3306
      targetPort: 3306
  selector:
    app: mysql
```

## 6. StatefulSet для MySQL

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
  namespace: redmine
spec:
  serviceName: mysql
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
          ports:
            - containerPort: 3306
          env:
            - name: MYSQL_DATABASE
              valueFrom:
                configMapKeyRef:
                  name: redmine-config
                  key: MYSQL_DATABASE
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: redmine-secrets
                  key: MYSQL_ROOT_PASSWORD
          volumeMounts:
            - name: mysql-data
              mountPath: /var/lib/mysql
          resources:
            requests:
              cpu: "250m"
              memory: "512Mi"
            limits:
              cpu: "1"
              memory: "1Gi"
  volumeClaimTemplates:
    - metadata:
        name: mysql-data
      spec:
        accessModes:
          - ReadWriteOnce
        resources:
          requests:
            storage: 10Gi
```

## 7. Service для Redmine

```yaml
apiVersion: v1
kind: Service
metadata:
  name: redmine
  namespace: redmine
spec:
  type: ClusterIP
  ports:
    - port: 80
      targetPort: 3000
  selector:
    app: redmine
```

## 8. Deployment для Redmine

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redmine
  namespace: redmine
spec:
  replicas: 2
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
          image: redmine:latest
          ports:
            - containerPort: 3000
          env:
            - name: REDMINE_DB_MYSQL
              valueFrom:
                configMapKeyRef:
                  name: redmine-config
                  key: REDMINE_DB_MYSQL
            - name: REDMINE_DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: redmine-secrets
                  key: REDMINE_DB_PASSWORD
            - name: REDMINE_SECRET_KEY_BASE
              valueFrom:
                secretKeyRef:
                  name: redmine-secrets
                  key: REDMINE_SECRET_KEY_BASE
          readinessProbe:
            httpGet:
              path: /
              port: 3000
            initialDelaySeconds: 30
            periodSeconds: 10
          livenessProbe:
            httpGet:
              path: /
              port: 3000
            initialDelaySeconds: 60
            periodSeconds: 20
          resources:
            requests:
              cpu: "250m"
              memory: "512Mi"
            limits:
              cpu: "1"
              memory: "1Gi"
```

## 9.  Ingress для внешнего доступа (Опционально)

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: redmine
  namespace: redmine
spec:
  rules:
    - host: redmine.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: redmine
                port:
                  number: 80
```

## 10. Улучшения для Production

### 10.1 initContainer (ожидание MySQL) добавить в Deployment Redmine (Kubernetes-native замена `depends_on`.)

```yaml
initContainers:
  - name: wait-for-mysql
    image: busybox
    command:
      - sh
      - -c
      - until nc -z redmine-mysql 3306; do sleep 5; done
```

### 10.2 Добавление TLS в Ingress

```yaml
ingress:
  enabled: true
  className: nginx
  host: redmine.example.com
  tls:
    enabled: true
    secretName: redmine-tls
```

### 10.3 Пробы для MySQL

```yaml
readinessProbe:
  tcpSocket:
    port: 3306

livenessProbe:
  tcpSocket:
    port: 3306
```

### 10.4 HPA для Redmine

```yaml
{{- if .Values.hpa.enabled }}
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
	name: {{ .Release.Name }}-redmine
spec:
	scaleTargetRef:
		apiVersion: apps/v1
		kind: Deployment
		name: {{ .Release.Name }}-redmine
	minReplicas: {{ .Values.hpa.minReplicas }}
	maxReplicas: {{ .Values.hpa.maxReplicas }}
	metrics:
	  - type: Resource
		resource:
			name: cpu
			target:
				type: Utilization
				averageUtilization: {{ .Values.hpa.targetCPUUtilizationPercentage }}
{{- end }}
```

 values.yaml

```yaml
hpa:
  enabled: true
  minReplicas: 1
  maxReplicas: 5
  targetCPUUtilizationPercentage: 70
```

### 10.5 PodSecurityContext / non-root

Redmine

```yaml
securityContext:
  runAsUser: 1000
  runAsGroup: 1000
  fsGroup: 1000
```

MySQL

```yaml
securityContext:
  fsGroup: 1000
```

---

#  Итоговая структура чарта

redmine/
├── Chart.yaml
├── values.yaml
└── templates/
    ├── configmap.yaml
    ├── secret.yaml
    ├── mysql-service.yaml
    ├── mysql-statefulset.yaml
    ├── redmine-deployment.yaml
    ├── redmine-service.yaml
    ├── hpa.yaml
    └── ingress.yaml

Итоговая структура чарта будет выглядеть как на диаграмме выше и доступна в репозитории с рабочими файлами. Это максимальный пример того как может быть перенесен один сервис на Kubernetes. Это информация для вдумчивого и первоначального изучения и понимания основ миграции, она не идеальна и вряд ли полностью пригодна для работы в продуктивной среде, но я попытался описать максимально возможные варианты. Для полноценной миграции вашего сервиса пользуйтесь документацией Kubernetes.

## Как развернуть проект локально.

1. Поднять кластер Kubernetes по типу Minikube или с помощью приложения Docker. 
2. Далее задеплоить с помощью Helm

Команды которые помогут:

kubectl create namespace redmine  - создаем namespace в нашем кластере
helm install redmine . -n redmine - устанавливаем наш helm chart
kubectl get pod -n redmine - смотрим какие наши поды
kubectl get svc -n redmine - смотрим наши сервисы
kubectl get hpa -n redmine - смотрим hpa контроллер
kubectl port-forward svc/redmine-redmine 3000:80 -n redmine - перебрасываем порт на localhost
https://localhost:3000/ - смотрим наше равзернутое приложение


# BONUS Конвертация Docker-compose с помощью приложения Kompose

Kompose — это официальный инструмент CNCF (Cloud Native Computing Foundation), предназначенный для автоматической конвертации docker-compose.yml в Kubernetes-манифесты.
Он не «понимает» Kubernetes, а делает механическое сопоставление сущностей Compose → K8s.

Как работает Kompose:

- Читает docker-compose.yml
- services
- volumes
- networks
- environment
- ports
- depends_on (частично)

Строит внутреннюю модель:

- один сервис ≈ один workload
- тома → PVC / emptyDir
- порты → Service

Генерирует Kubernetes YAML

- Deployment или StatefulSet
- Service
- PersistentVolumeClaim
- ConfigMap (редко, в основном env inline)
- добаляет большое количество kompose.* аннотаций
- (Опционально) Генерирует Helm Chart


Важно: Kompose не умеет корректно обрабатывать:

 - init-логику
 - Порядок старта, Liveness, Readness, Startup Probe
 - health-based зависимости
 - ingress
 - secrets
 Поэтому является лишь инструментом подготовки к переносу и настройки приложения для работы с Kubernetes

## Исходный docker-compose.yml

```yaml
services:
  redmine:
    image: redmine
    ports:
      - 8080:3000
    environment:
      REDMINE_DB_MYSQL: db
      REDMINE_DB_PASSWORD: example
      REDMINE_SECRET_KEY_BASE: supersecretkey

  db:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: example
      MYSQL_DATABASE: redmine
```
## Подход к конвертации

Kompose использует:
- services → workloads
- volumes → PVC
- labels → Kubernetes-специфичные параметры

Все параметры Kubernetes добавляются через labels.
## Подготовленный docker-compose.yml

Для того что бы не много автоматизировать процесс конвертации,  Kompose использует лейблы значение которых описано в его документации. Например лейбл kompose.service.expose создаст ingress. А kompose.controller.type: statefulset - создаст statefulset а не deployment который бы создался по умолчанию.

```yaml
version: "3.9"

services:
  redmine:
    image: redmine:5
    ports:
      - "8080:3000"
    environment:
      REDMINE_DB_MYSQL: db
      REDMINE_DB_PASSWORD: example
      REDMINE_SECRET_KEY_BASE: supersecretkey
    labels:
      kompose.service.expose: "redmine.example.local"


  db:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: example
      MYSQL_DATABASE: redmine
    volumes:
      - db-data:/var/lib/mysql
    labels:
      kompose.controller.type: statefulset

volumes:
  db-data:
```
## Установка Kompose

```bash
brew install kompose
```

или

```bash
curl -L https://github.com/kubernetes/kompose/releases/download/v1.34.0/kompose-linux-amd64 -o kompose
chmod +x kompose
sudo mv kompose /usr/local/bin/
```
## Конвертация в Helm Chart

```bash
kompose convert --chart --out redmine
```
## Полученная структура Helm Chart

```text
redmine/
├── Chart.yaml
├── values.yaml
├── templates/
│   ├── redmine-deployment.yaml
│   ├── redmine-service.yaml
│   ├── redmine-ingress.yaml
│   ├── mysql-statefulset.yaml
│   ├── mysql-service.yaml
│   ├── mysql-persistentvolumeclaim.yaml
│   └── NOTES.txt
```

## Проверка Helm Chart

```bash
helm lint ./redmine
helm template ./redmine
```
## Развертывание в Kubernetes

```bash
kubectl create namespace test-redmine
helm install redmine ./redmine --namespace test-redmine
```
## Проверка ресурсов

```bash
kubectl get all -n test-redmine
kubectl get pvc -n test-redmine
kubectl get ingress -n test-redmine
```

## Итог

- docker-compose подготовлен под Kubernetes заранее
- Kompose используется для конвертации в Helm Chart
- БД разворачивается как StatefulSet
- Persistent Volume создается автоматически
- Результат воспроизводим и пригоден для дальнейшей доработки
- К сожалению имеет не большой функционал и не пригоден для production

# Полезные ссылки
