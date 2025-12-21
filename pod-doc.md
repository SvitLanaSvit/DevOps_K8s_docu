# Документація про Pod у Kubernetes

## Опис
Pod — це найменша та найпростіша одиниця розгортання в Kubernetes. Pod містить один або декілька контейнерів, які мають спільні ресурси: мережу, сховище та специфікації запуску.

## Основні характеристики Pod:
- Може містити один або кілька контейнерів
- Контейнери в одному Pod мають спільну IP-адресу та порти
- Використовують спільні томи для зберігання даних
- Керується контролерами (наприклад, Deployment, ReplicaSet, Job і т.д.)

## Коли створюється новий Pod
- коли старий контейнер впав
- при оновленні застосунку
- при масштабуванні
Тому Pod – короткоживуча сутність.

## Як і коли створюються контейнери в Pod (таймлайн)
Pod спочатку існує як запис в Kubernetes API, а контейнери реально створюються вже на конкретній ноді, коли Pod туди призначив scheduler.

Типова послідовність:
1) Ви створюєте Pod/Deployment → Pod з’являється в API.
2) Scheduler призначає Pod на ноду.
3) kubelet на ноді створює Pod sandbox (мережа/volumes).
4) Запускаються `initContainers` (послідовно) — якщо вони є.
5) Запускаються основні `containers`.

Більш детально: див. [containers-doc.md](containers-doc.md).

## YAML-приклад Pod:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  containers:
    - name: example-container
      image: nginx:latest
      ports:
        - containerPort: 80
```

## InitContainers
`initContainers` — це спеціальні контейнери, які виконуються ДО запуску основних контейнерів у Pod.

Важливе:
- Запускаються послідовно (один за одним) і мають успішно завершитись (exit code 0).
- Поки init-контейнери не завершились — основні контейнери не стартують.
- Зручно для підготовчих кроків, які не повинні постійно працювати в основному контейнері.

Типові сценарії:
- Очікувати готовності залежності (БД/черги) перед стартом застосунку
- Підготувати файли/конфіг у volume
- Виконати міграції БД або одноразову ініціалізацію

### InitContainer як sidecar (restartPolicy: Always)
Є режим, коли initContainer може працювати як *sidecar*: для цього в init-контейнері задають `restartPolicy: Always`.
Такий контейнер стартує **до** основних контейнерів і може працювати паралельно з ними.

Детальніше та приклад YAML: [containers-doc.md](containers-doc.md).

### YAML-приклад Pod з initContainers
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: example-pod-with-init
spec:
  initContainers:
    - name: wait-for-service
      image: busybox:1.36
      command: ['sh', '-c', 'until nslookup example-service; do echo waiting; sleep 2; done']
  containers:
    - name: app
      image: nginx:latest
      ports:
        - containerPort: 80
```

### Корисні команди
- Подивитись статуси init-контейнерів: `kubectl describe pod <pod-name>`
- Логи конкретного init-контейнера: `kubectl logs <pod-name> -c <init-container-name>`
- Список контейнерів у Pod (включно з init): `kubectl get pod <pod-name> -o jsonpath='{.spec.initContainers[*].name} {.spec.containers[*].name}'`

## Scheduling (де Pod буде запущений)
Scheduler вибирає, **на якій Node** буде запущено Pod. Це можна контролювати через:
- `nodeSelector` / `nodeAffinity` — вибір нод за labels
- `podAffinity` — “постав поруч з іншими Pod’ами”
- `podAntiAffinity` — “не став поруч з іншими Pod’ами”

Детальніше з прикладами: [scheduling-doc.md](scheduling-doc.md).

## Конфігурація Pod (ConfigMap)
Часто Pod отримує конфіг через **ConfigMap**:
- як env змінні
- або як файли через volume

Детальніше з прикладами: [configmap-doc.md](configmap-doc.md).

## Основні команди для роботи з Pod:
- Створити Pod: `kubectl apply -f pod.yaml`
- Переглянути Pod: `kubectl get pods`
- Переглянути деталі Pod: `kubectl describe pod <pod-name>`
- Видалити Pod: `kubectl delete pod <pod-name>`

## Додатково
Докладніше: [Офіційна документація Kubernetes](https://kubernetes.io/docs/concepts/workloads/pods/)
