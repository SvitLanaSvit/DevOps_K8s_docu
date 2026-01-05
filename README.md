# Kubernetes — конспект і документація

Цей репозиторій містить короткі навчальні нотатки про базові об’єкти Kubernetes: Pod, Service, Namespace, Node та архітектуру кластера.

## Швидкий старт
- Якщо ти просто читаєш: відкрий будь-який файл зі списку нижче.
- Якщо ти вивчаєш Kubernetes з нуля: йди за рекомендованим порядком.

## Рекомендований порядок читання
1) [cluster-doc.md](cluster-doc.md) — що таке кластер і як він влаштований
2) [node-doc.md](node-doc.md) — що таке нода (worker/control plane)
3) [helm-doc.md](helm-doc.md) — Helm (charts, releases, values)
4) [scheduling-doc.md](scheduling-doc.md) — як scheduler розміщує Pod (Affinity/AntiAffinity)
5) [pod-doc.md](pod-doc.md) — Pod та життєвий цикл
6) [containers-doc.md](containers-doc.md) — containers / initContainers і коли вони реально створюються
7) [deployment-doc.md](deployment-doc.md) — Deployment / ReplicaSet (replicas, rollout, rollback)
8) [daemonset-doc.md](daemonset-doc.md) — DaemonSet (по одному Pod на кожній Node)
9) [job-doc.md](job-doc.md) — Job (одноразові/батч задачі)
10) [cronjob-doc.md](cronjob-doc.md) — CronJob (Job за розкладом)
11) [configmap-doc.md](configmap-doc.md) — ConfigMap (key/value і як файли)
12) [storageclass-doc.md](storageclass-doc.md) — StorageClass (класи сховища для PVC/PV)
13) [statefulset-doc.md](statefulset-doc.md) — StatefulSet (stable DNS + stable storage)
14) [service-doc.md](service-doc.md) — Service і типи доступу
15) [namespace-doc.md](namespace-doc.md) — Namespace і логічна ізоляція

## Документи
- [cluster-doc.md](cluster-doc.md)
- [node-doc.md](node-doc.md)
- [helm-doc.md](helm-doc.md)
- [scheduling-doc.md](scheduling-doc.md)
- [pod-doc.md](pod-doc.md)
- [containers-doc.md](containers-doc.md)
- [deployment-doc.md](deployment-doc.md)
- [daemonset-doc.md](daemonset-doc.md)
- [job-doc.md](job-doc.md)
- [cronjob-doc.md](cronjob-doc.md)
- [configmap-doc.md](configmap-doc.md)
- [storageclass-doc.md](storageclass-doc.md)
- [statefulset-doc.md](statefulset-doc.md)
- [service-doc.md](service-doc.md)
- [namespace-doc.md](namespace-doc.md)

## Схеми
- Скріншот діаграми кластера (відображається в GitHub):
  - [cluster-diagram.png](./screenshots/cluster-diagram.png)
- Скріншот про Deployment → ReplicaSet → Pods:
  - [deployment.png](./screenshots/deployment.png)
- Скріншот про labels/selector у Deployment:
  - [deployment_name.png](./screenshots/deployment_name.png)
- Скріншот про зміну версії (rollout) у Deployment:
  - [deployment_change_version.png](./screenshots/deployment_change_version.png)
- Скріншот про DaemonSet (1 Pod на кожній Node):
  - [daemont_set.png](./screenshots/daemont_set.png)
- Скріншот про Job (одноразова задача в Pod):
  - [job.png](./screenshots/job.png)
- Скріншот про CronJob (schedule + jobTemplate):
  - [job_yaml.png](./screenshots/job_yaml.png)
- Скріншот про PVC/PV (static provisioning):
  - [pvc_pv.png](./screenshots/pvc_pv.png)
- Скріншот про StorageClass → PV (static provisioning):
  - [pvc.png](./screenshots/pvc.png)
- Скріншот про StatefulSet + headless Service + DNS:
  - [statfull_set.png](./screenshots/statfull_set.png)
- Скріншот з `kubectl get sc` (StorageClass):
  - [storage_class_result.png](./screenshots/storage_class_result.png)
- Скріншот про PodAffinity / PodAntiAffinity:
  - [scheduling-affinity.png](./screenshots/scheduling-affinity.png)
- Скріншот про Service + namespace + DNS:
  - [service-cross-namespace.png](./screenshots/service-cross-namespace.png)
- Скріншот про Service (ClusterIP + port-forward):
  - [service.png](./screenshots/service.png)

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
