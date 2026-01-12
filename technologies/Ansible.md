# Ansible

> Платформа для автоматизации IT-задач и управления конфигурациями

## 🎯 Почему это важно

Ansible - один из самых популярных инструментов для автоматизации и управления конфигурациями. В отличие от других инструментов, Ansible не требует установки агентов на управляемые хосты и использует простой YAML синтаксис.

## 📋 Необходимые знания перед изучением

**ОБЯЗАТЕЛЬНО нужно знать:**
- [[Linux]] - командная строка, файловая система
- [[SSH и удаленный доступ]] - Ansible работает через SSH
- [[Bash]] - понимание скриптинга помогает
- YAML синтаксис - основа Ansible

**Полезно знать:**
- [[Python для SysAdmin]] - Ansible написан на Python
- [[Git]] - версионирование Ansible кода

## 📊 Уровни владения

### [[Middle SysAdmin]] уровень

#### Основы Ansible

**Архитектура:**
- Control node - машина с Ansible
- Managed nodes - управляемые хосты
- Agentless - не требует агентов
- Push-based модель

**Установка:**
```bash
# Ubuntu/Debian
apt install ansible

# RHEL/CentOS
yum install ansible

# Через pip
pip install ansible
```

#### Inventory файл

**Простой inventory (INI формат):**
```ini
# /etc/ansible/hosts или inventory.ini

[webservers]
web1.example.com
web2.example.com

[databases]
db1.example.com
db2.example.com

[production:children]
webservers
databases

[webservers:vars]
ansible_user=deploy
ansible_port=22
```

**YAML формат:**
```yaml
all:
  children:
    webservers:
      hosts:
        web1.example.com:
        web2.example.com:
      vars:
        ansible_user: deploy
    databases:
      hosts:
        db1.example.com:
```

#### Ad-hoc команды

```bash
# Проверка доступности хостов
ansible all -m ping

# Выполнение команды на всех хостах
ansible webservers -a "uptime"

# Копирование файла
ansible webservers -m copy -a "src=/tmp/file dest=/tmp/file"

# Установка пакета
ansible webservers -m apt -a "name=nginx state=present" -b

# Перезапуск службы
ansible webservers -m service -a "name=nginx state=restarted" -b
```

Флаги:
- `-m` - модуль
- `-a` - аргументы модуля
- `-b` - become (sudo)
- `-K` - запросить sudo пароль

#### Playbooks

**Простой playbook:**
```yaml
---
# webserver.yml
- name: Установка и настройка Nginx
  hosts: webservers
  become: yes
  
  tasks:
    - name: Установить Nginx
      apt:
        name: nginx
        state: present
        update_cache: yes
    
    - name: Скопировать конфигурацию
      copy:
        src: nginx.conf
        dest: /etc/nginx/nginx.conf
      notify: Restart Nginx
    
    - name: Убедиться что Nginx запущен
      service:
        name: nginx
        state: started
        enabled: yes
  
  handlers:
    - name: Restart Nginx
      service:
        name: nginx
        state: restarted
```

**Запуск playbook:**
```bash
ansible-playbook webserver.yml
ansible-playbook webserver.yml --check  # Dry run
ansible-playbook webserver.yml -vvv     # Verbose
```

#### Основные модули

**Управление пакетами:**
- `apt` - Debian/Ubuntu
- `yum` / `dnf` - RHEL/CentOS
- `package` - универсальный

**Файлы и директории:**
- `copy` - копирование файлов
- `file` - создание файлов/директорий
- `template` - Jinja2 шаблоны
- `lineinfile` - изменение строк в файлах

**Службы:**
- `service` / `systemd` - управление службами

**Команды:**
- `command` - простые команды
- `shell` - команды с shell функциями
- `script` - выполнение скриптов

**Пользователи:**
- `user` - управление пользователями
- `authorized_key` - SSH ключи

#### Variables и Facts

**Переменные:**
```yaml
---
- name: Пример с переменными
  hosts: webservers
  vars:
    http_port: 80
    app_name: myapp
  
  tasks:
    - name: Вывести переменную
      debug:
        msg: "Port is {{ http_port }}"
```

**Внешний файл переменных:**
```yaml
# vars/main.yml
http_port: 80
app_name: myapp

# playbook
- name: Playbook
  hosts: webservers
  vars_files:
    - vars/main.yml
```

**Facts (автоматически собираемая информация):**
```yaml
- name: Использование facts
  hosts: all
  tasks:
    - name: Показать IP адрес
      debug:
        msg: "IP: {{ ansible_default_ipv4.address }}"
    
    - name: Показать ОС
      debug:
        msg: "OS: {{ ansible_distribution }} {{ ansible_distribution_version }}"
```

#### Templates (Jinja2)

**Шаблон nginx.conf.j2:**
```jinja
server {
    listen {{ http_port }};
    server_name {{ server_name }};
    
    location / {
        proxy_pass http://{{ backend_host }}:{{ backend_port }};
    }
}
```

