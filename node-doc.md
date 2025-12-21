# Документація про Node у Kubernetes

## Що таке Node
Node (нода) — це **машина, на якій реально запускаються Pod’и/контейнери**.

Node має дві “частини”:
- **Реальна інфраструктура**: VM або фізичний сервер.
- **Kubernetes-об’єкт Node**: запис в API Server, який описує цю машину та її стан.

На worker-ноді зазвичай працюють ключові компоненти:
- **kubelet** — агент Kubernetes, який отримує інструкції (PodSpec) і запускає/контролює контейнери.
- **container runtime** — containerd або CRI-O (саме він створює й запускає контейнери).
- **kube-proxy** (або інший dataplane) — мережеві правила для Service.
- **CNI плагін** — налаштовує мережу Pod’ів.

## Master і Worker: що це означає
У старіших матеріалах часто зустрічається термін **master node**. У сучасній термінології Kubernetes правильніше казати **control plane**.

Спрощено:
- **Master / Control plane** — “мозок” кластера: приймає запити, зберігає стан, планує Pods.
- **Worker** — “виконавець”: саме тут реально запускаються Pods/контейнери.

### Що зазвичай працює на Master (Control plane)
- **kube-apiserver** — приймає kubectl/API запити.
- **etcd** — база стану кластера.
- **kube-scheduler** — вибирає Node для Pod’ів.
- **kube-controller-manager** — контролери (ReplicaSet/Deployment тощо).

### Що зазвичай працює на Worker
- **kubelet** + **container runtime** — запускають/контролюють контейнери.
- **CNI** + **kube-proxy** (або інший dataplane) — мережа Pod’ів і Service.

### Важлива примітка
- У managed Kubernetes (AKS/EKS/GKE) **control plane часто “керований” і не видимий як ваші ноди**.
- У self-managed кластерах control plane може бути на окремих нодах або (в навчальних кластерах) на тій же машині.

## Control plane vs Worker nodes
- **Control plane** (керуючі компоненти) — API Server, Scheduler, Controller Manager, etcd.
- **Worker nodes** — виконують робочі навантаження (Pods).

У багатьох managed-кластерах control plane прихований від вас, а ви керуєте лише worker-нодами (node pools).

## Коли створюється Node
Node з’являється **до** того, як на ньому можуть запускатися Pod’и.

З практичної точки зору є два кроки:
1) **Створюється машина** (VM/сервер) — це робить або ви, або хмарний провайдер/автоскейлер.
2) **Машина приєднується до кластера** — на ноді запускається kubelet, який реєструє Node в API Server.

Після реєстрації Scheduler може призначати на цю ноду Pod’и.

## Хто “створює” ноди (типові сценарії)
### 1) Managed Kubernetes (AKS/EKS/GKE тощо)
- Ви створюєте/масштабуєте **node pool**.
- Хмара створює/видаляє VM.
- kubelet на VM автоматично реєструє Node в кластері.
- Часто є Cluster Autoscaler, який додає/забирає ноди під навантаження.

### 2) Self-managed (on-prem/VM)
- Ви самі створюєте VM/сервер.
- Встановлюєте runtime + kubelet.
- Приєднуєте ноду до кластера (наприклад, `kubeadm join ...`).

## Як Pod потрапляє на Node (коротко)
- Pod створюється в API.
- Scheduler вибирає Node.
- kubelet на цьому Node створює sandbox/мережу/volumes і запускає initContainers → containers.

## Корисні команди kubectl
- Список нод: `kubectl get nodes`
- Деталі ноди: `kubectl describe node <node-name>`
- Показати ресурси/мітки: `kubectl get node <node-name> -o wide`
- Заборонити планування нових Pod’ів на ноду: `kubectl cordon <node-name>`
- Евакуювати Pod’и та підготувати до обслуговування: `kubectl drain <node-name> --ignore-daemonsets`
- Дозволити планування знову: `kubectl uncordon <node-name>`

## Часті питання
### Якщо Pod “живе в API”, то де він реально працює?
На Node. Pod як об’єкт — це опис. Реальне виконання — це процеси контейнерів на ноді.

### Чому Pod може бути Pending?
Часто через те, що **немає відповідної ноди** (ресурсів не вистачає, не підходять taints/tolerations, nodeSelector/affinity, немає вільних місць, тощо).

## Додатково
- Nodes: https://kubernetes.io/docs/concepts/architecture/nodes/
- kubelet: https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet/
- Cluster (загальна картина): [cluster-doc.md](cluster-doc.md)
