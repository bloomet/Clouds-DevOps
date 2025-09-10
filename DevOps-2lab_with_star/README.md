# Лабораторная №2 со звездочкой. Работа с Docker Compose
### Требования:
- Написать “плохой” Docker Compose файл, в котором есть не менее трех “bad practices” по их написанию.
- Написать “хороший” Docker Compose файл, в котором эти плохие практики исправлены.
- В Readme описать каждую из плохих практик в плохом файле, почему она плохая и как в хорошем она была исправлена, как исправление повлияло на результат.
- После предыдущих пунктов в хорошем файле настроить сервисы так, чтобы контейнеры в рамках этого compose-проекта так же поднимались вместе, но не "видели" друг друга по сети. В отчете описать, как этого добились и кратко объяснить принцип такой изоляции.
## 1. Создание "плохого" Docker Compose файла

Для начала установим Docker и Docker-compose:
```
sudo apt-get update
sudo apt-get install -y docker.io docker-compose
```
В рабочей директории создадим плохой Docker Compose файл с тремя плохими практиками:
- При использование `:latest` при каждом запуске загружается самая новая версия образа, что приводит к непредсказуемому поведению и нестабильности системы.
- `privileged: true` создают угрозу безопасности, давая доступ к имени хоста.
- Секреты хранятся в переменных средах, поэтому могут быть легко считаны из контейнера.
```
version: '3.8'

services:
  app1:
    image: ubuntu:latest
    command: bash -c "apt-get update && apt-get install -y curl && curl app2"
    privileged: true
    environment:
      - SECRET_KEY=secret1

  app2:
    image: ubuntu:latest
    command: bash -c "apt-get update && apt-get install -y curl && curl app1"
    privileged: true
    ports:
      - "80:8080"
```
Запустим файл:

![photo_2024-11-27_17-20-44](https://github.com/user-attachments/assets/a08c25d2-dcd6-4124-97ba-40657fba89fa)
## 2. Создание "хорошего" Docker Compose файла
Также в рабочей директории создадим хороший Docker Compose файл, где исправим плохие практики:
- Указываем конкретную версию образа.  Контейнеры будут использовать фиксированную версию образа, что исключает неожиданные изменения в поведении приложения.
- Уберем привилегии. Контейнеры будут работать в изолированной среде, что снижает риск взлома. 
- Перенесем секреты в Docker Secrets. Секреты будут храниться в защищенном месте и будут доступны только тем контейнерам, которым они необходимы, повышая безопасность системы.

```
version: '3.8'

services:
  app1:
    image: ubuntu:22.04
    command: bash -c "apt-get update && apt-get install -y curl && curl app2"
    privileged: false

  app2:
    image: ubuntu:22.04
    command: bash -c "apt-get update && apt-get install -y curl && curl app1"
    privileged: false
    ports:
      - "8080:8080"
    secrets:
      - secret_key

secrets:
  secret_key:
    file: ./secrets/secret_key.txt
```
Запустим файл:

![photo_2024-11-27_17-20-48](https://github.com/user-attachments/assets/90b5af86-7b07-4c1b-bf8c-2f3fff01ff9d)

## 3. Создание Docker Compose файла с сетевой изоляицией

Чтобы контейнеры не "видели" друга друга по сети, создадим новый Docker Compose файл, где добавим блок с объявлением сетей. Также подключим каждый контейнер к разным сетям.
```
version: '3.8'

services:
  app1:
    image: ubuntu:22.04
    command: bash -c "apt-get update && apt-get install -y curl && curl app2"
    privileged: false
    networks:
      - net1

  app2:
    image: ubuntu:22.04
    command: bash -c "apt-get update && apt-get install -y curl && curl app1"
    privileged: false
    ports:
      - "8080:8080"
    secrets:
      - secret_key
    networks:
      - net2

secrets:
  secret_key:
    file: ./secrets/secret_key.txt

networks:
  net1:
    driver: bridge
  net2:
    driver: bridge
```
Проверим действительно ли контейнеры изолированы друг от друга:

![image](https://github.com/user-attachments/assets/6be08025-2c8d-46c3-b6b2-779a1e015656)

Возникают ошибки `Could not resolve host`. Эти ошибки указывают на то, что контейнеры не могут найти друг друга по имени, так как они находятся в разных сетях.
