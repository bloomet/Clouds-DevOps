# Лабораторная №3

## Условие
1. Написать “плохой” CI/CD файл, который работает, но в нем есть не менее пяти “bad practices” по написанию CI/CD
2. Написать “хороший” CI/CD, в котором эти плохие практики исправлены
3. В Readme описать каждую из плохих практик в плохом файле, почему она плохая и как в хорошем она была исправлена, как исправление повлияло на результат
4. Прочитать историю про Васю (она быстрая, забавная и того стоит): https://habr.com/ru/articles/689234/

## Решение:

Итак, есть у нас CI/CD: 

```
on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v1 

      - name: Install dependencies
        run: |
          sudo apt-get update
          sudo apt-get install -y nodejs npm 
      - name: Lint code
        run: npm install eslint
           

      - name: Run tests
        run: npm test
```

Что он делает: 

1. Клонирует код из репозитория
   
2. Обновляет пакетный менеджер и устанавливает **nodejs** и **npm** на ВМ
   
3. Устанавливает **ESLint**

4. Запускает тесты

Как бы звучит всё достаточно легко, но здесь есть некоторые недостатки, например:

1. Используется **checkout@v1**, то есть ошибка заключается в том, что мы не используем актуальные версии

2. Следующий ужас на крыльях ночи заключается в установке записимостей:
   А) Использование **sudo**, т.к. могут возникнуть проблемы с предоставлением прав
   
   В) Могут установиться не те версии, которые мы хотим из - за **apt-get**
   
   С) **apt-get update** занимает ну очень много времени (Спойлер, в версии с хорошими практиками время build сократилось существенно), нужно найти альтернативу
   
   D) Ставим непонятно какие зависимости

3. Нет ограничения по времени для теста. Если что-то зависает или просто долго выполняется - должно прерываться

### Итого:
У нас появилось аж 6 проблем с нашим файлом

### Фикс:

```
on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-24.04

    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Set up Node.js
        uses: actions/setup-node@v2
        with:
          node-version: '16'

      - name: Cache npm dependencies
        uses: actions/cache@v3
        with:
          path: ~/.npm
          key: ${{ runner.os }}-node-${{ env.NPM_CACHE_VERSION }}
          restore-keys: |
            ${{ runner.os }}-node-

      - name: Install dependencies
        run: npm install

      - name: Lint code
        run: npm install eslint

      - name: Run tests
        run: npm test -- --timeout=5000
```
Что изменилось:

1. Мы используем только актуальные версии

2. Установили нужную нам версию **Node.js**, а также управляем кэшированием зависимостей

3. Используем **npm install** вместо **apt-get install**

**Время установки зависимостей у CI/CD с "плохими практиками":**

<img width="1855" height="872" alt="image" src="https://github.com/user-attachments/assets/1335ced2-b1dc-433d-b0f6-3ea0e8113356" />


**Время установки зависимостей у CI/CD с "хорошими практиками":**

<img width="1415" height="70" alt="image" src="https://github.com/user-attachments/assets/f5d7eb1c-d11f-4c12-b00a-9e60eae59121" />

**Разница огромна**

4. Больше не используем **sudo**

5. Поставили таймер на выполнение тестов

6. Установка npm зависимостей закэширована

7. Используется конкретная версия среды (OS), которая исполняет pipeline

P.S.: тесты workflow "плохих" и "хороших" практик соответственно:

1.

<img width="1865" height="874" alt="image" src="https://github.com/user-attachments/assets/b690270e-5a4e-4ebc-b298-66188da9f5bf" />

![image](https://github.com/user-attachments/assets/a0123a24-dd33-43f2-9941-8e8cf0886799)


2.


<img width="1868" height="643" alt="image" src="https://github.com/user-attachments/assets/d8aa026a-157e-4163-ab8e-545088fa9ef8" />

<img width="1788" height="673" alt="image" src="https://github.com/user-attachments/assets/9f207638-97e0-459a-bec2-5c3a4ce08415" />

Тесты проводились в приватном репозитории. В данном случае на скринах отличается время установки запвисимостей, т.к. ранее это уже проводилось, а основное изменение на скрине - кеширование. 

# That's all!
