# Контейнери в Kubernetes (containers, initContainers)

## Що таке “контейнер” у Kubernetes
У Kubernetes контейнер — це процес(и), запущені вашим container runtime (containerd/CRI-O тощо) на вузлі (node) згідно зі специфікацією Pod.

Важливо розуміти термінологію:
- **Pod** — це “обгортка/група” для контейнерів зі спільними ресурсами (мережевий namespace, volumes, метадані).
- **Контейнери** — це те, що реально запускається runtime-ом на вузлі.
- Контейнери не “створюються всередині Pod”, навпаки: Pod — абстракція, яка описує набір контейнерів, а kubelet на ноді вже створює й запускає їх.

## Де контейнер “створюється” насправді
Коротка відповідь: **на конкретному Node**, а не “всередині Pod”.

Як це працює практично:
- **Pod** живе в Kubernetes як об’єкт/опис у **API Server** (yaml → запис у кластері).
- Коли Scheduler призначив Pod на ноду, **kubelet на цій ноді** читає `spec` Pod.
- Далі kubelet через CRI просить **container runtime** (наприклад, containerd/CRI-O) **створити та запустити контейнери**.

Тобто “Pod” — це **декларація** (що треба запустити), а “контейнери” — це **реальні процеси**, які запускаються runtime-ом на ноді.

## Де описуються контейнери в маніфесті Pod
У `spec` Pod є кілька списків:
- `spec.containers` — основні (app) контейнери, які мають працювати під час життя Pod.
- `spec.initContainers` — контейнер(и) для підготовки, які мають завершитися успішно ДО старту основних контейнерів.
- `spec.ephemeralContainers` — тимчасові контейнери для дебагу (часто додаються командою `kubectl debug`; не є “частиною” звичайного деплойменту в тому ж сенсі, що `containers`).

## Коли і в який момент створюються контейнери (таймлайн)
Нижче — типовий сценарій, коли ви створюєте Pod/Deployment:

## Схема (візуально)
Як читати: рухаємося зверху вниз. Поки Pod не призначили на ноду — контейнери фізично НЕ створюються.

### ASCII-схема (простий варіант)
```text
Клієнт/контролер        API Server           Scheduler              Node (kubelet + runtime)
------------------      -----------          ---------              -----------------------
kubectl apply
або Deployment/Job  -->  створюється Pod
                           (лише запис)  -->  вибір Node
                                              + запис spec.nodeName  -->  створення Pod sandbox
                                                                       (мережа CNI + volumes)
                                                                     --> initContainers: init-1 -> init-2 -> ... -> init-N
                                                                     --> main containers: app-1 || app-2 || ...
                                                                     --> Pod Running
```

### Mermaid-схема (якщо ваш Markdown підтримує Mermaid)
```mermaid
flowchart TD
  A[Створення Pod/Deployment] --> B[Pod з'являється в API Server (лише запис)]
  B --> C[Scheduler призначає Pod на Node (spec.nodeName)]
  C --> D[kubelet на Node підхоплює Pod]
  D --> E[Pod sandbox: мережа (CNI) + volumes]
  E --> F[initContainers (послідовно: init-1 → init-2 → ...)]
  F --> G[main containers (паралельно)]
  G --> H[Pod Running]
```

1) **Ви створюєте об’єкт Pod** (напряму або через Deployment/Job).
- Pod з’являється в API Server.

2) **Scheduler призначає Pod на конкретну ноду**.
- У Pod з’являється поле `spec.nodeName`.

3) **kubelet на цій ноді бачить Pod і починає “materialize” його**.
- kubelet через CRI просить runtime підготувати Pod.

4) **Створюється Pod sandbox**.
- Runtime створює “sandbox” (часто це спеціальний pause-контейнер) і мережевий namespace.
- Налаштовується мережа (через CNI) та монтуються volumes.

5) **Запускаються `initContainers` (якщо вони є)**.
- Запускаються **послідовно** (1 → 2 → 3 …).
- Кожен init-контейнер має завершитись з exit code 0.
- Якщо init-контейнер падає — kubelet може його перезапускати (поведінка залежить від політик і версії, але для вас правило просте: Pod не перейде до запуску основних контейнерів, доки init не пройшли).

6) **Запускаються основні `containers`**.
- Після успішного завершення всіх init-контейнерів kubelet створює і стартує основні контейнери.

7) **Pod у стані Running**, поки працює хоча б один основний контейнер (залежить від типу workload).

## Перезапуск контейнерів vs “перезапуск Pod”
- Kubernetes зазвичай **перезапускає контейнер** в межах того самого Pod (наприклад, при crash loop) — це робота kubelet.
- Pod як об’єкт може бути “замінений” контролером (ReplicaSet/Deployment): якщо Pod видалили/загинув — контролер створить НОВИЙ Pod (з новим ім’ям/UID).

Ключові налаштування:
- `spec.restartPolicy` (для Pod): `Always` (типово), `OnFailure`, `Never`.
- Probe-и (`livenessProbe`, `readinessProbe`, `startupProbe`) можуть спричиняти перезапуски контейнерів або впливати на готовність.

## Часті питання
### Чому initContainers потрібні, якщо можна “sleep” в основному контейнері?
Тому що init-контейнер:
- ізолює підготовчі дії від основного процесу застосунку;
- гарантує порядок: “спочатку підготувати/дочекатися — потім запускати app”.

### Чи мають initContainers доступ до volume?
Так — init-контейнери можуть монтувати ті ж volumes і готувати дані/файли для основних контейнерів.

## Корисні команди
- Подивитись, на якій ноді Pod: `kubectl get pod <pod> -o wide`
- Подивитись стан init/containers: `kubectl describe pod <pod>`
- Логи init-контейнера: `kubectl logs <pod> -c <init-container>`
- Логи основного контейнера: `kubectl logs <pod> -c <container>`

## Додатково
- Pod lifecycle: https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/
- Init containers: https://kubernetes.io/docs/concepts/workloads/pods/init-containers/
- Node (що це і коли з'являється): [node-doc.md](node-doc.md)
- Cluster (що таке кластер): [cluster-doc.md](cluster-doc.md)
