# Презентация проекта с использованием Ansible-docker
## Введение
В этом проекте мы использовали инструменты Ansible, Docker и Docker Compose для создания и развертывания простого приложения на удаленном компьютере. Мы использовали Docker для создания контейнера приложения, Docker Compose для управления контейнерами и Ansible для автоматизации развертывания и настройки.

## Шаги проекта
1. **Написание Dockerfile:** Мы создали Dockerfile, который описывает, как создать образ Docker для нашего приложения.
2. **Написание docker-compose.yaml:** Мы создали файл docker-compose.yaml, который описывает, как запустить контейнеры Docker для нашего приложения.
3. **Написание простейшего приложения app.py:** Мы написали простой Python-скрипт, который выводит "Hello World!" на экран.
4. **Настройка Ansible:** Мы настроили Ansible для автоматизации развертывания и настройки нашего приложения на удаленном компьютере.
5. **Использование roles в Ansible:** Мы использовали роли в Ansible для разделения задач развертывания и настройки.
6. **Использование ansible vault:** Мы использовали ansible vault для защиты конфиденциальных данных, таких как пароли и ключи.
7. **Установка Docker на удаленном компьютере:** Мы установили Docker на удаленном компьютере, чтобы иметь возможность запускать контейнеры Docker.
8. **Запуск приложения Docker контейнера на удаленном компьютере:** Мы запустили контейнер Docker с нашим приложением на удаленном компьютере.
9. **Проверка запуска контейнеров:** Мы проверили, что контейнеры Docker успешно запущены и приложение работает корректно.
## Результаты
В данном разделе будут проанализированы промежуточные результаты, которые были получены в процессе выполнения задания.  
1. Написали простой Dockerfile для запуска фреймворка создания веб-приложений на языке программирования Python:  
```
FROM python:3.10-alpine
WORKDIR /code
ENV FLASK_APP=app.py
ENV FLASK_RUN_HOST=0.0.0.0
RUN apk add --no-cache gcc musl-dev linux-headers
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt
EXPOSE 5000
COPY . .
CMD ["flask", "run"]
```
2. Написали docker-compose.yaml для создания сети и системы хранения данных Redis:
```
services:
  web:
    build: .
    ports:
      - "8000:5000"
  redis:
    image: "redis:alpine"
```
3. Написали приложение, используя Python, для вывода на экран ```Hello World!```:
```
import time

import redis
from flask import Flask

app = Flask(__name__)
cache = redis.Redis(host='redis', port=6379)

def get_hit_count():
    retries = 5
    while True:
        try:
            return cache.incr('hits')
        except redis.exceptions.ConnectionError as exc:
            if retries == 0:
                raise exc
            retries -= 1
            time.sleep(0.5)

@app.route('/')
def hello():
    count = get_hit_count()
    return 'Hello World! I have been seen {} times.\n'.format(count)
```
4. На этапе настройки Ansible были созданы задачи (tasks), настроен конфигурационный файл Ansible (ansible.cfg), задачи были распределены по ролям, и для шифрования пароля учетной записи ```svc_ansible``` был использован ```vault```. Это позволило автоматизировать развертывание и настройку приложения на удаленном компьютере, а также защитить конфиденциальные данные. Ниже представлен tasks:
```
- name: Install dependencies
  apt:
   name: "{{item}}"
   state: present
  loop:
   - ca-certificates
   - curl
- name: Add GPG key
  apt_key:
    url: https://download.docker.com/linux/ubuntu/gpg
    state: present
- name: Install Docker
  apt:
   name: "{{item}}"
   state: present
  loop:
    - docker-ce
    - docker-ce-cli
    - containerd.io
    - docker-buildx-plugin
    - docker-compose-plugin
- name: Check if docker service is running
  service:
    name: docker
    state: started
    enabled: yes
  register: docker_service
- name: Print the status of docker service
  debug:
    var: docker_service.status
- name: Add svc_ansible user to docker group
  user:
    name: svc_ansible
    groups: docker
    append: yes

- name: Create directory
  file:
    path: "{{dest_folder}}"
    mode: 0755
    owner: svc_ansible
    group: svc_ansible
    state: directory

- name: Copy files to server
  copy:
   src: "{{ item }}"
   dest: "{{dest_folder}}"
   mode: 0755
  loop:
    - "Dockerfile"
    - "docker-compose.yaml"
    - "app.py"
    - "requirements.txt"
- name: Start Docker Compose application
  shell: docker compose -f /home/svc_ansible/docker_app/docker-compose.yaml up -d
- name: Check if container is running
  shell: docker ps | grep docker_app*
  register: output
  ignore_errors: true
- name: Print the status of docker container
  debug:
    var: output
```
5. В качестве удаленного компьютера мы использовали машину с именем docker-wireguard. При выполнении шага установки Docker мы получили следующий результат:  
![image](https://github.com/Virus-virus69/Ansible-docker/assets/145215499/187fb9a0-3979-41f8-a17d-70e9ec0cf99f)  
6. Следующие два шага были посвящены запуску приложения с помощью команды docker compose up -d и проверке успешного запуска контейнеров с помощью команды ```docker ps | grep docker_app*```:  
![image](https://github.com/Virus-virus69/Ansible-docker/assets/145215499/3656c889-9d36-4bdc-a42b-a5cac80cc12f)
7. В конечном итоге будет получено подтверждение о том, что все tasks были успешно выполнены:
![image](https://github.com/Virus-virus69/Ansible-docker/assets/145215499/b53edd09-52bd-4441-beda-8448a10b6d80)

## Заключение
В этом проекте мы продемонстрировали, как использовать инструменты Ansible, Docker и Docker Compose для создания и развертывания простого приложения на удаленном компьютере. Мы использовали Docker для создания контейнера приложения, Docker Compose для управления контейнерами и Ansible для автоматизации развертывания и настройки. Этот проект может быть использован как отправная точка для более сложных проектов, где требуется развертывание и настройка приложений на удаленных компьютерах.
