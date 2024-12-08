# Лабораторная работа №2

Ansible + Caddy

## Задача

На целевом сервере установить Ansible и вебсервер Caddy

## Ход работы

1. Локально устанавливаем pip менеджер и далее Ansible.

2. Создаем файлы конфигурации ansible.cfg и inventory/hosts. В inventory/hosts прописываем данные от удаленного сервера:
   ![img.png](static/lr2/img.png)
3. Подключаемся к клиенту командами ansible my_servers -m ping и ansible my_servers -m setup: 2.png
   ![img_1.png](static/lr2/img_1.png)
4. Инициализируем локально конфигурационное "дерево" и наполним файл с шагами, которые будут выполняться в плейбуке, а
   также добавим шаги создания и удаления файла:

```yaml
---
- name: Create test file
  ansible.builtin.shell:
    cmd: echo task_test_file_content > $HOME/test.txt

- name: Delete test file
  ansible.builtin.shell:
    cmd: rm $HOME/test.txt

- name: Install prerequisites
  apt:
    pkg:
      - debian-keyring
      - debian-archive-keyring
      - apt-transport-https
      - curl

- name: Add key for Caddy repo
  apt_key:
    url: https://dl.cloudsmith.io/public/caddy/stable/gpg.key
    state: present
    keyring: /usr/share/keyrings/caddy-stable-archive-keyring.gpg

- name: add Caddy repo
  apt_repository:
    repo: "deb [signed-by=/usr/share/keyrings/caddy-stable-archive-keyring.gpg] https://dl.cloudsmith.io/public/caddy/stable/deb/debian any-version main"
    state: present
    filename: caddy-stable

- name: add Caddy src repo
  apt_repository:
    repo: "deb-src [signed-by=/usr/share/keyrings/caddy-stable-archive-keyring.gpg] https://dl.cloudsmith.io/public/caddy/stable/deb/debian any-version main"
    state: present
    filename: caddy-stable

- name: Install Caddy webserver
  apt:
    name: caddy
    update_cache: yes
    state: present

- name: Create config file
  template:
    src: templates/Caddyfile.j2  # Откуда берем
    dest: /etc/caddy/Caddyfile  # Куда кладем

- name: Reload with new config
  service:
    name: caddy
    state: reloaded
```

5. Далее инициализируем файл конфигурации самого плейбука:

```yaml
---
- name: Install and configure Caddy webserver
  hosts: my_servers

  roles:
    - caddy_deploy
```

6. Создадим шаблон (Jinja2) и переменные для того, чтобы запустить Caddy. Также зарегистрируем
   домен https://kirmashmishi.duckdns.org.
7. Запустим плейбук и проверим работоспособность:
   ![img_2.png](static/lr2/img_2.png)