# Настройка переменных окружения для Starflare

## 1. Создайте файл .env в корне проекта:

```bash
# GitHub OAuth Configuration
VUE_APP_CLIENT_ID=ваш_github_client_id
GITHUB_CLIENT_ID=ваш_github_client_id
GITHUB_CLIENT_SECRET=ваш_github_client_secret

# API Configuration
VUE_APP_API_BASE_URL=https://gitstars.sergeygulyaev.ru/api

# Redis Configuration
REDIS_PASSWORD=ваш_redis_пароль
```

## 2. Получите GitHub OAuth credentials:

1. Перейдите в GitHub Settings → Developer settings → OAuth Apps
2. Нажмите "New OAuth App"
3. Заполните:
   - Application name: `Starflare`
   - Homepage URL: `https://gitstars.sergeygulyaev.ru`
   - Authorization callback URL: `https://gitstars.sergeygulyaev.ru/#/login`
4. Скопируйте Client ID и Client Secret в .env файл

## 3. Пересоберите Docker образ:

```bash
# Остановите текущие контейнеры
docker-compose down

# Пересоберите с новыми переменными
docker-compose build --no-cache

# Запустите
docker-compose up -d
```

## 4. Проверьте что переменные передались:

```bash
# Проверьте логи сборки
docker-compose logs frontend

# Войдите в контейнер и проверьте (для отладки)
docker-compose exec frontend sh
# В контейнере найдите файлы в /usr/share/nginx/html/js/
# Найдите строку с client_id в main.js файлах
```

## Troubleshooting:

Если client_id всё ещё не передается:

1. Убедитесь что .env файл находится в той же папке что и docker-compose.yml
2. Переменные должны начинаться с VUE*APP* для Vue.js
3. Пересоберите образ полностью: `docker-compose build --no-cache`
