# lab-05
otus | mysql cluster

### Домашнее задание
Развернуть InnoDB или PXC кластер

#### Цель:
Перевести базу веб-проекта на один из вариантов кластера MySQL: Percona XtraDB Cluster или InnoDB Cluster.

#### Описание/Пошаговая инструкция выполнения домашнего задания:
1. Разворачиваем отказоустойчивый кластер MySQL (PXC || Innodb) на ВМ или в докере любым способом
2. Создаем внутри кластера вашу БД для проекта

#### Критерии оценки:
Статус "Принято" ставится при выполнении описанного требования.


### Выполнение домашнего задания

#### Создание стенда

Стенд будем разворачивать с помощью Terraform на YandexCloud, настройку серверов будем выполнять с помощью Ansible.

Необходимые файлы размещены в репозитории GitHub по ссылке:
```
https://github.com/SergSha/lab-05.git
```

Схема:

<img src="pics/infra.png" alt="infra.png" />

Для начала получаем OAUTH токен:
```
https://cloud.yandex.ru/docs/iam/concepts/authorization/oauth-token
```

Настраиваем аутентификации в консоли:
```
export YC_TOKEN=$(yc iam create-token)
export TF_VAR_yc_token=$YC_TOKEN
```

Скачиваем проект с гитхаба:
```
git clone https://github.com/SergSha/lab-05.git && cd ./lab-05
```

В файле provider.tf нужно вставить свой 'cloud_id':
```
cloud_id  = "..."
```

При необходимости в файле main.tf вставить нужные 'ssh_public_key' и 'ssh_private_key', так как по умолчанию соответсвенно id_rsa.pub и id_rsa:
```
ssh_public_key  = "~/.ssh/id_rsa.pub"
ssh_private_key = "~/.ssh/id_rsa"
```

Для того чтобы развернуть стенд, нужно выполнить следующую команду:
```
terraform init && terraform apply -auto-approve && \
sleep 60 && ansible-playbook ./provision.yml
```

На всех серверах будут установлены ОС Almalinux 8, настроены смнхронизация времени Chrony, система принудительного контроля доступа SELinux, в качестве firewall будет использоваться NFTables.

Сервера nginx-01 и nginx-02 будут служить балансировщиками нагрузки для распределения пакетов между серверами backend-01 и backend-02. Для балансировки на серверах nginx-01 и nginx-02 будет использоватья приложение Nginx. 
Балансировку настроим в режиме Round Robin, с максимальном количеством ошибок - 2, и timeout - 90 секунд:
```
upstream backend {
        server {{ ip_address['backend-01'] }}:80 max_fails=2 fail_timeout=90s;
        server {{ ip_address['backend-02'] }}:80 max_fails=2 fail_timeout=90s;
}
```
