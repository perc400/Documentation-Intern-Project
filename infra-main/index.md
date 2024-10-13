# [infra-main]

## Общая информация
 - Имя хоста: infra-main 
 - Операционная система: Rocky Linux 8.10 (Green Obsidian)
 - Ядро: Linux 4.18.0-553.el8_10.x86_64   
 - Архитектура: x86-64
 - IP-адрес: 192.168.140.136

## Сервисы
  1. docker.service
  - Информация
    ```
    Листинг
    ```
  2. node_exporter.service
  - Информация
    ```
    Листинг
    ```

## Контейнеры
  1. GitLab CE
  - Информация
    ```
    Листинг
    ```
  - Конфигурация [gitlab.rb]
    ```
    Листинг
    ```
  2. Docker Registry
  - Информация
    ```
    Листинг
    ```
  3. Cadvisor
  - Информация
    ```
    Листинг
    ```

## docker-compose.yml
  - Описание
    ```
    Листинг
    ```

## GitLab
  - Репозитории
    1. registration-publisher-service
       - Dockerfile
         ```
         Листинг
         ```
       - .gitlab-ci.yml
         ```
         Листинг
         ```
    2. consumer-service
       - Dockerfile
         ```
         Листинг
         ```
       - .gitlab-ci.yml
         ```
         Листинг
         ```
  - GitLab Runner
    1. Информация

## Ansible
  - Inventory-файл
  - Роли
  - Playbooks
    1. tt.yml
       ```
       Листинг
       ```
    2. bb.yml
       ```
       Листинг
       ```
    3. rr.yml
       ```
       Листинг
       ```

## firewalld
  - Информация
