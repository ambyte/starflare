# Starflare Docker Setup

Этот документ описывает, как запустить Starflare используя Docker и Docker Compose.

## Быстрый старт

```bash
# Клонируйте репозиторий
git clone <repository-url>
cd starflare

# Скопируйте и настройте переменные окружения
cp .env.example .env
# Отредактируйте .env файл с вашими GitHub OAuth credentials

# Запустите приложение
docker-compose up -d

# Приложение будет доступно по адресу http://localhost:3000
```

## Архитектура

### Сервисы

1. **frontend** - Vue.js приложение на Nginx

   - Порт: 3000
   - Собирается из исходного кода
   - Настроен для работы с клиентским роутингом

2. **backend** - Mock сервер для Cloudflare Functions

   - Порт: 7001
   - Эмулирует API endpoints из папки functions/
   - Обрабатывает GitHub OAuth token exchange

3. **redis** - Кеширование (опционально)
   - Порт: 6379
   - Используется для кеширования данных

### Файлы конфигурации

- `Dockerfile` - Продакшн образ с многоэтапной сборкой
- `docker-compose.yml` - Конфигурация приложения
- `.dockerignore` - Исключения для Docker build

## Настройка GitHub OAuth

1. Создайте GitHub OAuth App:

   - Перейдите в Settings -> Developer settings -> OAuth Apps
   - Нажмите "New OAuth App"
   - Заполните поля:
     - Application name: "Starflare"
     - Homepage URL: "http://localhost:3000"
     - Authorization callback URL: "http://localhost:3000"

2. Скопируйте Client ID и Client Secret в .env файл:
   ```
   GITHUB_CLIENT_ID=your_client_id_here
   GITHUB_CLIENT_SECRET=your_client_secret_here
   ```

## Команды управления

```bash
# Запуск приложения
docker-compose up -d

# Просмотр логов
docker-compose logs -f

# Остановка сервисов
docker-compose down

# Остановка с удалением volumes
docker-compose down -v

# Пересборка образов
docker-compose build --no-cache

# Запуск только определенного сервиса
docker-compose up frontend

# Масштабирование сервиса
docker-compose up --scale frontend=2
```

## Мониторинг

### Health Checks

Все сервисы настроены с health checks:

- Frontend: проверка HTTP ответа на порту 80
- Backend: проверка HTTP ответа на порту 7001
- Redis: проверка ping команды

### Просмотр статуса

```bash
# Статус всех сервисов
docker-compose ps

# Детальная информация о health checks
docker inspect starflare-frontend | grep -A 10 "Health"
```

## Развертывание

### Локальное развертывание

```bash
docker-compose up -d
```

### Развертывание на сервере

1. Скопируйте файлы на сервер
2. Настройте переменные окружения
3. Настройте обратный прокси (Nginx/Traefik)
4. Запустите с restart policy:

```bash
docker-compose up -d --restart unless-stopped
```

### CI/CD интеграция

Пример для GitHub Actions:

```yaml
- name: Deploy with Docker Compose
  run: |
    docker-compose pull
    docker-compose up -d
```

## Troubleshooting

### Общие проблемы

1. **Порты заняты**

   ```bash
   # Проверьте занятые порты
   netstat -tulpn | grep :3000

   # Измените порты в docker-compose.yml
   ports:
     - "3001:80"  # Вместо 3000:80
   ```

2. **Проблемы с разрешениями**

   ```bash
   # Исправьте права на файлы
   sudo chown -R $USER:$USER .
   ```

3. **Проблемы с сетью**
   ```bash
   # Пересоздайте сеть Docker
   docker-compose down
   docker network prune
   docker-compose up -d
   ```

### Логи и отладка

```bash
# Логи всех сервисов
docker-compose logs

# Логи конкретного сервиса
docker-compose logs frontend

# Следить за логами в реальном времени
docker-compose logs -f backend

# Войти в контейнер для отладки
docker-compose exec frontend sh
```

### Очистка

```bash
# Удалить все остановленные контейнеры
docker container prune

# Удалить неиспользуемые образы
docker image prune

# Полная очистка системы Docker
docker system prune -a
```

## Дополнительные возможности

### Мониторинг с Prometheus и Grafana

Добавьте в docker-compose.yml:

```yaml
prometheus:
  image: prom/prometheus
  ports:
    - '9090:9090'
  volumes:
    - ./monitoring/prometheus.yml:/etc/prometheus/prometheus.yml

grafana:
  image: grafana/grafana
  ports:
    - '3001:3000'
  environment:
    - GF_SECURITY_ADMIN_PASSWORD=admin
```

### Backup базы данных

```bash
# Создать backup Redis
docker-compose exec redis redis-cli --rdb /data/backup.rdb

# Восстановить backup
docker-compose down
docker-compose up -d
```

## Ссылки

- [Docker Documentation](https://docs.docker.com/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [Vue.js Docker Guide](https://vuejs.org/guide/extras/deployment.html#docker)
- [Nginx Docker Hub](https://hub.docker.com/_/nginx)
