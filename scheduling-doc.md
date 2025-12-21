# Scheduling у Kubernetes (NodeSelector, Affinity, AntiAffinity)

## Навіщо це потрібно
Scheduler (планувальник) вирішує, **на яку Node** поставити Pod. За замовчуванням він намагається знайти будь-яку підходящу ноду з ресурсами.

Механізми нижче дозволяють керувати розміщенням:
- обмежити Pod тільки певними нодами (за labels)
- розміщувати Pod поруч/не поруч з іншими Pod’ами
- уникати нод з “taints” або дозволяти розміщення через “tolerations”

## Де це в YAML
У Pod це описується в `spec`:
- `spec.nodeSelector`
- `spec.affinity`:
  - `nodeAffinity`
  - `podAffinity`
  - `podAntiAffinity`
- `spec.tolerations`

## nodeSelector (найпростіше)
Вибирає ноди за labels.

```yaml
spec:
  nodeSelector:
    disktype: ssd
```

## NodeAffinity
Схоже на `nodeSelector`, але гнучкіше (вирази, required/preferred).

- **requiredDuringSchedulingIgnoredDuringExecution** — жорстка вимога (якщо не виконується, Pod буде Pending).
- **preferredDuringSchedulingIgnoredDuringExecution** — м’яке побажання (scheduler спробує, але може поставити інакше).

Приклад (жорстко вимагати label):
```yaml
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: disktype
                operator: In
                values: ["ssd"]
```

## PodAffinity
Каже scheduler’у: “розмісти цей Pod **поруч** з Pod’ами, які мають певні labels”.

Ключовий параметр — `topologyKey`:
- якщо `topologyKey: kubernetes.io/hostname` — “поруч” означає на тій самій ноді
- якщо `topologyKey: topology.kubernetes.io/zone` — “поруч” означає в тій самій зоні

Приклад (м’яко намагатися бути на тій самій ноді, що й `app=app2`):
```yaml
spec:
  affinity:
    podAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
        - weight: 100
          podAffinityTerm:
            labelSelector:
              matchLabels:
                app: app2
            topologyKey: kubernetes.io/hostname
```

## PodAntiAffinity
Протилежність PodAffinity: “розмісти Pod **не поруч** з іншими Pod’ами”.

## Схема (скріншот)
![PodAffinity / PodAntiAffinity](scheduling-affinity.png)

Найчастіший кейс — **розносити репліки одного застосунку по різних нодах**, щоб падіння однієї ноди не вбило всі репліки.

Приклад (жорстко заборонити 2 Pod’и з `app=app1` на одній ноді):
```yaml
spec:
  affinity:
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        - labelSelector:
            matchLabels:
              app: app1
          topologyKey: kubernetes.io/hostname
```

## "Soft" vs "Hard" правила (як на твоєму скріні)
- **Soft** = `preferredDuringSchedulingIgnoredDuringExecution` (побажання)
- **Hard** = `requiredDuringSchedulingIgnoredDuringExecution` (вимога)

## Taints & Tolerations (коротко)
Це інший механізм:
- **taint** ставиться на Node і каже: “сюди не можна планувати Pod’и без дозволу”.
- **toleration** додається в Pod і каже: “мені можна на цю tainted-ноду”.

## Як дебажити, чому Pod не планується
- `kubectl describe pod <pod> -n <ns>` — подивитись Events (часто там причина)
- `kubectl get nodes --show-labels` — перевірити labels нод
- `kubectl describe node <node>` — перевірити taints і ресурси

## Додатково
- Affinity and anti-affinity: https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/
