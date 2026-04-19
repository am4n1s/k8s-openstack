
# k8s-on-openstack

Автоматическое развёртывание Kubernetes кластера с помощью Ansible.

В локальной среде используется Minikube вместо OpenStack.

## Требования

- Ubuntu 22.04 / 24.04

- Docker

- Minikube

- kubectl

- Ansible 2.10+

## Установка зависимостей

```bash

pip3 install kubernetes --break-system-packages

ansible-galaxy collection install community.general kubernetes.core

```

## Как запустить

### Шаг 1 — Поднять кластер

```bash

ansible-playbook playbooks/provision.yml

```

### Шаг 2 — Установить Kubernetes компоненты

```bash

ansible-playbook playbooks/k8s-install.yml

```

### Шаг 3 — Проверить кластер

```bash

kubectl get nodes

kubectl get pods -A

```

### Удалить все ресурсы

```bash

ansible-playbook playbooks/cleanup.yml

```

## Структура проекта

- `roles/openstack-provision` — запуск Minikube

- `roles/k8s-prerequisites` — проверка инструментов

- `roles/k8s-master` — настройка master, установка addons

- `roles/k8s-worker` — деплой тестового приложения

## Результат

После запуска плейбуков кластер готов:

- 1 нода (control-plane)

- Namespace k8s-demo

- Nginx deployment (2 реплики)

- Ingress, Dashboard, Metrics-server addons

