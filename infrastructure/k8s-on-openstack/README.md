# k8s-on-openstack

Автоматическое развёртывание Kubernetes кластера с помощью Ansible.
В локальной среде используется Minikube вместо OpenStack (недостаточно RAM для DevStack).

## Архитектура

- 1 master нода  (minikube)         — control-plane
- 2 worker ноды  (minikube-m02/m03) — workloads
- CNI: Flannel   (10.244.0.0/16)

## Требования

- Ubuntu 22.04 / 24.04
- Docker
- Minikube
- kubectl
- Ansible 2.10+
- Минимум 8 GB RAM

## Установка зависимостей
```
pip3 install kubernetes --break-system-packages
```

```
ansible-galaxy collection install community.general kubernetes.core
```
## Как запустить

### Шаг 1 — Поднять инфраструктуру
```
ansible-playbook playbooks/provision.yml
```
Автоматически запускает Minikube с 3 нодами и Flannel CNI.
Если кластер остановлен — запустит сам.

### Шаг 2 — Установить Kubernetes компоненты
```
ansible-playbook playbooks/k8s-install.yml
```
Устанавливает ingress, dashboard, metrics-server, деплоит nginx (2 реплики).

### Шаг 3 — Проверить кластер
```
kubectl get nodes
kubectl get pods -A
```
### Удалить все ресурсы
```
ansible-playbook playbooks/cleanup.yml
```
## Структура проекта
```
k8s-on-openstack/
├── ansible.cfg
├── inventory/hosts.ini
├── group_vars/all.yml           — все переменные
├── roles/
│   ├── openstack-provision/     — запуск Minikube (3 ноды + Flannel)
│   ├── k8s-prerequisites/       — проверка инструментов + загрузка образов
│   ├── k8s-master/              — настройка master, addons (ingress, dashboard)
│   └── k8s-worker/              — деплой nginx (2 реплики)
└── playbooks/
    ├── provision.yml
    ├── k8s-install.yml
    └── cleanup.yml
```
Условие                       | Реализация
------------------------------|--------------------------------------------
Сеть + subnet + router        | Внутренняя сеть Minikube 192.168.49.0/24
Security group                | Не требуется (локальная Docker сеть)
1 master + 2 worker VM        | minikube + minikube-m02 + minikube-m03
Floating IP для master        | minikube ip — 192.168.49.2
containerd                    | Встроен в Minikube ноды
kubeadm + kubelet + kubectl   | Установлены на всех нодах
CNI Flannel                   | kube-flannel-ds запущен на каждой ноде
Инициализация master          | kubeadm init (выполнен Minikube)
Подключение workers           | kubeadm join (выполнен Minikube)


## Результат после запуска


```text
NAME           STATUS   ROLES           VERSION
minikube       Ready    control-plane   v1.35.1
minikube-m02   Ready    <none>          v1.35.1
minikube-m03   Ready    <none>          v1.35.1
После запуска обоих плейбуков кластер полностью готов.
3 ноды в статусе Ready, поды распределены по нодам.
