# [app-server]

## Общая информация
- Static hostname: app-server 
- Operating System: Rocky Linux 8.10 (Green Obsidian)
- Kernel: Linux 4.18.0-553.el8_10.x86_64   
- Architecture: x86-64
- IP: 192.168.140.137

## Сервисы
  1. docker.service
  - Информация
  2. node_exporter.service
  - Информация
    ```service
    [Unit]
    Description=Node Exporter Service
    After=network.target
    
    [Service]
    User=nodeuser
    Group=nodeuser
    Type=simple
    ExecStart=/usr/local/bin/node_exporter
    ExecReload=/bin/kill -HUP
    Restart=on-failure
    
    [Install]
    WantedBy=multi-user.target
    ```

## Контейнеры
  1. postgres:15.8
  - Информация
    
    | CONTAINER ID | IMAGE | COMMAND | PORTS | NAMES |
    | ------------ | ----- | ------- | ----- | ----- |
    | 25ebc09815e0 | postgres:15.8 | "docker-entrypoint.s…" | 0.0.0.0:5432->5432/tcp, :::5432->5432/tcp | postgresql |
    
  2. gitlab/gitlab-runner:latest
  - Информация

    | CONTAINER ID | IMAGE | COMMAND | PORTS | NAMES |
    | ------------ | ----- | ------- | ----- | ----- |
    | e88537f16f42 | gitlab/gitlab-runner:latest | "/usr/bin/dumb-init …" | - | gitlab-runner |
  
  - Конфигурация [/srv/gitlab-runner/config/config.toml]
    ```toml
    concurrent = 1
    check_interval = 0
    shutdown_timeout = 0
    
    [session_server]
      session_timeout = 1800
    
    [[runners]]
      name = "Runner for publisher and consumer applications"
      url = "http://192.168.140.136:8080"
      id = 34
      token = "-"
      executor = "docker"
      clone_url = "http://192.168.140.136:8080"
      [runners.custom_build_dir]
      [runners.cache]
        MaxUploadedArchiveSize = 0
        [runners.cache.s3]
        [runners.cache.gcs]
        [runners.cache.azure]
      [runners.docker]
        tls_verify = false
        image = "docker:latest"
        privileged = true
        disable_entrypoint_overwrite = false
        oom_kill_disable = false
        disable_cache = false
        volumes = ["/cache", "/var/run/docker.sock:/var/run/docker.sock"]
        shm_size = 0
    ```

  3. rabbitmq:3.13.7-management
  - Информация

    | CONTAINER ID | IMAGE | COMMAND | PORTS | NAMES |
    | ------------ | ----- | ------- | ----- | ----- |
    | 67ec4361cf5e | rabbitmq:3.13.7-management | "docker-entrypoint.s…" | 4369/tcp, 5671/tcp, 0.0.0.0:5672->5672/tcp, :::5672->5672/tcp, 15671/tcp, 15691-15692/tcp, 25672/tcp, 0.0.0.0:15672->15672/tcp, :::15672->15672/tcp | rabbitmq |

  4. google/cadvisor:latest
  - Информация

    | CONTAINER ID | IMAGE | COMMAND | PORTS | NAMES |
    | ------------ | ----- | ------- | ----- | ----- |
    | 691de35e3182 | google/cadvisor:latest | "/usr/bin/cadvisor -…" | 0.0.0.0:9090->8080/tcp, [::]:9090->8080/tcp | cadvisor |

## docker-compose.yml
  - Описание
    ```yaml
    services:
    consumer_service:
      image: '192.168.140.136:5000/consumer_service:latest'
      container_name: consumer_service
      depends_on:
        - rabbitmq
      restart: always
      environment:
        - RABBITMQ_HOST=rabbitmq
  
    publisher_service:
      image: '192.168.140.136:5000/publisher_service:latest'
      container_name: publisher_service
      depends_on:
        - rabbitmq
        - postgresql
      restart: always
      ports:
        - '7070:8080'
      environment:
        - POSTGRES_CONNECTION=Host=postgresql;Port=5432;Database=usersdb;Username=api;Password=api;Pooling=true;
        - RABBITMQ_HOST=rabbitmq
  
    rabbitmq:
      image: 'rabbitmq:3.13.7-management'
      container_name: rabbitmq
      restart: always
      ports:
        - '5672:5672'
        - '15672:15672'
      volumes:
        - /rabbitmq/data/:/var/lib/rabbitmq
        - /rabbitmq/log/:/var/log/rabbitmq
  
    postgresql:
      image: 'postgres:15.8'
      container_name: postgresql
      restart: always
      ports:
        - '5432:5432'
      volumes:
        - /postgres/data/:/var/lib/postgresql/data
        - /postgres/log/:/var/log/postgresql
      environment:
        POSTGRES_USER: api
        POSTGRES_PASSWORD: api
        POSTGRES_DB: usersdb
  
    gitlab-runner:
      image: 'gitlab/gitlab-runner:latest'
      container_name: gitlab-runner
      restart: always
      volumes:
        - /srv/gitlab-runner/config:/etc/gitlab-runner
        - /var/run/docker.sock:/var/run/docker.sock
      environment:
        - CI_SERVER_URL=http://192.168.140.136:8080
        - REGISTRATION_TOKEN=-
        - RUNNER_EXECUTOR=docker
  
    cadvisor:
      image: google/cadvisor:latest
      container_name: cadvisor
      privileged: true
      restart: always
      ports:
        - '9090:8080'
      volumes:
        - /:/rootfs:ro
        - /var/run:/var/run:rw
        - /sys:/sys:ro
        - /var/lib/docker/:/var/lib/docker:ro
        - /dev/disk/:/dev/disk:ro
    ```

### Скрипт с POST-запросом к publisher_service
```csharp
using System.Net.Http.Json;

Console.WriteLine("\nTarget URL: ");
string? targetUrl = Console.ReadLine();

using var client = new HttpClient();

int requestsCount = 8;
var chars = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789";
var nameChars = new char[8];
var passwordChars = new char[8];
var random = new Random();

for (int i = 0; i < requestsCount; i++)
{
    for (int j = 0; j < nameChars.Length; j++)
    {
        nameChars[j] = chars[random.Next(chars.Length)];
        passwordChars[j] = chars[random.Next(chars.Length)];
    }

    var userCredentials = new
    {
        name = new string(nameChars),
        password = new string(passwordChars)
    };

    var jsonContent = JsonContent.Create(userCredentials);

    await client.PostAsync(targetUrl, jsonContent);
}
```
