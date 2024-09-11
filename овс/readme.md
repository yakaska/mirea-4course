# Практика 2

Установить зависимости

```bash
sudo apt update
sudo apt upgrade -y

sudo apt install python3 python3-pip -y

sudo apt install ansible -y

ansible --version
```

Сгенерировать ключи, отправить на машины

```bash
ssh-keygen -t rsa -b 4096
ssh-copy-id user@remote_host

ssh user@remote_host
```

Сделать файл инвентарный файл для ansible

```bash
nano hosts.ini

[all]
node1 ansible_host=192.168.1.100 ansible_user=user
node2 ansible_host=192.168.1.101 ansible_user=user
[docker]
node1
node2
```

Проверить пинг

```bash
ansible all -m ping
```

Создание плейбука у установкой докера и пакета

```bash
mkdir ~/ansible-docker-setup
cd ~/ansible-docker-setup

nano install_docker_and_neofetch.yml
```

```yml
---
- name: Установка Docker и Neofetch на узлы
  hosts: docker
  become: true  # выполнение от имени root
  tasks:
    - name: Установка необходимых пакетов для Docker
      apt:
        name:
          - apt-transport-https
          - ca-certificates
          - curl
          - software-properties-common
        state: present
        update_cache: yes

    - name: Добавление GPG ключа Docker
      apt_key:
        url: https://download.docker.com/linux/ubuntu/gpg
        state: present

    - name: Добавление репозитория Docker
      apt_repository:
        repo: deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable
        state: present

    - name: Установка Docker
      apt:
        name: docker-ce
        state: latest

    - name: Убедиться, что Docker запущен
      service:
        name: docker
        state: started
        enabled: true

    - name: Установка Neofetch
      apt:
        name: neofetch
        state: present
        update_cache: yes

```

Запуск и проверка установки

```bash
ansible-playbook install_docker_and_neofetch.yml -i hosts


ansible docker -m shell -a "docker --version" -i hosts

ansible docker -m shell -a "neofetch" -i hosts
```

Установка nginx и шаблонизация

```yaml
---
- name: Установка и настройка Nginx на всех узлах
  hosts: all
  become: true  # Выполнение от имени root
  vars:
    server_name: "{{ inventory_hostname }}"
  tasks:
    - name: Обновление кэша пакетов
      apt:
        update_cache: yes

    - name: Установка UFW, если его нет
      apt:
        name: ufw
        state: present
      ignore_errors: yes

    - name: Установка Nginx
      apt:
        name: nginx
        state: present
      ignore_errors: yes

    - name: Запуск и включение автозапуска Nginx
      service:
        name: nginx
        state: started
        enabled: true

    - name: Настройка веб-страницы с использованием шаблона
      template:
        src: templates/index.html.j2  # Путь к шаблону Jinja2
        dest: /var/www/html/index.html
        owner: www-data
        group: www-data
        mode: '0644'

    - name: Открытие порта 80 для HTTP
      ufw:
        rule: allow
        port: 80
        proto: tcp
      when: ansible_facts['os_family'] == "Debian"
      ignore_errors: yes

    - name: Открытие порта 443 для HTTPS
      ufw:
        rule: allow
        port: 443
        proto: tcp
      when: ansible_facts['os_family'] == "Debian"
      ignore_errors: yes

    - name: Включение UFW
      ufw:
        state: enabled
      when: ansible_facts['os_family'] == "Debian"

    # Проверка, что Nginx запущен
    - name: Проверка статуса Nginx
      service_facts:

    - name: Убедиться, что Nginx запущен
      debug:
        msg: "Nginx запущен"
      when: ansible_facts.services['nginx.service'].state == 'running'

    # Проверка, что Nginx добавлен в автозапуск
    - name: Убедиться, что Nginx включен в автозагрузку
      debug:
        msg: "Nginx включен в автозагрузку"
      when: ansible_facts.services['nginx.service'].enabled

```

Файл шаблона

```bash
nano index.html.j2
```

```html
<!DOCTYPE html>
<html>
<head>
    <title>Добро пожаловать на сервер {{ server_name }}</title>
</head>
<body>
    <h1>Добро пожаловать на сервер {{ server_name }}</h1>
</body>
</html>
```

Запуск плейбука

```bash
ansible-playbook -i hosts.ini playbook.yml
```

Проверка nginx вручную

```bash
ansible docker -m shell -a "systemctl status nginx" -i hosts

ansible docker -m shell -a "systemctl is-enabled nginx" -i hosts
```
