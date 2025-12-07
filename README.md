# Практическое задание №10: JWT-аутентификация и авторизация
## Группа: ЭФМО-02-25
## ФИО: Евдоков Богдан Вадимович
## 🎯 Цель работы
Реализовать систему аутентификации и авторизации на основе JWT (JSON Web Tokens) в Go-приложении. Создать middleware для проверки токенов и контроля доступа по ролям (RBAC).

## 📋 Предварительные требования
* Go (версия 1.25 или выше)

* Git for Windows

* PowerShell (встроен в Windows 10/11)

## 📁 Структура проекта

<img width="417" height="516" alt="image" src="https://github.com/user-attachments/assets/17958402-55ea-4ebd-b6ae-e2edda953cd4" />



<br>

<img width="356" height="119" alt="image" src="https://github.com/user-attachments/assets/a9e5de15-35ba-4d85-b217-34ad436173f3" />



## 🚀 Инструкция по запуску
**1. Клонирование и настройка окружения**
```powershell
# Создайте проект (если еще не создан)
mkdir pz10-auth
cd pz10-auth

# Инициализация Go модуля
go mod init example.com/pz10-auth

# Установите зависимости
go get github.com/go-chi/chi/v5
go get github.com/golang-jwt/jwt/v5
go get golang.org/x/crypto/bcrypt
```
**2. Установите переменные окружения**
```powershell
# В PowerShell установите переменные окружения
$env:JWT_SECRET="my-super-secret-key-change-in-production"
$env:JWT_TTL="24h"
$env:APP_PORT="8080"
```
**3. Создайте структуру проекта**

*Создайте все файлы из практического задания в соответствующих директориях.*

**4. Запуск сервера**
```powershell
go run ./cmd/server/main.go
```
*Сервер запустится на порту 8080. В консоли появится сообщение:*

```text
listening on :8080
```
## 🔐 Тестирование работы API
**1. Получение JWT токена (Логин)**

Для администратора:
```powershell
# Используйте Invoke-RestMethod в PowerShell
$bodyAdmin = @{
    Email = "admin@example.com"
    Password = "secret123"
} | ConvertTo-Json

$responseAdmin = Invoke-RestMethod -Uri "http://localhost:8080/api/v1/login" `
    -Method Post `
    -Body $bodyAdmin `
    -ContentType "application/json"

Write-Host "Токен администратора: $($responseAdmin.token)"
```

Для обычного пользователя:
```powershell
$bodyUser = @{
    Email = "user@example.com"
    Password = "secret123"
} | ConvertTo-Json

$responseUser = Invoke-RestMethod -Uri "http://localhost:8080/api/v1/login" `
    -Method Post `
    -Body $bodyUser `
    -ContentType "application/json"

Write-Host "Токен пользователя: $($responseUser.token)"
```
**2. Доступ к защищенным эндпоинтам**

Получение профиля пользователя (доступно всем аутентифицированным пользователям):
```powershell
$TOKEN_ADMIN = $responseAdmin.token

$headers = @{
    "Authorization" = "Bearer $TOKEN_ADMIN"
}

# Запрос информации о текущем пользователе
Invoke-RestMethod -Uri "http://localhost:8080/api/v1/me" `
    -Method Get `
    -Headers $headers
```
Доступ к админской статистике (только для администраторов):
```powershell
# С токеном администратора - успешно
Invoke-RestMethod -Uri "http://localhost:8080/api/v1/admin/stats" `
    -Method Get `
    -Headers $headers

# С токеном обычного пользователя - ошибка 403 Forbidden
$TOKEN_USER = $responseUser.token
$userHeaders = @{
    "Authorization" = "Bearer $TOKEN_USER"
}

try {
    Invoke-RestMethod -Uri "http://localhost:8080/api/v1/admin/stats" `
        -Method Get `
        -Headers $userHeaders `
        -ErrorAction Stop
} catch {
    Write-Host "Ошибка доступа (ожидаемо): $($_.Exception.Message)"
}
```
## 📊 Примеры ответов API
Успешный логин:
```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```
Профиль пользователя (/api/v1/me):
```json
{
  "id": 1,
  "email": "admin@example.com",
  "role": "admin"
}
```
Админская статистика (/api/v1/admin/stats):
```json
{
  "users": 2,
  "version": "1.0"
}
```
Ошибка авторизации (403):
```json
{
  "error": "forbidden"
}
```
Ошибка аутентификации (401):
```json
{
  "error": "unauthorized"
}
```
🔧 Технические особенности реализации
1. JWT Структура
```go
// Заголовок
{
  "alg": "HS256",
  "typ": "JWT"
}

// Payload (клеймы)
{
  "sub": 1,
  "email": "admin@example.com",
  "role": "admin",
  "iat": 1695801110,
  "exp": 1695804710,
  "iss": "pz10-auth",
  "aud": "pz10-clients"
}
```
2. Middleware Цепочка
```text
HTTP Request
    ↓
AuthN Middleware (проверка JWT)
    ↓
AuthZ Middleware (проверка ролей)
    ↓
Обработчик запроса
```

## 📸 Скриншоты/вставки вывода
1. Успешный /login (токен)
<img width="1919" height="1079" alt="Снимок экрана 2025-12-07 182736" src="https://github.com/user-attachments/assets/040c2ec4-fa4c-4ba4-ada6-8e714078c331" />

2. /me и /admin/stats для admin
<img width="1919" height="1078" alt="Снимок экрана 2025-12-07 182745" src="https://github.com/user-attachments/assets/0bbb30d9-daaf-4136-92f0-41d251d5429d" />

3. 403 для user на /admin/stats
<img width="1919" height="1079" alt="Снимок экрана 2025-12-07 182813" src="https://github.com/user-attachments/assets/d55e18f3-8cf4-40df-86e2-b402f9bd2466" />


4. Refresh-флоу (старый/новый access)
<img width="1918" height="1079" alt="Снимок экрана 2025-12-07 182820" src="https://github.com/user-attachments/assets/3fa1eadb-13d2-46e5-a5ee-5f4d161781dd" />
