# Documentation [Intern-Project]
(В процессе написания)
## Описание проекта:
Этот проект создан для демонстрации навыков настройки инфраструктуры. В рамках проекта настроено несколько VM, автоматизированы процессы развертывания приложений и мониторинга, а также внедрены системы контейнеризации и логирования. Проект охватывает аспекты администрирования серверов, работы с Docker, CI/CD, мониторинга и логирования.

## Навигация по документации:
### Разделы:
- [infra-main](infra-main/index.md)
- [app-server](app-server/index.md)
- [proxy-server](proxy-server/index.md)
- [monitoring-server](monitoring-server/index.md)

## Основные задачи:
1. Установка и конфигурация инфраструктуры:
   - Виртуальные машины с ОС Rocky Linux и Ubuntu для работы с сервисами (GitLab, PostgreSQL, RabbitMQ и др.).

2. SSH:
   - Настройка безопасного доступа по SSH с использованием ключей, изменены стандартные порты.

3. CI/CD с GitLab:
   - Развертывание GitLab в контейнере и настройка пайплайнов для автоматической сборки и деплоя приложений с использованием GitLab CI.

4. Контейнеризация приложений:
   - Контейнеризация приложений, создание Dockerfile для их развертывания в изолированной среде.

5. Подключение приватного Docker Registry с аутентификацией:
   - Конфигурация приватного Docker Registry для хранения образов.

6. Nginx как reverse-proxy:
   - Конфигурация Nginx для проксирования трафика к внутренним сервисам, таким как GitLab и приложения.

7. Мониторинг и логирование:
   - Конфигурация Prometheus и Grafana для мониторинга серверов и сервисов, Loki для сбора логов. Конфигурация алертинга через Telegram.

### Использованные технологии
- ОС: Rocky Linux 8, Ubuntu Server 22.04;
- Контейнеризация: Docker, Docker-compose;
- CI/CD: GitLab;
- Мониторинг: Prometheus, Grafana, Loki;
- Системы управления конфигурациями: Ansible;
- Прочие сервисы: PostgreSQL, RabbitMQ, Nginx.

### Репозитории
- https://github.com/perc400/registration-publisher-service-web
- https://github.com/perc400/consumer-service-console
- https://github.com/perc400/ansible
