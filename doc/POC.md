# Proof of Concept

1. Підготуємо окремий локальний кластер та налаштуємо його:

```sh
$ k3d cluster create argo

...
INFO[0014] Cluster 'argo' created successfully!
INFO[0014] You can now use it like this: kubectl cluster-info

$ kubectl cluster-info
Kubernetes control plane is running at https://0.0.0.0:57045
CoreDNS is running at https://0.0.0.0:57045/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
Metrics-server is running at https://0.0.0.0:57045/api/v1/namespaces/kube-system/services/https:metrics-server:https/proxy
```

2. Створимо окремий неймспейс під потреби ArgoCD

```sh
$ kubectl create namespace argocd
namespace/argocd created
```

3. Скористаємось скриптом (маніфестом) для інсталяції та перевіримо стан системи після встановлення:

```sh
$ kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

$ kubectl get pod -n argocd -w

NAME                                               READY   STATUS    RESTARTS   AGE
argocd-application-controller-0                    1/1     Running   0          2m11s
argocd-applicationset-controller-65bb5ff89-2wspg   1/1     Running   0          2m11s
argocd-dex-server-69b469f8fb-4kkqm                 1/1     Running   0          2m11s
argocd-notifications-controller-64bc7c9f7-hk92s    1/1     Running   0          2m11s
argocd-redis-867d4785f-w7q8b                       1/1     Running   0          2m11s
argocd-repo-server-5744559fff-r2646                1/1     Running   0          2m11s
argocd-server-697df9f478-stl24                     1/1     Running   0          2m11s
```

4. Отримаємо доступ до інтерфейсу ArgoCD GUI:

Скористаємось Port Forwarding за допомогою локального порта 8080. В команді ми посилаємось на сервіс svc/argocd-server який знаходиться в namespace -n argocd. Kubectl автоматично знайде endpoint сервісу та встановить переадресацію портів з локального порту 8080 на віддалений 443

```sh
$ kubectl port-forward svc/argocd-server -n argocd 8080:443&

[1] 19102
Forwarding from 127.0.0.1:8080->8080
Forwarding from [::1]:8080 -> 8080
Handling connection for 8080
```

5. Отримаємо пароль

```sh
$ kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}"|base64 -d;echo
```

6. Отриманий пароль та логін admin вводимо в Web-інтерфейс ArgoCD за адресою 127.0.0.1:8080. Погодимось з використанням самопідписаного сертифікату