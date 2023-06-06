# lab10
# Часть 1   

1. Для начала развернём две виртуальные машины.
2. На первой ВМ установим  Ansible и сгенерируем ssh-ключ.   

 ![image](https://github.com/lepeha81/lab10/blob/main/1.PNG)

**После генерации ключа у нас появился следующий файл:**   

 ![image](https://github.com/lepeha81/lab10/blob/main/2.PNG)   

3. Скопируем ssh ключ на VM2:

``` $ ssh-copy-id <username>@<VM2_IP>```     
**Первая часть выполнена**  

# Часть 2   

**Создадим необходимые файлы для работы Ansible**   
1. inventory   
```
[my_hosts]
VM1 ansible_host=<VM1_IP>
VM2 ansible_host=<VM2_IP>   
```
2. ansible.cfg   
```
[defaults]
inventory = inventory
```
- `[my_hosts]`: это имя группы хостов в инвентаре. 
- `VM1 ansible_host=<vm1_ip>`: это строка определения хоста, где `VM1` - это имя хоста, а `ansible_host` - это переменная Ansible, содержащая IP-адрес хоста.
- `VM2 ansible_host=<vm2_ip>`: это строка определения второго хоста. 
- `[defaults]`: это имя секции, содержащей определения по умолчанию для конфигурации Ansible. 
- `inventory = inventory`: это строка для задания пути к файлу инвентаря. Она указывает, что файл инвентаря находится в том же каталоге, что и конфигурационный файл Ansible.
- 
3. playbook.yml   
```
---
- name: Update packages and prepare VM2
  hosts: VM2
  tasks:
    - name: Update packages
      apt:
        upgrade: yes
        update_cache: yes

    - name: Install Docker dependencies
      apt:
        name: "{{ item }}"
        state: present
      loop:
        - apt-transport-https
        - ca-certificates
        - curl
        - software-properties-common

    - name: Add Docker GPG key
      apt_key:
        url: https://download.docker.com/linux/ubuntu/gpg
        state: present

    - name: Add Docker repository
      apt_repository:
        repo: deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable
        state: present

    - name: Install Docker
      apt:
        name: docker-ce
        state: present

    - name: Install Docker Compose
      pip:
        name: docker-compose
        state: present

    - name: Start Docker service
      service:
        name: docker
        state: started
```
Этот скрипт написан на языке YAML для автоматической инсталляции и настройки программного обеспечения на виртуальной машине (VM2). Давай построчно разберем, что происходит в каждой задаче (task):

1. `name: Update packages and prepare VM2` - Задаёт название задачи, которая будет выполняться.
2. `hosts: VM2` - Задаёт имя хоста (в данном случае - имя виртуальной машины), на котором будет выполняться задача.
3. `tasks:` - Задаёт начало списка задач, которые будут выполняться на указанном хосте.
4. `name: Update packages` - Задаёт название новой задачи, которая будет устанавливать последние версии пакетов.
5. `apt:` - Указывает на использование APT для установки и обновления пакетов.
6. `upgrade: yes` - Говорит APTу об обновлении всех установленных пакетов до последних версий.
7. `update_cache: yes` - Запрашивает обновление кэша репозитариев APT перед установкой новых пакетов.
8. `name: Install Docker dependencies` - Эта задача устанавливает зависимости Docker.
9. `apt:` - Cнова указывает на использование APT.
10. `name:` - Задаёт название задачи.
11. `loop:` - Указывает на использование цикла.
12. `item:` - Указывает на список элементов, которые будут использоваться в цикле.
13. `- apt-transport-https`, `- ca-certificates`, `- curl`, `- software-properties-common` - Объявляет список зависимостей, которые будут установлены на VM2.
14. `state: present` - Говорит APTу установить зависимости на VM2.
15. `name: Add Docker GPG key` - Задаёт название новой задачи, которая добавляет GPG-ключ для установки Docker.
16. `apt_key:` - Указывает на установку GPG-ключа через APT.
17. `url:` - Указывает URL, по которому можно загрузить GPG-ключ.
18. `state: present` - Говорит APTу установить GPG-ключ на VM2.
19. `name: Add Docker repository` - Заголовок новой задачи, добавляющей репозитарий Docker.
20. `apt_repository:` - Указывает на установку репозитария Docker через APT.
21. `repo:` - Указывает на репозитарий Docker, который нужно добавить. Добавляет репозитарий Docker для операционной системы Ubuntu версии 18.04 (bionic).
22. `name: Install Docker` - Название задачи, устанавливающей Docker.
23. `apt:` - Снова указывает на использование APT.
24. `name: docker-ce` - Указывает на имя пакета, который нужно установить через APT.
25. `state: present` - Говорит APTу установить пакет Docker на VM2.
26. `name: Install Docker Compose` - Добавляет новую задачу, устанавливающую Docker Compose.
27. `pip:` - Указывает на установку Docker Compose через Pip.
28. `name: docker-compose` - Указывает на имя пакета, который нужно установить через Pip.
29. `state: present` - Говорит Pip установить пакет Docker Compose на VM2.
30. `name: Start Docker service` - Заголовок задачи, запускающий сервис Docker.
31. `service:` - Указывает на запуск сервиса через systemd.
32. `name: docker` - Указывает на имя сервиса Docker, который следует запустить.
33. `state: started` - Говорит systemd запустить сервис Docker.

# Часть 3
1.Подключимся к VM2 по ssh. Нам необходимо узнать запущенные контейнеры, их статусы и их логи для этого:   
```
$ docker ps -a   
Команда docker ps -a, где a - сокращение от all выводит список всех контейнеров, независимо от их состояния.
```
```
docker logs <container_id>
```
Команда "docker logs" выводит логи, созданные контейнером Docker.

2. Отправим запрос на серверный контейнер.   
```
$ curl http://localhost:<port>
```
Эта команда отправляет HTTP-запрос на локальный сервер, используя указанный порт, и выводит ответ от сервера в терминал. Вместо `<port>` нужно указать номер порта, на котором запущен сервер. Эта команда может быть полезна для отладки серверных приложений и проверки ответов от сервера.








