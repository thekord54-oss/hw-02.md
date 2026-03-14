# Домашнее задание к занятию «Система мониторинга Zabbix» - Шумихин Кирилл
**Выполнил:** Шумихин Кирилл

---

## Задание 1. Установка Zabbix Server с веб-интерфейсом

### Используемое окружение
- ОС: Debian 11
- БД: PostgreSQL (из стандартного репозитория Debian 11)
- Web: Apache2 + PHP
- Zabbix: 7.0 LTS

### Команды установки
```bash
# Обновление индекса пакетов
sudo apt update

# Установка PostgreSQL
sudo apt install -y postgresql

# Подключение репозитория Zabbix 7.0 LTS
wget https://repo.zabbix.com/zabbix/7.0/debian/pool/main/z/zabbix-release/zabbix-release_latest_7.0+debian11_all.deb
sudo dpkg -i zabbix-release_latest_7.0+debian11_all.deb
sudo apt update

# Установка Zabbix Server, фронтенда и агента
sudo apt install -y zabbix-server-pgsql zabbix-frontend-php php8.2-pgsql zabbix-apache-conf zabbix-sql-scripts zabbix-agent

# Создание БД и пользователя
sudo -u postgres psql -c "create user zabbix with password 'StrongPass123!';"
sudo -u postgres psql -c "create database zabbix owner zabbix;"

# Импорт схемы
zcat /usr/share/zabbix-sql-scripts/postgresql/server.sql.gz | PGPASSWORD='StrongPass123!' psql -U zabbix -h localhost zabbix

# Настройка подключения к БД в конфиге Zabbix Server
sudo sed -i "s/^# DBPassword=.*/DBPassword=StrongPass123!/" /etc/zabbix/zabbix_server.conf

# Перезапуск и включение сервисов
sudo systemctl restart zabbix-server zabbix-agent apache2
sudo systemctl enable zabbix-server zabbix-agent apache2
```

### Скриншот авторизации в админке
![Авторизация в Zabbix](./img/task1-zabbix-login.png)

---

## Задание 2. Установка Zabbix Agent на два хоста

### Хосты
- `zbx-server` — 192.168.1.10
- `zbx-agent-1` — 192.168.1.11
- `zbx-agent-2` — 192.168.1.12

### Команды на хостах с агентами
```bash
# На каждом агенте
wget https://repo.zabbix.com/zabbix/7.0/debian/pool/main/z/zabbix-release/zabbix-release_latest_7.0+debian11_all.deb
sudo dpkg -i zabbix-release_latest_7.0+debian11_all.deb
sudo apt update
sudo apt install -y zabbix-agent

# Настройка доступа сервера и имени хоста
sudo sed -i "s/^Server=.*/Server=192.168.1.10/" /etc/zabbix/zabbix_agentd.conf
sudo sed -i "s/^ServerActive=.*/ServerActive=192.168.1.10/" /etc/zabbix/zabbix_agentd.conf
sudo sed -i "s/^Hostname=.*/Hostname=zbx-agent-1/" /etc/zabbix/zabbix_agentd.conf

# Для второго хоста указывается его имя:
# Hostname=zbx-agent-2

# Запуск и добавление в автозагрузку
sudo systemctl restart zabbix-agent
sudo systemctl enable zabbix-agent

# Проверка статуса
sudo systemctl status zabbix-agent --no-pager
```

### Добавление хостов в Zabbix Server
1. `Configuration` → `Hosts` → `Create host`.
2. Добавлен интерфейс Agent (`IP` хоста, `Port 10050`).
3. Подключён шаблон `Linux by Zabbix agent`.
4. Сохранено и проверено, что статус `Enabled`.

### Проверка поступления данных
```bash
# На сервере — проверка доступности агента
zabbix_get -s 192.168.1.11 -k agent.ping
zabbix_get -s 192.168.1.12 -k agent.ping

# Лог агента
sudo tail -n 50 /var/log/zabbix/zabbix_agentd.log
```

### Скриншоты
1. Configuration > Hosts (подключенные агенты):

![Hosts](./img/task2-hosts.png)

2. Лог zabbix-agent:

![Agent log](./img/task2-agent-log.png)

3. Monitoring > Latest data по двум хостам:

![Latest data](./img/task2-latest-data.png)

---

## Задание 3* (дополнительно)
Не выполнялось.

---

## Итог
- Установлен и настроен Zabbix Server с PostgreSQL и веб-интерфейсом.
- Подключены 2 хоста с установленным Zabbix Agent.
- Подтверждено получение метрик в `Latest data`.
