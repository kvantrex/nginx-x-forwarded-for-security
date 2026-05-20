# Nginx X-Forwarded-For Chain Test Stand

Тестовый стенд, демонстрирующий настройку Nginx в режиме обратного прокси с поддержкой произвольных цепочек маршрутизации и надёжной защитой от спуфинга заголовка "X-Forwarded-For".

## Описание
Решение задачи по организации безопасного проброса IP-адресов через каскад reverse-proxy серверов
- Приложение получает корректную цепочку: Клиент → Nginx1 → Nginx2 → ... → NginxN
- Пользовательский "X-Forwarded-For" **полностью игнорируется** на границе доверенной сети
- Конфигурация **позиционно-независима**: один файл nginx/default.conf подходит для любого количества узлов в цепочке

## Deploy
1. Install docker & docker-compose 
2. cd /opt && git clone https://github.com/kvantrex/nginx-x-forwarded-for-security.git
3. cd nginx-x-forwarded-for-security
4. docker compose up -d

## Проверка цепочки взаимодействия nginx

Обращение к nginx1\
curl -H "X-Next-Hop: nginx1" http://localhost:8081 \
Обращение к nginx2 через nginx1\
curl -H "X-Next-Hop: nginx2" http://localhost:8081 \
Обращение к nginx3 через nginx1\
curl -H "X-Next-Hop: nginx3" http://localhost:8081

Обращение к nginx2\
curl -H "X-Next-Hop: nginx2" http://localhost:8082 \
Обращение к nginx1 через nginx2\
curl -H "X-Next-Hop: nginx2" http://localhost:8082 \
Обращение к nginx3 через nginx2\
curl -H "X-Next-Hop: nginx3" http://localhost:8082

Обращение к nginx3\
curl -H "X-Next-Hop: nginx3" http://localhost:8083 \
Обращение к nginx1 через nginx3\
curl -H "X-Next-Hop: nginx2" http://localhost:8083 \
Обращение к nginx2 через nginx3\
curl -H "X-Next-Hop: nginx3" http://localhost:8083

* Вместо localhost можно использовать IP адрес сервера и тестировать с удаленного хоста\

## Проверка работы заголовка X-Forwarded-For

Отправка своего заголовка в nginx1\
curl -H "X-Forwarded-For: 8.8.8.8" http://localhost:8081 

Отправка своего заголовка в nginx2\
curl -H "X-Forwarded-For: 8.8.8.8" http://localhost:8082 

Отправка своего заголовка в nginx3\
curl -H "X-Forwarded-For: 8.8.8.8" http://localhost:8083

* Вместо localhost можно использовать IP адрес сервера и тестировать с удаленного хоста\

