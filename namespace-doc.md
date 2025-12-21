# Документація про Namespace у Kubernetes

## Опис
Namespace — це логічний простір імен у Kubernetes, який дозволяє розділяти ресурси кластера між командами, середовищами (dev/stage/prod) або проєктами.

## Навіщо використовувати Namespace
- Ізоляція ресурсів між проєктами/командами
- Зручна організація об’єктів (Pods, Services, Deployments тощо)
- Застосування політик на рівні простору імен (RBAC, ResourceQuota, NetworkPolicy)
- Уникнення конфліктів імен ресурсів (однакові назви в різних namespace)

## Вбудовані Namespace (типово)
- `default` — простір імен за замовчуванням
- `kube-system` — системні компоненти Kubernetes
- `kube-public` — публічно доступні дані (рідко використовується напряму)
- `kube-node-lease` — об’єкти lease для вузлів

## YAML-приклад Namespace
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: example-namespace
  labels:
    purpose: demo
```

## Як створити Namespace
- Через маніфест: `kubectl apply -f namespace.yaml`
- Однією командою: `kubectl create namespace example-namespace`

## Основні команди kubectl
- Переглянути namespace: `kubectl get namespaces`
- Переглянути ресурси в namespace: `kubectl get all -n <namespace>`
- Створити ресурс у namespace (приклад): `kubectl apply -f app.yaml -n <namespace>`
- Видалити namespace: `kubectl delete namespace <namespace>`

## Встановити namespace за замовчуванням для поточного контексту
```bash
kubectl config set-context --current --namespace=<namespace>
```

## Примітки
- Namespace не ізолює вузли (nodes) — це саме логічна ізоляція в API.
- Для реальної ізоляції трафіку та доступу використовуйте RBAC та NetworkPolicy.

## Додатково
Докладніше: https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/
