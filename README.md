# Kubernetes — конспект і документація

Цей репозиторій містить короткі навчальні нотатки про базові об’єкти Kubernetes: Pod, Service, Namespace, Node та архітектуру кластера.

## Швидкий старт
- Якщо ти просто читаєш: відкрий будь-який файл зі списку нижче.
- Якщо ти вивчаєш Kubernetes з нуля: йди за рекомендованим порядком.

## Рекомендований порядок читання
1) [cluster-doc.md](cluster-doc.md) — що таке кластер і як він влаштований
2) [node-doc.md](node-doc.md) — що таке нода (worker/control plane)
3) [pod-doc.md](pod-doc.md) — Pod та життєвий цикл
4) [containers-doc.md](containers-doc.md) — containers / initContainers і коли вони реально створюються
5) [service-doc.md](service-doc.md) — Service і типи доступу
6) [namespace-doc.md](namespace-doc.md) — Namespace і логічна ізоляція

## Документи
- [cluster-doc.md](cluster-doc.md)
- [node-doc.md](node-doc.md)
- [pod-doc.md](pod-doc.md)
- [containers-doc.md](containers-doc.md)
- [service-doc.md](service-doc.md)
- [namespace-doc.md](namespace-doc.md)

## Схеми
- Скріншот діаграми кластера (відображається в GitHub):
  - [cluster-diagram.png](cluster-diagram.png)

## Папки
- [screenshots/](screenshots/) — вихідні скріншоти

## Корисні команди kubectl (мінімум)
```bash
kubectl get nodes
kubectl get ns
kubectl get pods -A
kubectl get svc -A
kubectl describe pod <pod-name> -n <namespace>
```

## Посилання на офіційну документацію
- Architecture: https://kubernetes.io/docs/concepts/architecture/
- Pods: https://kubernetes.io/docs/concepts/workloads/pods/
- Services: https://kubernetes.io/docs/concepts/services-networking/service/
- Namespaces: https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/
- Nodes: https://kubernetes.io/docs/concepts/architecture/nodes/
