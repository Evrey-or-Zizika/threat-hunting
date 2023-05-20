# Развертывание Threat intelligence Platform OpenCTI
## Цель работы 
1. Освоить базовые подходы процессов Threat Intelligence
2. Освоить современные инструменты развертывания контейнеризованных приложений
3. Получить навыки поиска информации об угрозах ИБ
## Ход выполнения работы
1. Файл кофигураций
    * Клонируем репозитарий
        ```
        ┌──(root㉿kali)-[~/opencti/docker]
        └─# git clone https://github.com/OpenCTI-Platform/docker.git 
        Cloning into 'docker'...
        remote: Enumerating objects: 577, done.
        remote: Counting objects: 100% (306/306), done.
        remote: Compressing objects: 100% (102/102), done.
        remote: Total 577 (delta 239), reused 261 (delta 204), pack-reused 271
        Receiving objects: 100% (577/577), 109.06 KiB | 1.12 MiB/s, done.
        Resolving deltas: 100% (371/371), done.
        ```
    * Генерируем UUID
        ```
        ┌──(root㉿kali)-[~/opencti/docker]
        └─# cat /proc/sys/kernel/random/uuid
        b0f3e13c-3049-4354-a1b1-2389e1013ef2
        ``` 
    
    * Увеличиваем виртуальную память
        ```
        ┌──(root㉿kali)-[~/opencti/docker]
        └─# sysctl -w vm.max_map_count=1048575
        vm.max_map_count = 1048575
        ```
2. Запуск OpenCTI
    * Запуск контейнера в фоновом режиме
    ```
    ┌──(root㉿kali)-[~/opencti/docker]
    └─# docker-compose up -d
    Creating network "docker-compose_default" with the default driver
    Creating docker-compose_rabbitmq_1      ... done
    Creating docker-compose_redis_1         ... done
    Creating docker-compose_elasticsearch_1 ... done
    Creating docker-compose_minio_1         ... done
    Creating docker-compose_opencti_1       ... done
    Creating docker-compose_connector-import-file-stix_1 ... done
    Creating docker-compose_connector-export-file-txt_1  ... done
    Creating docker-compose_connector-export-file-stix_1 ... done
    Creating docker-compose_connector-export-file-csv_1  ... done
    Creating docker-compose_worker_1                     ... done
    Creating docker-compose_worker_2                     ... done
    Creating docker-compose_worker_3                     ... done
    Creating docker-compose_connector-import-document_1  ... done
    ```
    После этого, можно перейти на `localhost:8088`
3. Переходим в веб-интерфейс OpenCTI
![All text](./screenshots/lab_4_0.png)
5. Зайходим на сайт с документацией по импорту файлов и видим, что существует библиотека python для работы с OpenCTI. 
   Напишем код для импорта файла в opencti:
    ```
    import pycti
    from stix2 import TLP_GREEN
    from datetime import datetime
    date = datetime.today().strftime("%Y-%m-%dT%H:%M:%SZ")

    api_url = 'http://localhost:8080'
    api_token = 'cedd95c3-744d-43ef-a300-2bc54a99baad'
    client = pycti.OpenCTIApiClient(api_url, api_token)

    TLP_GREEN_CTI = client.marking_definition.read(id=TLP_GREEN["id"])
    with open('hosts.txt', 'r') as f:
        domains = f.read().splitlines()
    k = 1
    for domain in domains:
        indicator = client.indicator.create(
        name="Malicious domain {}".format(k),
        description="MPVS hosts",
        pattern_type="stix",
        label="mpvs hosts",
        pattern="[domain-name:value = '{}']".format(domain),
        x_opencti_main_observable_type="IPv4-Addr",
        valid_from=date,
        update=True,
        score=75,
        markingDefinitions=[TLP_GREEN_CTI["id"]],
        )
        print("Создан индикатор:", indicator["id"])
        k += 1
    ```
    Ответ:
    ```
    ┌──(root㉿kali)-[~/opencti/docker]
    └─# python import.py
    INFO:pycti.entities:Listing Threat-Actors with filters null.
    Indicators imported successfully!
    ```
    ![All text](./screenshots/lab_4_1.png)
5. Преобразуем все индикаторы в Observables
![All text](./screenshots/lab_4_2.png)
![All text](./screenshots/lab_4_3.png)
7. Импортируем сетевой трафик, полученный в lab_2 в OpenCTI
![All text](./screenshots/lab_4_4.png)
9. Добавим файл в рабочую область
![All text](./screenshots/lab_4_5.png)
11. Переходим в раздел с анализом и фильтруем поиск по нежелательному траффик
    ![All text](./screenshots/lab_4_6.png)
    Отсюда выясним, что пользователь посетил 40 нежелательных доменов
## Оценка результата
С помощью платформы OpenCTI удалось проанализировать трафик на предмет перехода по нежелательным доменам.
# Выводы
Таким образом, были изучены возможности работы с платформой threat intelligence OpenCTI
