## Техническое задание 
1. Написать “плохой” CI/CD файл, который работает, но в нем есть не менее пяти “bad practices” по написанию CI/CD
2. Написать “хороший” CI/CD, в котором эти плохие практики исправлены
3. В Readme описать каждую из плохих практик в плохом файле, почему она плохая и как в хорошем она была исправлена, как исправление повлияло на результат
4. Прочитать историю про Васю (она быстрая, забавная и того стоит): https://habr.com/ru/articles/689234/

> Плохие практики 
```yaml
name: BadPractices

on:
  - push
  - pull_request

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-python@v3
        with:
          python-version: '3.9'
      - run: pip install pytest
      - run: pytest testing/
      - run: ...
```

> Исправленная версия
```yaml
name: Good Practices

on:
  push:
    branches:
      - master
      - dev
  pull_request:
    branches:
      - master
#      - ...

jobs:
  build:
    runs-on: ubuntu-22.04
    steps:
      - name: check code
        uses: actions/checkout@v3
      - name: set up python
        uses: actions/setup-python@v3
        with:
          python-version: '3.9'
      - name: install deps
        run: pip install pytest
  test:
    runs-on: ubuntu-22.04
    needs:
      - build
    steps:
      - name: run tests
        run: pytest testing/
  deploy:
    runs-on: ubuntu-22.04
    needs:
      - build
      - test
    steps: deploy
#    ...
```
### Ошибка 1 - запускать без разделения на ветки 
```yaml
on:
  - push
  - pull_request
```
В таком коде пайплайн будет запускаться при каждом `push` и `pull_request`, что может быть не нужным(например не в `main`, `dev` или других не рабочих версиях). Для избежания частого запуска, укажем в каких ветках и при каких действиях он будет запускаться.

```yaml
on:
  push:
    branches:
      - master
      - dev 
  pull_request:
    branches:
      - master
      - ...
```

### Ошибка 2 - Использование последней версии ОС
```yaml
runs-on: ubuntu-latest
```
При использовании ubuntu-latest будет загружаться последняя версия образа. Т.к. со временем будут выходить новые версии, может возникнуть конфликт версий, поэтому лучше написать конкретную версию. 
```yaml
runs-on: ubuntu-22.04
```

### Ошибка 3 - не использование зависимостей
Некоторые этапы зависят от других, для того, что бы избежать запуска `test`, когда не пройден `build` добавим следующие строчки:
```yaml
  test:
    needs:
      - build

  deploy:
    needs:
      - build
      - test
```
### Ошибка 4 - нет `name`
В исправленном файле для каждого процесса добавляем `name`. Это может помочь быстрее поймать ошибку и определить, на каком этапе она падает

Так же это делает код более читаемым

Например
```yaml
    - name: check code
      uses: actions/checkout@v3
```

### Ошибка 5 - нет разделения `jobs`

```yaml
jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: pip install pytest
      - run: pytest testing/
      - run: ...
```
Если нет разделения на этапы - `build`, `test` и тд, код становится менее читаемым, сложнее разобраться при ошибках, какой именно этап не пройден. 

```yaml
jobs:
  build:
    ...
  test:
    ...
  deploy:
    ...
```
