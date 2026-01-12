# Secrets

**Secret** — це Kubernetes-об’єкт для зберігання чутливих даних (паролі, токени, ключі), які потім можна підключати в Pod як:
- **env змінні**
- **файли** через volume
- **imagePullSecrets** (доступ до приватного registry)

> Важливо: значення в Secret зазвичай **base64**, це **не шифрування**. Безпека залежить від налаштувань кластера (RBAC, encryption at rest, доступ до etcd).

## Secret vs ConfigMap

- **ConfigMap** — некритичні конфіги.
- **Secret** — чутливі дані.

Технічно інжект у Pod схожий (env/volume), різниця — в призначенні і політиках безпеки.

## Типи Secret

- `Opaque` — звичайний Secret (найчастіше).
- `kubernetes.io/tls` — TLS сертифікат (`tls.crt` + `tls.key`).
- `kubernetes.io/dockerconfigjson` — креденшіали для docker registry.

## Створення Secret (через kubectl)

### 1) Літеральні значення (Opaque)

```bash
kubectl create secret generic db-creds \
  --from-literal=username=app \
  --from-literal=password='S3cr3t!'
```

### 2) З файлу

```bash
kubectl create secret generic app-key \
  --from-file=private.key=./private.key
```

### 3) TLS

```bash
kubectl create secret tls my-tls \
  --cert=./tls.crt \
  --key=./tls.key
```

### 4) Docker registry (imagePullSecrets)

```bash
kubectl create secret docker-registry regcred \
  --docker-server=registry.example.com \
  --docker-username=myuser \
  --docker-password='mypassword' \
  --docker-email=me@example.com
```

## Використання в Pod

### 1) Як env змінні (конкретні ключі)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app
spec:
  containers:
    - name: app
      image: busybox:1.36
      command: ["sh", "-c", "echo $DB_USER; echo $DB_PASS; sleep 3600"]
      env:
        - name: DB_USER
          valueFrom:
            secretKeyRef:
              name: db-creds
              key: username
        - name: DB_PASS
          valueFrom:
            secretKeyRef:
              name: db-creds
              key: password
```

### 2) Як env змінні (усі ключі одразу)

```yaml
envFrom:
  - secretRef:
      name: db-creds
```

### 3) Як файли через volume

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app
spec:
  volumes:
    - name: secrets
      secret:
        secretName: app-key
  containers:
    - name: app
      image: busybox:1.36
      command: ["sh", "-c", "ls -la /secrets; sleep 3600"]
      volumeMounts:
        - name: secrets
          mountPath: /secrets
          readOnly: true
```

### 4) ImagePullSecrets (приватні image)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: private-app
spec:
  imagePullSecrets:
    - name: regcred
  containers:
    - name: app
      image: registry.example.com/team/app:1.0.0

## `kubernetes.io/dockerconfigjson` через YAML (як на скріні)

Іноді secret для registry роблять не через `kubectl create secret docker-registry ...`, а напряму YAML-ом.
Тоді використовується тип `kubernetes.io/dockerconfigjson` і ключ **`.dockerconfigjson`**.

Важливо:
- в `data` значення має бути **base64** від JSON
- ключ має називатися саме **`.dockerconfigjson`**
- тип має бути саме **`kubernetes.io/dockerconfigjson`**

Приклад (шаблон):

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: regcred
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: <BASE64_JSON>
```

Як отримати `<BASE64_JSON>`:

1) Створи файл `dockerconfigjson` (приклад структури):

```json
{
  "auths": {
    "https://index.docker.io/v1/": {
      "username": "<user>",
      "password": "<pass>",
      "auth": "<base64(user:pass)>"
    }
  }
}
```

2) Закодуй його в base64 і встав у YAML.

Після створення secret підключай так само, як і раніше, через `imagePullSecrets`.

### Варіант: підв’язати secret до ServiceAccount

Щоб не прописувати `imagePullSecrets` у кожному Pod/Deployment, можна додати його в ServiceAccount:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-sa
  namespace: default
imagePullSecrets:
  - name: regcred
```

Тоді всі Pod-и з `serviceAccountName: app-sa` зможуть тягнути приватні images.
```

## Корисні команди kubectl

```bash
kubectl get secret -A
kubectl describe secret <name> -n <namespace>

# Подивитись YAML (дані будуть base64)
kubectl get secret <name> -n <namespace> -o yaml

# Подивитись декодовані значення (обережно!)
kubectl get secret <name> -n <namespace> -o jsonpath='{.data.password}' | base64 -d
```

## Практичні поради

- Не коміть Secrets у git у відкритому вигляді.
- Використовуй мінімальні права (RBAC) — хто може читати secrets.
- Для production часто застосовують рішення на кшталт External Secrets / Vault / Sealed Secrets.

## Посилання

- https://kubernetes.io/docs/concepts/configuration/secret/
