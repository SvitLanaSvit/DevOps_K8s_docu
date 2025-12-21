# Документація про ConfigMap у Kubernetes

## Що таке ConfigMap
ConfigMap — це об’єкт Kubernetes для зберігання **несекретної** конфігурації.

Дані в ConfigMap — це пари **key → value**. Значення часто зберігають як:
- окремі текстові значення (рядки)
- “файли” (коли key — назва файлу, а value — вміст файлу)

> Для секретів (паролі/токени/ключі) використовуйте **Secret**, а не ConfigMap.

## Варіант 1: key/value (env або окреме значення)
### YAML-приклад ConfigMap
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  LOG_LEVEL: "info"
  FEATURE_X_ENABLED: "true"
```

### Як підключити в Pod як env
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app
spec:
  containers:
    - name: app
      image: nginx:latest
      env:
        - name: LOG_LEVEL
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: LOG_LEVEL
```

### Як підключити ВСІ ключі одразу (envFrom)
```yaml
spec:
  containers:
    - name: app
      image: nginx:latest
      envFrom:
        - configMapRef:
            name: app-config
```

## Варіант 2: як “файли” (volume)
Ідея: key = ім’я файлу, value = вміст. Kubernetes створить файли в змонтованій директорії.

### YAML-приклад ConfigMap з файлами
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-files
data:
  app.conf: |
    server {
      listen 8080;
      location / {
        return 200 'ok';
      }
    }
  settings.json: |
    {
      "featureX": true,
      "logLevel": "info"
    }
```

### Підключити ConfigMap як volume
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app
spec:
  volumes:
    - name: config
      configMap:
        name: app-files
  containers:
    - name: app
      image: nginx:latest
      volumeMounts:
        - name: config
          mountPath: /etc/app
          readOnly: true
```

Після цього всередині контейнера з’являться:
- `/etc/app/app.conf`
- `/etc/app/settings.json`

### Взяти тільки один ключ як окремий файл (items)
```yaml
spec:
  volumes:
    - name: config
      configMap:
        name: app-files
        items:
          - key: app.conf
            path: nginx.conf
```

## Корисні команди kubectl
- Створити/оновити: `kubectl apply -f configmap.yaml`
- Подивитись: `kubectl get configmaps -n <namespace>`
- Деталі: `kubectl describe configmap <name> -n <namespace>`
- Вивести YAML: `kubectl get configmap <name> -n <namespace> -o yaml`

## Примітки
- ConfigMap живе в конкретному **namespace**.
- Обмеження на розмір існують (для великих файлів краще артефакти/обʼєктне сховище).

## Додатково
- ConfigMaps: https://kubernetes.io/docs/concepts/configuration/configmap/
