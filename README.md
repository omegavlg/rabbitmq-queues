# Домашнее задание к занятию "`Очереди RabbitMQ`" - `Дедюрин Денис`

---

## Задание 1. Установка RabbitMQ

Используя Vagrant или VirtualBox, создайте виртуальную машину и установите RabbitMQ. Добавьте management plug-in и зайдите в веб-интерфейс.

Итогом выполнения домашнего задания будет приложенный скриншот веб-интерфейса RabbitMQ.

### Ответ:

**Устанавливаем RabbitMQ:**
```
yum update -y
yum install -y epel-release
yum install -y erlang
yum install -y rabbitmq-server
```
**Запускаем:**
```
systemctl enable rabbitmq-server
systemctl start rabbitmq-server
```
**Активируем плагин управления RabbitMQ:**
```
rabbitmq-plugins enable rabbitmq_management
```
**Перезапускаем:**
```
systemctl restart rabbitmq-server
```
<img src = "img/01.png" width = 100%>

---
## Задание 2. Отправка и получение сообщений
Используя приложенные скрипты, проведите тестовую отправку и получение сообщения. Для отправки сообщений необходимо запустить скрипт producer.py.

Для работы скриптов вам необходимо установить Python версии 3 и библиотеку Pika. Также в скриптах нужно указать IP-адрес машины, на которой запущен RabbitMQ, заменив localhost на нужный IP.
```
$ pip install pika
```
Зайдите в веб-интерфейс, найдите очередь под названием hello и сделайте скриншот. После чего запустите второй скрипт consumer.py и сделайте скриншот результата выполнения скрипта

В качестве решения домашнего задания приложите оба скриншота, сделанных на этапе выполнения.

Для закрепления материала можете попробовать модифицировать скрипты, чтобы поменять название очереди и отправляемое сообщение.

### Ответ:

**Устанавливаем Python3:**
```
yum install -y python3
```
**Устанавливаем библиотеку pika:**
```
pip3 install pika
```
**Редактируем скрипты:**

Скрипт **producer.py**
```
#!/usr/bin/env python
# coding=utf-8
import pika

connection = pika.BlockingConnection(pika.ConnectionParameters('192.168.58.101'))
channel = connection.channel()

channel.queue_declare(queue='hello')

channel.basic_publish(exchange='', routing_key='hello', body='Hello Netology!')
print(" [x] Sent 'Hello Netology!'")
connection.close()
```
Скрипт **consumer.py**
```
#!/usr/bin/env python
# coding=utf-8
import pika

connection = pika.BlockingConnection(pika.ConnectionParameters('192.168.58.101'))
channel = connection.channel()

channel.queue_declare(queue='hello')

def callback(ch, method, properties, body):
    print(" [x] Received %r" % body)

channel.basic_consume(queue='hello', on_message_callback=callback, auto_ack=True)

print(' [*] Waiting for messages. To exit press CTRL+C')
channel.start_consuming()
```
Выполняем скрипт **producer.py**
<img src = "img/02.png" width = 100%>

Скриншот из web-интерфейса **RabbitMQ**:
<img src = "img/03.png" width = 100%>

---
## Задание 3. Подготовка HA кластера
Используя Vagrant или VirtualBox, создайте вторую виртуальную машину и установите RabbitMQ. Добавьте в файл hosts название и IP-адрес каждой машины, чтобы машины могли видеть друг друга по имени.

Пример содержимого hosts файла:
```
$ cat /etc/hosts
192.168.0.10 rmq01
192.168.0.11 rmq02
```
После этого ваши машины могут пинговаться по имени.

Затем объедините две машины в кластер и создайте политику ha-all на все очереди.

В качестве решения домашнего задания приложите скриншоты из веб-интерфейса с информацией о доступных нодах в кластере и включённой политикой.

Также приложите вывод команды с двух нод:
```
$ rabbitmqctl cluster_status
```
Для закрепления материала снова запустите скрипт producer.py и приложите скриншот выполнения команды на каждой из нод:
```
$ rabbitmqadmin get queue='hello'
```
После чего попробуйте отключить одну из нод, желательно ту, к которой подключались из скрипта, затем поправьте параметры подключения в скрипте consumer.py на вторую ноду и запустите его.

Приложите скриншот результата работы второго скрипта.

---
## Задание 4*. Ansible playbook
Напишите плейбук, который будет производить установку RabbitMQ на любое количество нод и объединять их в кластер. При этом будет автоматически создавать политику ha-all.

Готовый плейбук разместите в своём репозитории.