**Использование:**
```yaml
- name: Развернуть конфигурацию из шаблона
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/sites-available/{{ app_name }}.conf
```

#### Conditions и Loops

**Условия:**
```yaml
- name: Установить на Ubuntu
  apt:
    name: nginx
  when: ansible_distribution == "Ubuntu"

- name: Установить на CentOS
  yum:
    name: nginx
  when: ansible_distribution == "CentOS"
```

**Циклы:**
```yaml
- name: Создать несколько пользователей
  user:
    name: "{{ item }}"
    state: present
  loop:
    - alice
    - bob
    - charlie

- name: Установить пакеты
  apt:
    name: "{{ item }}"
  loop:
    - nginx
    - postgresql
    - redis
```

### [[Senior SysAdmin]] уровень

#### Roles - Структура проекта

**Создание роли:**
```bash
ansible-galaxy init webserver
```

**Структура роли:**
```
webserver/
├── defaults/       # Переменные по умолчанию
│   └── main.yml
├── files/          # Статические файлы
├── handlers/       # Handlers (перезапуск служб)
│   └── main.yml
├── meta/           # Метаданные и зависимости
│   └── main.yml
├── tasks/          # Основные задачи
│   └── main.yml
├── templates/      # Jinja2 шаблоны
├── tests/          # Тесты
│   └── test.yml
└── vars/           # Переменные роли
    └── main.yml
```

**Пример tasks/main.yml:**
```yaml
---
- name: Установить Nginx
  apt:
    name: nginx
    state: present
  tags: packages

- name: Скопировать конфигурацию
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
  notify: Restart Nginx
  tags: config

- name: Убедиться что Nginx запущен
  service:
    name: nginx
    state: started
    enabled: yes
  tags: service
```

**Использование роли:**
```yaml
---
- name: Настройка веб-сервера
  hosts: webservers
  roles:
    - webserver
    - { role: ssl, when: use_ssl }
```

#### Ansible Galaxy

```bash
# Поиск ролей
ansible-galaxy search nginx

# Установка роли
ansible-galaxy install geerlingguy.nginx

# Установка из requirements.yml
# requirements.yml:
roles:
  - name: geerlingguy.nginx
  - src: https://github.com/user/role.git
    version: master

ansible-galaxy install -r requirements.yml
```

#### Vault - Управление секретами

```bash
# Создать зашифрованный файл
ansible-vault create secrets.yml

# Редактировать
ansible-vault edit secrets.yml

# Зашифровать существующий файл
ansible-vault encrypt vars.yml

# Расшифровать
ansible-vault decrypt vars.yml

# Использование в playbook
ansible-playbook site.yml --ask-vault-pass
# или
ansible-playbook site.yml --vault-password-file ~/.vault_pass
```

**Пример секретов:**
```yaml
# secrets.yml
db_password: "supersecret123"
api_key: "abc123xyz"
```

#### Dynamic Inventory

**AWS EC2 динамический inventory:**
```yaml
# aws_ec2.yml
plugin: aws_ec2
regions:
  - us-east-1
filters:
  tag:Environment: production
keyed_groups:
  - key: tags.Role
    prefix: role
```

**Использование:**
```bash
ansible-playbook -i aws_ec2.yml playbook.yml
```

#### Ansible для различных платформ

**Windows:**
```yaml
- name: Управление Windows
  hosts: windows
  tasks:
    - name: Установить IIS
      win_feature:
        name: Web-Server
        state: present
    
    - name: Копировать файл
      win_copy:
        src: file.txt
        dest: C:\temp\file.txt
```

**Сетевое оборудование:**
```yaml
- name: Настройка Cisco
  hosts: routers
  connection: network_cli
  tasks:
    - name: Сохранить конфигурацию
      ios_command:
        commands:
          - write memory
```
- Связано с: [[Сетевое оборудование]]

#### Best Practices и продвинутые техники

**Структура проекта:**
```
ansible-project/
├── inventory/
│   ├── production/
│   │   ├── hosts.yml
│   │   └── group_vars/
│   └── staging/
│       ├── hosts.yml
│       └── group_vars/
├── roles/
│   ├── common/
│   ├── webserver/
│   └── database/
├── playbooks/
│   ├── site.yml
│   ├── webservers.yml
│   └── databases.yml
├── group_vars/
│   └── all.yml
├── host_vars/
├── ansible.cfg
└── requirements.yml
```

**ansible.cfg:**
```ini
[defaults]
inventory = inventory/production/hosts.yml
roles_path = roles
host_key_checking = False
retry_files_enabled = False
gathering = smart
fact_caching = jsonfile
fact_caching_connection = /tmp/ansible_facts

[privilege_escalation]
become = True
become_method = sudo
become_user = root
become_ask_pass = False
```

**Теги для выборочного выполнения:**
```yaml
- name: Полный playbook
  hosts: all
  tasks:
    - name: Обновить пакеты
      apt:
        update_cache: yes
      tags: packages
    
    - name: Настроить firewall
      ufw:
        rule: allow
        port: '22'
      tags: security
```

