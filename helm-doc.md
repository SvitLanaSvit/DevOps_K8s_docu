# Helm

**Helm** — це пакетний менеджер для Kubernetes.
Він дозволяє встановлювати/оновлювати/видаляти набори ресурсів Kubernetes як один “пакет”, використовуючи шаблони і параметри.

## Основні терміни

- **Chart** — пакет Helm (набір шаблонів + значення + метадані).
- **Release** — інсталяція Chart у кластер (конкретний екземпляр із конкретними `values`).
- **Values** — параметри, які підставляються в шаблони (наприклад, `replicaCount`, `image.tag`, порти).
- **Repository** — місце, де зберігаються charts (аналог “репозиторію пакетів”).

## Навіщо Helm

- один командний крок для встановлення цілого застосунку (Deployment/Service/ConfigMap/PVC…)
- оновлення версій (upgrade) і **rollback**
- параметризація під середовища (dev/stage/prod) через `values.yaml`
- повторне використання (chart можна перевикористати багато разів)

## Chart структура (мінімум)

Типова структура:

- `Chart.yaml` — назва/версія chart
- `values.yaml` — дефолтні значення
- `templates/` — YAML-шаблони Kubernetes ресурсів

## Як Helm “генерує” YAML

Helm бере `templates/*.yaml` і підставляє значення з `values.yaml` + твої `--set` / `-f my-values.yaml`.

Корисно перевіряти, що саме буде застосовано:

```bash
helm template <release-name> <chart-path-or-repo/chart> -n <namespace>
```

## Найчастіші команди

### Репозиторії

```bash
helm repo add <name> <url>
helm repo update
helm search repo <keyword>
```

### Встановлення / оновлення

```bash
# Встановити
helm install <release-name> <repo/chart> -n <namespace> --create-namespace

# Оновити реліз
helm upgrade <release-name> <repo/chart> -n <namespace>

# Install-or-upgrade (практичний варіант)
helm upgrade --install <release-name> <repo/chart> -n <namespace> --create-namespace
```

### Values

```bash
# Подивитись values chart
helm show values <repo/chart>

# Передати values файлом
helm upgrade --install <release> <repo/chart> -n <ns> -f values.yaml

# Передати одиночне значення
helm upgrade --install <release> <repo/chart> -n <ns> --set image.tag=1.2.3
```

### Перегляд стану

```bash
helm list -A
helm status <release-name> -n <namespace>
helm get manifest <release-name> -n <namespace>
helm get values <release-name> -n <namespace>
```

### Rollback / uninstall

```bash
helm history <release-name> -n <namespace>
helm rollback <release-name> <revision> -n <namespace>

helm uninstall <release-name> -n <namespace>
```

## Helm vs kubectl apply (інтуїтивно)

- `kubectl apply -f ...`: ти сам керуєш YAML, руками підтримуєш відмінності між середовищами.
- Helm: ти керуєш **шаблонами** + **values**, Helm управляє релізом і дає rollback.

## Поширені помилки

- “Не в той namespace”: Helm завжди прив’язує release до namespace, перевіряй `-n <namespace>`.
- Конфлікт імен: два релізи можуть створити ресурси з однаковими іменами (залежить від chart), тоді буде помилка.
- Значення не застосувались: перевір `helm get values` і `helm get manifest`.

## Artifact Hub (де шукати charts)

**Artifact Hub** — це каталог (directory) для Kubernetes artifacts, зокрема **Helm charts**. Там зручно:
- шукати офіційні/популярні charts
- дивитись приклади `values.yaml`, версії та документацію
- знаходити назву repo/chart, яку потім використовуєш у `helm repo add` / `helm install`

Сайт: https://artifacthub.io/

## Посилання

- https://helm.sh/docs/
- Artifact Hub: https://artifacthub.io/
