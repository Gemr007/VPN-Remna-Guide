# 🛡️ Remnawave VPN — Полное руководство по установке

[![Remnawave](https://img.shields.io/badge/Panel-Remnawave-6366f1?style=for-the-badge)](https://docs.rw)
[![Xray](https://img.shields.io/badge/Core-Xray-blue?style=for-the-badge)](https://github.com/XTLS/Xray-core)
[![eGamesAPI](https://img.shields.io/badge/Script-eGamesAPI-green?style=for-the-badge)](https://github.com/eGamesAPI/remnawave-reverse-proxy)
[![License](https://img.shields.io/badge/License-MIT-yellow?style=for-the-badge)](LICENSE)

> Гайд по развёртыванию собственного VPN на базе панели **Remnawave** + **Xray** с помощью установочного скрипта от **eGamesAPI**.

---

## 📋 Содержание

1. [Подготовка](#1-подготовка)
   - [Выбор сервера](#11-выбор-сервера)
   - [Архитектура: панель и нода](#12-архитектура-панель-и-нода)
2. [Домен](#2-домен)
   - [Бесплатный домен (FreeDNS)](#21-бесплатный-домен-freedns)
   - [Платный домен + Cloudflare](#22-платный-домен--cloudflare)
3. [Установка панели](#3-установка-панели)
   - [Подключение к серверу](#31-подключение-к-серверу)
   - [Панель и нода на одном сервере](#32-панель-и-нода-на-одном-сервере)
   - [Панель и нода на разных серверах](#33-панель-и-нода-на-разных-серверах)
4. [Дополнительные транспорты](#4-дополнительные-транспорты)
   - [XHTTP](#41-xhttp)
   - [Hysteria2](#42-hysteria2)
5. [Серверный роутинг](#5-серверный-роутинг)
6. [Защита серверов](#6-защита-серверов)

---

## 1. Подготовка

### 1.1 Выбор сервера

Для развёртывания VPN понадобится VPS/VDS **за рубежом** (Нидерланды, Финляндия, Германия — хороший выбор).  
Ниже несколько хостингов, которые принимают российские карты и чьи подсети не заблокированы РКН:

| Хостинг | Локации | Комментарий |
|---|---|---|
| [Node Host](https://t.me/nodehost_bot?start=4677) | DE, SE, FI, PL, NL | Telegram-бот; есть 1G и 10G каналы. На DE — YouTube с рекламой |
| [Hip.Hosting](https://hip.hosting/?code=8038f2c25ad5281fabbb) | Много локаций | Стабильный, широкий выбор |
| [EXPRESSHOST](https://t.me/ExpressHost_Bot?start=ref_vJJK1zUM) | Базовые EU | Telegram-бот; бывают редкие отвалы |
| [LolzTeam каталог](https://lolz.live/forums/763/) | Разные | Большой список — читайте отзывы перед покупкой |

---

### 1.2 Архитектура: панель и нода

Инфраструктура Remnawave состоит из двух компонентов:

- **Панель** — центр управления нодами, конфигами и подключениями.
- **Нода** — сервер с Xray-ядром, через который идёт трафик пользователей.

Скрипт eGamesAPI умеет устанавливать **оба компонента на один сервер** — это удобно для личного использования (10–15 человек).

#### Минимальные характеристики серверов

| Конфигурация | CPU | RAM | Диск |
|---|---|---|---|
| Панель + нода (1 сервер) | 1–2 ядра | 2 ГБ | 15–20 ГБ |
| Только панель | 2 ядра | 2 ГБ | 20 ГБ |
| Только нода (до ~15 чел.) | 1 ядро | 1 ГБ | 10 ГБ |

> [!TIP]
> Если планируете больше 15 пользователей или хотите удобнее настраивать дополнительные транспорты — берите два сервера сразу.

---

## 2. Домен

### 2.1 Бесплатный домен (FreeDNS)

1. Зарегистрируйтесь на [FreeDNS](https://freedns.afraid.org/).
2. Перейдите во вкладку **Registry** / **Domain Registry**.
3. Выберите любой домен со статусом **Public** (бесплатный).
4. В меню **Add a new subdomain** создайте DNS-записи:

![Пример добавления субдомена в FreeDNS](Freedns.png)

#### Панель и нода на **одном** сервере

| Тип | Имя | Значение | Прокси |
|---|---|---|---|
| A | `example.com` | `your_server_ip` | DNS only |
| CNAME | `panel.example.com` | `example.com` | DNS only |
| CNAME | `sub.example.com` | `example.com` | DNS only |
| CNAME | `node.example.com` | `example.com` | DNS only |

#### Панель и нода на **разных** серверах

| Тип | Имя | Значение | Прокси |
|---|---|---|---|
| A | `example.com` | `panel_server_ip` | DNS only |
| CNAME | `panel.example.com` | `example.com` | DNS only |
| CNAME | `sub.example.com` | `example.com` | DNS only |
| A | `node.example.com` | `node_server_ip` | DNS only |

> [!NOTE]
> После добавления записей подождите 10–15 минут. Проверка:
> ```bash
> ping subdomain.example.com
> ```
> Если в выводе виден IP вашего сервера — записи применились.

---

### 2.2 Платный домен + Cloudflare

Платный домен даёт своё имя и возможность использовать Cloudflare, где DNS-записи применяются мгновенно.

Купить домен можно на: [SpaceWeb](https://sweb.ru/), [LuxHOST](https://luxhost.cc/), [TimeWeb](https://timeweb.cloud/) и др.

Таблицы DNS-записей — те же, что в разделе выше.

Инструкция по переносу домена на Cloudflare: [bisquit.host/cloudflare/transfer-domain](https://wiki.bisquit.host/cloudflare/transfer-domain).

---

## 3. Установка панели

### 3.1 Подключение к серверу

Есть два способа:

**Способ 1 — через терминал ОС** (PowerShell, CMD, Terminal):

```bash
ssh root@IP_SERVER
```

При первом подключении введите `yes`, затем вставьте пароль (он не отображается — это нормально).

![Подключение через SSH-терминал](screenshots/ssh_terminal.png)

**Способ 2 — SSH-клиент (рекомендуется)**

Один раз вводите IP / логин / пароль — и подключаетесь в один клик.  
Популярные варианты: **Termius** (красивый UI, бесплатного функционала хватает), MobaXterm, Putty, SmarTTY.

Скачать Termius: [termius.com](https://termius.com/)

Нажмите **New Host**, заполните IP, Username, Password и нажмите **Connect**.

![Настройка Termius](screenshots/termius.png)

---

### 3.2 Панель и нода на одном сервере

```bash
# 1. Если вы не root — переключитесь
sudo -i

# 2. Обновите пакеты
apt update && apt upgrade -y

# 3. Запустите скрипт eGamesAPI
bash <(curl -Ls https://raw.githubusercontent.com/eGamesAPI/remnawave-reverse-proxy/refs/heads/main/install_remnawave.sh)
```

4. Выберите язык → **1. Установка компонентов Remnawave** → **1. Установить панель и ноду на один сервер** → **1. Nginx**
5. Введите домены панели, подписки и ноды.
6. Выберите **ACME HTTP-01**, укажите email.

По завершении установки скрипт выведет:

```
=================================================
               УСТАНОВКА ЗАВЕРШЕНА!
=================================================
Панель доступна по адресу:
https://panel.example.com
-------------------------------------------------
Логин: LOGIN
Пароль: Password
-------------------------------------------------
Для повторного запуска менеджера:
remnawave_reverse
```

> [!WARNING]
> Сохраните URL панели и секретный ключ. Без них войти не получится.

Перейдите по ссылке в браузер — панель готова к работе.

---

### 3.3 Панель и нода на разных серверах

Следуйте [официальной документации Remnawave](https://docs.rw/docs/install/requirements).  
Скрипт eGamesAPI также поддерживает раздельную установку — выберите соответствующий пункт меню на шаге 4.

---

## 4. Дополнительные транспорты

### 4.1 XHTTP

XHTTP — транспорт поверх TLS/HTTPS с маскировкой под обычный веб-трафик.

#### Шаг 1 — Добавить inbound в Default профиль панели

```json
{
  "tag": "XHTTP",
  "listen": "/dev/shm/xrxh.socket,0666",
  "protocol": "vless",
  "settings": {
    "clients": [],
    "fallbacks": [],
    "decryption": "none"
  },
  "sniffing": {
    "enabled": true,
    "destOverride": ["http", "tls", "quic"]
  },
  "streamSettings": {
    "network": "xhttp",
    "xhttpSettings": {
      "mode": "auto",
      "path": "/xhttppath/",
      "extra": {
        "noSSEHeader": true,
        "xPaddingBytes": "100-1000",
        "scMaxBufferedPosts": 30,
        "scMaxEachPostBytes": 1000000,
        "scStreamUpServerSecs": "20-80"
      }
    }
  }
}
```

#### Шаг 2 — Добавить location в nginx.conf на ноде

```bash
cd /opt/remnanode && docker restart remnanode
nano /opt/remnanode/nginx.conf
```

Вставьте в конец файла:

```nginx
location /xhttppath/ {
    client_max_body_size 0;
    proxy_set_header X-Real-IP $proxy_protocol_addr;
    proxy_set_header X-Forwarded-For $proxy_protocol_addr;
    proxy_set_header Host $host;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection $connection_upgrade;
    proxy_http_version 1.1;
    client_body_timeout 5m;
    proxy_read_timeout 315s;
    proxy_send_timeout 5m;
    proxy_pass http://unix:/dev/shm/xrxh.socket;
}
```

Затем перезапустите nginx:

```bash
docker exec remnawave-nginx nginx -t && docker restart remnawave-nginx
```

#### Шаг 3 — Настроить хост в панели

В разделе **Хосты** создайте хост как на скриншоте:

![Настройка XHTTP хоста](screenshots/xhttp_host.png)

В **расширенных настройках хоста** → кнопка **xHTTP** → вставьте:

```json
{
  "xmux": {
    "cMaxReuseTimes": 0,
    "maxConcurrency": "16-32",
    "maxConnections": 0,
    "hKeepAlivePeriod": 0,
    "hMaxRequestTimes": "600-900",
    "hMaxReusableSecs": "1800-3000"
  },
  "noGRPCHeader": false,
  "xPaddingBytes": "100-1000",
  "scMaxEachPostBytes": 1000000,
  "scMinPostsIntervalMs": 30,
  "scStreamUpServerSecs": "20-80"
}
```

#### Шаг 4 — Перезапустить ноду

```bash
docker restart remnanode
```

---

### 4.2 Hysteria2

> [!IMPORTANT]
> Для Hysteria2 необходим TLS-сертификат, полученный через **ACME HTTP-01** при установке панели.

#### Шаг 1 — Подключить сертификаты к контейнеру ноды

```bash
# Замените DOMAIN на ваш домен ноды (например node.example.com)
sed -i 's|      - /dev/shm:/dev/shm:rw$|      - /dev/shm:/dev/shm:rw\n      - /etc/letsencrypt/live/DOMAIN/fullchain.pem:/var/lib/remnawave/configs/xray/ssl/cert.pem:ro\n      - /etc/letsencrypt/live/DOMAIN/privkey.pem:/var/lib/remnawave/configs/xray/ssl/cert.key:ro|g' /opt/remnanode/docker-compose.yml
```

#### Шаг 2 — Создать хук автообновления сертификата

```bash
nano /etc/letsencrypt/renewal-hooks/deploy/restart-remnanode.sh
```

Вставьте в файл:

```bash
#!/bin/bash
cd /opt/remnanode && docker compose restart remnanode
```

Затем нажмите `CTRL+O` → `Enter` → `CTRL+X`.

#### Шаг 3 — Открыть порт и поднять контейнер

```bash
chmod +x /etc/letsencrypt/renewal-hooks/deploy/restart-remnanode.sh
ufw allow 443/udp
cd /opt/remnanode
docker compose down && docker compose up -d
sleep 5
docker exec -it remnanode ls -la /var/lib/remnawave/configs/xray/ssl/
```

> [!TIP]
> Все команды можно скопировать разом — они выполнятся последовательно.

Убедитесь, что в выводе последней команды видны файлы `cert.pem` и `cert.key`.

#### Шаг 4 — Создать профиль Hysteria2 в панели

Перейдите: **Профили** → нажмите **+** → задайте имя `Hysteria2` → удалите дефолтный конфиг и вставьте:

```json
{
  "log": { "loglevel": "none" },
  "inbounds": [
    {
      "tag": "HYSTERIA-BBR",
      "port": 443,
      "listen": "0.0.0.0",
      "protocol": "hysteria",
      "settings": {
        "clients": [],
        "version": 2
      },
      "streamSettings": {
        "network": "hysteria",
        "security": "tls",
        "finalmask": {
          "quicParams": {
            "debug": false,
            "congestion": "bbr"
          }
        },
        "tlsSettings": {
          "alpn": ["h3"],
          "certificates": [
            {
              "keyFile": "/var/lib/remnawave/configs/xray/ssl/cert.key",
              "certificateFile": "/var/lib/remnawave/configs/xray/ssl/cert.pem"
            }
          ]
        },
        "hysteriaSettings": {
          "version": 2
        }
      }
    }
  ],
  "outbounds": [
    { "tag": "DIRECT", "protocol": "freedom" },
    { "tag": "BLOCK", "protocol": "blackhole" }
  ],
  "routing": {
    "rules": [
      { "ip": ["geoip:private"], "outboundTag": "BLOCK" },
      { "domain": ["geosite:private"], "outboundTag": "BLOCK" },
      { "protocol": ["bittorrent"], "outboundTag": "BLOCK" }
    ]
  }
}
```

Нажмите **Сохранить**.

#### Шаг 5 — Настроить хост

Заполните хост как на скриншоте:

![Настройка Hysteria2 хоста](screenshots/hysteria2_host.png)

Присвойте профиль ноде и добавьте во внутренний сквад.  
Финальные настройки JSON:

![Финальные настройки JSON](screenshots/hysteria2_json.png)

> [!NOTE]
> Hysteria2 работает только в клиентах **INCY** и **HAPP**.

---

## 5. Серверный роутинг

> ⚠️ Раздел в разработке. Следите за обновлениями репозитория.

---

## 6. Защита серверов

> ⚠️ Раздел в разработке. Следите за обновлениями репозитория.

---

## 🤝 Участие в проекте

Нашли ошибку или хотите дополнить гайд? Открывайте [Issue](../../issues) или Pull Request — буду рад.

---

<div align="center">

Сделано с ❤️ для русскоязычного комьюнити  
[⬆️ Наверх](#-remnawave-vpn--полное-руководство-по-установке)

</div>
