# StatefulSet

**StatefulSet** — контролер для стано-залежних застосунків (DB, черги, кластери), де важливі:
- **стабільні імена Pod-ів** (наприклад, `mysql-0`, `mysql-1`)
- **стабільні DNS-імена**
- **стабільне сховище** (окремий PV для кожної репліки)
- **упорядкований старт/скейл** (0 → 1 → 2) і (2 → 1 → 0)

## Візуальна ідея (Headless Service + DNS)

![StatefulSet + headless Service (ClusterIP: None) + DNS для кожного pod](./screenshots/statfull_set.png)

На схемі показано типову модель для БД:
- є StatefulSet з репліками
- є **headless Service** (`clusterIP: None`)
- кожна репліка має **своє DNS** (наприклад, `db-mysql-0.default.svc...`, `db-mysql-1.default.svc...`)

## Чому тут Service з `clusterIP: None` (headless)

Headless Service **не дає один спільний ClusterIP** і не робить load-balancing на один IP.
Натомість він публікує в DNS **окремі записи для кожного Pod-а**, щоб можна було звертатись до конкретної репліки.

Формула DNS для Pod у StatefulSet (спрощено):

`<pod-name>.<service-name>.<namespace>.svc.cluster.local`

Наприклад:
- `db-mysql-0.db-mysql.default.svc.cluster.local`
- `db-mysql-1.db-mysql.default.svc.cluster.local`

> На твоєму скріні видно ідею: звертаємось до конкретного pod через DNS.

## StatefulSet vs Deployment

- **Deployment**: Pod-и взаємозамінні, імена випадкові, підходить для stateless (web, api).
- **StatefulSet**: Pod-и “особистості” (identity), кожен має стабільну назву/мережу/диск.

## Зберігання (volumeClaimTemplates)

StatefulSet часто описує `volumeClaimTemplates`, щоб для кожної репліки автоматично створювався **свій PVC** (а під нього — свій PV).

Це дає:
- `mysql-0` завжди монтує “свій” том
- `mysql-1` — інший том

## Мінімальний приклад YAML (дуже спрощено)

> Це “форма”, щоб побачити ключові частини: headless Service + StatefulSet.

### 1) Headless Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: db-mysql
spec:
  clusterIP: None
  selector:
    app: db-mysql
  ports:
    - name: mysql
      port: 3306
      targetPort: 3306
```

### 2) StatefulSet

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: db-mysql
spec:
  serviceName: db-mysql
  replicas: 2
  selector:
    matchLabels:
      app: db-mysql
  template:
    metadata:
      labels:
        app: db-mysql
    spec:
      containers:
        - name: mysql
          image: mysql:8
          ports:
            - containerPort: 3306
          volumeMounts:
            - name: data
              mountPath: /var/lib/mysql
  volumeClaimTemplates:
    - metadata:
        name: data
      spec:
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: 10Gi
```

## Корисні команди kubectl

```bash
kubectl get sts -A
kubectl describe sts <name> -n <namespace>

kubectl get pods -n <namespace> -l app=db-mysql -o wide

# Подивитись, які PVC створились для кожної репліки
kubectl get pvc -n <namespace>

# Перевірити DNS записи (потрібен pod з shell, наприклад busybox)
# nslookup db-mysql-0.db-mysql.default.svc.cluster.local
```

## Посилання

- https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/
- Headless Services: https://kubernetes.io/docs/concepts/services-networking/service/#headless-services