```bash
# Выполнить только security задачи
ansible-playbook site.yml --tags security

# Пропустить security задачи
ansible-playbook site.yml --skip-tags security
```

**Делегирование и локальные действия:**
```yaml
- name: Действие на другом хосте
  command: some_command
  delegate_to: localhost

- name: Локальное действие
  local_action:
    module: command
    cmd: echo "Running locally"
```

**Blocks для группировки и обработки ошибок:**
```yaml
- name: Попытка с rescue
  block:
    - name: Опасная операция
      command: /bin/false
  rescue:
    - name: Обработка ошибки
      debug:
        msg: "Произошла ошибка, откатываем..."
  always:
    - name: Всегда выполняется
      debug:
        msg: "Cleanup действия"
```

#### CI/CD Integration

```yaml
# .gitlab-ci.yml
ansible-deploy:
  stage: deploy
  script:
    - ansible-playbook -i inventory/production playbooks/deploy.yml
  only:
    - master
```

Связано с: [[CI-CD]]

#### Ansible Tower / AWX

- Web UI для Ansible
- RBAC (Role-Based Access Control)
- Job scheduling
- Централизованное логирование
- REST API

#### Testing

**Ansible-lint:**
```bash
ansible-lint playbook.yml
```

**Molecule для тестирования ролей:**
```bash
# Установка
pip install molecule molecule-docker

# Инициализация
molecule init role my-role

# Тестирование
molecule test
```

## 🔗 Связанные технологии

### Обязательные зависимости
- [[Linux]] - основная платформа
- [[SSH и удаленный доступ]] - транспорт для Ansible

### Дополняющие технологии
- [[Git]] - версионирование Ansible кода
- [[Python для SysAdmin]] - написание кастомных модулей
- [[Docker]] - управление контейнерами через Ansible
- [[Kubernetes]] - деплой в K8s через Ansible

### Альтернативы
- **Puppet** - более сложный, с агентами
- **Chef** - Ruby-based
- **SaltStack** - быстрее, но сложнее
- **Terraform** - больше для provisioning инфраструктуры ([[Infrastructure as Code]])

## 📚 Ресурсы для изучения

### Официальные
- docs.ansible.com - отличная документация
- Ansible Galaxy - готовые роли

### Книги
- "Ansible for DevOps" by Jeff Geerling
- "Ansible: Up and Running" by Lorin Hochstein

### Курсы
- Ansible для начинающих (Udemy)
- Red Hat Certified Specialist in Ansible Automation

### Практика
- Ansible Playground
- Собственная домашняя лаборатория

## 💡 Best Practices

1. **Используй роли** для переиспользования кода
2. **Версионируй в Git** все playbook'и
3. **Используй Vault** для секретов
4. **Пиши идемпотентные playbook'и** - можно запускать много раз
5. **Тестируй в staging** перед production
6. **Используй теги** для гранулярного контроля
7. **Документируй роли** - README.md в каждой роли
8. **Используй ansible-lint** для проверки кода
9. **Ограничивай с --limit** при работе с production

## 🎯 Практические задачи

### Middle уровень
- [ ] Установить Ansible локально
- [ ] Создать inventory с несколькими группами
- [ ] Написать playbook для установки веб-сервера
- [ ] Использовать шаблоны для конфигураций
- [ ] Создать handlers для перезапуска служб
- [ ] Использовать переменные и loops

### Senior уровень
- [ ] Создать структуру с ролями
- [ ] Настроить Ansible Vault
- [ ] Написать роль и опубликовать в Galaxy
- [ ] Настроить динамический inventory (AWS/Azure)
- [ ] Интегрировать с CI/CD
- [ ] Написать кастомный модуль на Python
- [ ] Настроить Molecule для тестирования
- [ ] Реализовать zero-downtime deployment

## 🚨 Типичные ошибки

1. **Не идемпотентные playbook'и** - результат зависит от количества запусков
2. **Hardcoded значения** - используй переменные
3. **Нет тестирования** - всегда тестируй в staging
4. **Большие монолитные playbook'и** - используй роли
5. **Секреты в plain text** - используй Vault
6. **Не используются tags** - сложно выполнить часть playbook'а
7. **Игнорирование return codes** - не проверяют результаты

## 📈 Когда использовать Ansible

### ✅ Хорошие случаи
- Configuration management
- Application deployment
- Infrastructure provisioning
- Orchestration сложных задач
- Patching и updates

### ⚠️ Ограничения
- Медленнее чем SaltStack для больших инфраструктур
- Требует Python на управляемых хостах
- Масштабируется хуже чем решения с агентами

---

**Связанные уровни:**
- 📈 [[Middle SysAdmin]] - начинаешь изучать Ansible здесь (ОБЯЗАТЕЛЬНО)
- 🚀 [[Senior SysAdmin]] - продвинутое использование Ansible

**Требует знания:**
- [[Linux]] - обязательно
- [[SSH и удаленный доступ]] - обязательно
- [[Bash]] - полезно

#технология #ansible #automation #configuration-management #iac #devops
