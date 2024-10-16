# [infra-main]

## Разделы
1. Общая информация
2. Сервисы
3. Контейнеры
4. docker-compose.yml
5. GitLab
6. Ansible
7. firewalld

## Общая информация
 - Имя хоста: infra-main 
 - Операционная система: Rocky Linux 8.10 (Green Obsidian)
 - Ядро: Linux 4.18.0-553.el8_10.x86_64   
 - Архитектура: x86-64
 - IP-адрес: 192.168.140.136

## Сервисы
  1. docker.service
  2. node_exporter.service
  - Информация [/etc/systemd/system/node_exporter.service]
    ```service
    [Unit]
    Description=Node Exporter Service
    After=network.target
    
    [Service]
    User=nodeuser
    Group=nodeuser
    Type=simple
    ExecStart=/usr/local/bin/node_exporter
    ExecReload=/bin/kill -HUP $MAINPID
    Restart=on-failure
    
    [Install]
    WantedBy=multi-user.target
    ```
  - Установка [/root/node_exporter/install-script.sh]
    ```bash
    #!/bin/bash

    set -e
    
    touch node_exporter.service
    cat > node_exporter.service << EOF
    [Unit]
    Description=Node Exporter Service
    After=network.target
    
    [Service]
    User=nodeuser
    Group=nodeuser
    Type=simple
    ExecStart=/usr/local/bin/node_exporter
    ExecReload=/bin/kill -HUP $MAINPID
    Restart=on-failure
    
    [Install]
    WantedBy=multi-user.target
    EOF
    
    cp node_exporter.service /etc/systemd/system/node_exporter.service
    
    wget https://github.com/prometheus/node_exporter/releases/download/v1.8.2/node_exporter-1.8.2.linux-amd64.tar.gz
    
    packageName="node_exporter"
    archiveName="$(ls -1 | grep -E "${packageName}" | grep -E "tar.gz" | grep -v "service")"
    installDir="/usr/local/bin/"
    
    extractArchieve() {
            if [[ -f "${archiveName}" ]]; then
                    tar -xf $(ls -1 | grep -E "${packageName}" | grep -E "tar.gz" | grep -v "service")
            else
                    exit 1
            fi
    }
    
    echo -e "\n1. Unzipping tar.gz..."
    extractArchieve
    echo -e "1. +"
    
    cd $(ls -1 | grep -E "${packageName}" | grep -v "tar.gz" | grep -v "service")
    
    echo -e "\n3. Copying ${packageName} to ${installDir}"
    cp "${packageName}" "${installDir}"
    echo -e "3. +"
    
    echo -e "\n4. Creating nodeuser with no home directory and no login"
    echo -e "\n4. Giving permissions to nodeuser"
    useradd --no-create-home --shell /bin/false nodeuser
    chown -R nodeuser:nodeuser /usr/local/bin/"${packageName}"
    echo -e "4. +"
    
    systemctl --now enable node_exporter.service
    systemctl status node_exporter.service
    ```

## Контейнеры
  1. GitLab CE
  - Информация
    
    | CONTAINER ID | IMAGE | COMMAND | PORTS | NAMES |
    | ------------ | ---- | -------- | ----- | ----- |
    | 1ba7a8c36667 | gitlab/gitlab-ce:17.1.7-ce.0 | "/assets/wrapper" | 0.0.0.0:22->22/tcp, :::22->22/tcp, 0.0.0.0:443->443/tcp, :::443->443/tcp, 0.0.0.0:8080->80/tcp, [::]:8080->80/tcp | gitlab |
    
  - Конфигурация [/srv/gitlab/config/gitlab.rb] — пропущены закомментированные строки
    ```pp
    external_url 'http://192.168.140.139:8888'
    nginx['listen_port'] = 80
    nginx['listen_https'] = false
    nginx['proxy_set_headers'] = {
      "Host" => "192.168.140.139:8888",
      "X-Real-IP" => "$remote_addr",
      "X-Forwarded-For" => "$proxy_add_x_forwarded_for",
      "X-Forwarded-Proto" => "http",
      "X-Forwarded-Ssl" => "off",
    }
    ```
  2. Docker Registry
  - Информация
    
    | CONTAINER ID | IMAGE | COMMAND | PORTS | NAMES |
    | ------------ | ----- |-------- |------ |------ |
    | d65158029db8 | registry:2.7.0 | "/entrypoint.sh /etc…" | 0.0.0.0:5000->5000/tcp, :::5000->5000/tcp | docker_registry |
    
  3. Cadvisor
  - Информация

    | CONTAINER ID | IMAGE | COMMAND | PORTS | NAMES |
    | ------------ | ----- |-------- |------ |------ |
    | 45f7c9d237fa | google/cadvisor:latest | "/usr/bin/cadvisor -…" | 0.0.0.0:5050->8080/tcp, [::]:5050->8080/tcp | cadvisor |

## docker-compose.yml
  - Описание
    ```yaml
    services:
    gitlab:
      image: gitlab/gitlab-ce:17.1.7-ce.0
      container_name: gitlab
      restart: always
      hostname: '192.168.140.136'
      environment:
        GITLAB_OMNIBUS_CONFIG: |
          external_url 'http://192.168.140.136'
      ports:
        - '8080:80'
        - '443:443'
        - '22:22'
      volumes:
        - '/srv/gitlab/config:/etc/gitlab'
        - '/srv/gitlab/logs:/var/log/gitlab'
        - '/srv/gitlab/data:/var/opt/gitlab'
      shm_size: '256m'
  
    cadvisor:
      image: google/cadvisor:latest
      container_name: cadvisor
      privileged: true
      restart: always
      ports:
        - '5050:8080'
      volumes:
        - /:/rootfs:ro
        - /var/run:/var/run:rw
        - /sys:/sys:ro
        - /var/lib/docker/:/var/lib/docker:ro
        - /dev/disk/:/dev/disk:ro
  
    docker-registry:
      image: registry:2.7.0
      container_name: docker_registry
      restart: always
      ports:
        - '5000:5000'
      volumes:
        - '/srv/gitlab/registry/data:/var/lib/registry'
        - '/srv/gitlab/registry/auth:/auth'
      environment:
        REGISTRY_AUTH: htpasswd
        REGISTRY_AUTH_HTPASSWD_PATH: /auth/htpasswd
        REGISTRY_AUTH_HTPASSWD_REALM: "Registry Realm"
    ```

## GitLab
  - Репозитории (Dockerfile, .gitlab-ci.yml, исходный код)
    1. [registration-publisher-service](https://github.com/perc400/registration-publisher-service-web)
    2. [consumer-service](https://github.com/perc400/consumer-service-console)
  - GitLab Runner
    1. Информация

## Ansible [/root/ansible]
  - [Inventory-файл](https://github.com/perc400/ansible/blob/main/hosts)
  - [Переменные групп](https://github.com/perc400/ansible/blob/main/group_vars/all_VMs)
  - [Роли](https://github.com/perc400/ansible/tree/main/roles)
  - Playbooks:
      - [Развертывание сервисов RedHat/Debian](https://github.com/perc400/ansible/blob/main/install_services_pb.yml)
      - [Установка Grafana Loki](https://github.com/perc400/ansible/blob/main/install_loki_pb.yml)
      - [Установка tcpdump](https://github.com/perc400/ansible/blob/main/install_tcpdump_pb.yml)
      - [Проверка доступности ВМ](https://github.com/perc400/ansible/blob/main/ping_all_pb.yml)

## firewalld
  - Информация
