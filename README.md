# Terra-VPN-TV

## 1. Проверка доступа к приложению
### Описание процесса
Приложение отправляет запрос на сервер для проверки, есть ли доступ у пользователя.
**Запрос содержит два обязательных поля:**
- token — уникальный токен авторизации пользователя.
- device_id — уникальный идентификатор устройства.
### Запрос
- Метод: POST
- URL: /api/check_access
  
**Тело:**
```json
{
  "token": "user_token_here",
  "device_id": "device_id_here"
}
```
### Ответы сервера
#### Если доступ запрещён:
**Сервер возвращает:**
- Поле success: false, указывающее, что доступ запрещён.
- Поле telegram_link — ссылка на Telegram-канал для получения информации.
```json
{
  "success": false,
  "telegram_link": "https://t.me/joinchat/example_channel"
}
```
**Действие приложения:**
- Сгенерировать QR-код на основе telegram_link и отобразить поверх экрана.
#### Если доступ разрешён:
Сервер возвращает только поле success: true:
```json
{
  "success": true
}
```
**Действие приложения:**
- Отобразить кнопку подключения к VPN.
## 2. Включение VPN
### Описание процесса
- Пользователь нажимает кнопку "Включить VPN".
- Приложение отправляет запрос на сервер с:
  - Уникальным device_id в теле запроса.
  - Токеном авторизации (Bearer token) в заголовке.
### Запрос
- Метод: POST
- URL: /api/vpn/start
### Заголовки:
```http
Authorization: Bearer user_token_here
```
Тело:
```json
{
  "device_id": "device_id_here"
}
```
### Ответы сервера
#### Если включение прошло успешно:
Сервер возвращает настройки VPN:
```json
{
  "success": true,
  "vpn_settings": {
    "server_address": "vpn.example.com",
    "username": "vpn_user",
    "password": "vpn_password",
    "encryption": "AES-256",
    "protocol": "OpenVPN"
  }
}
```
**Действие приложения:**
- Настроить VPN-соединение на телевизоре, используя полученные параметры.
- Если произошла ошибка:
  - Сервер возвращает success: false и сообщение об ошибке:
```json
{
  "success": false,
  "error_message": "Unable to start VPN"
}
```
**Действие приложения:**
- Отобразить сообщение об ошибке.
## 3. Выключение VPN
### Описание процесса
- Пользователь нажимает кнопку "Выключить VPN".
- Приложение отправляет запрос с:
  - Уникальным device_id в теле запроса.
  - Токеном авторизации (Bearer token) в заголовке.
### Запрос
- Метод: POST
- URL: /api/vpn/stop
### Заголовки:
```http
Authorization: Bearer user_token_here
```
Тело:
```json
{
  "device_id": "device_id_here"
}
```
### Ответы сервера
#### Если отключение прошло успешно:
Сервер возвращает:
```json
{
  "success": true
}
```
#### Если произошла ошибка:
Сервер возвращает:
```json
{
  "success": false,
  "error_message": "Failed to disconnect VPN"
}
```
### Действие приложения:
Показать сообщение об ошибке или успехе в зависимости от ответа сервера.
## 4. Запрос трафика и информации о подписке
### Описание процесса
- Приложение периодически (например, каждые 60 секунд) отправляет запрос для получения информации о:
  - Использованном трафике (сколько данных получено и отправлено).
  - Остатке дней подписки.
### Запрос
- Метод: POST
- URL: /api/vpn/traffic
### Заголовки:
```http
Authorization: Bearer user_token_here
```
Тело:
```json
{
  "device_id": "device_id_here"
}
```
### Ответ сервера
Сервер возвращает данные о трафике и подписке:
```json
{
  "success": true,
  "data_usage": {
    "received_traffic_mb": 1024,
    "sent_traffic_mb": 512
  },
  "subscription": {
    "days_remaining": 15
  }
}
```
### Действие приложения:
- Обновить отображение информации о затраченном трафике (полученные и отправленные данные) и оставшихся днях подписки.
- Если произошла ошибка:
#### Сервер возвращает:
```json
{
  "success": false,
  "error_message": "Unable to fetch traffic data"
}
```
#### Действие приложения:
- Показать сообщение об ошибке или попробовать повторить запрос позже.
## Сводная таблица эндпоинтов
```markdown
| Этап                | URL                 | Метод | Поля запроса                 | Поля ответа                               |
|---------------------|---------------------|-------|------------------------------|-------------------------------------------|
| Проверка доступа    | `/api/check_access` | POST  | `token`, `device_id`         | `success`, `telegram_link` (если `false`) |
| Включение VPN       | `/api/vpn/start`    | POST  | `device_id` + `Bearer token` | `success`, `vpn_settings`                 |
| Выключение VPN      | `/api/vpn/stop`     | POST  | `device_id` + `Bearer token` | `success`, `error_message` (если `false`) |
| Запрос трафика      | `/api/vpn/traffic`  | POST  | `device_id` + `Bearer token` | `success`, `data_usage`, `subscription`   |
```
