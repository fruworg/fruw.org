---
title: Динамический whitelist в caddy
description: envip
date: 2023-10-01T01:10:00+06:00
tags: [linux, caddy]
---

# envip
[Небольшая утилита](https://github.com/fruworg/envip), которая позволяет реализовать динамический whitelist для caddy:
Часть поддоменов будут доступны списку явно разрешённых IP.
Список IP автоматически пополняется при успешном прохождении basic auth на отдельном поддомене.

## Установка
### Caddy

В Caddyfile необходимо добавить секцию с поддоменом для обновления whitelist (необходимо заменить [строку basic auth](https://caddyserver.com/docs/caddyfile/directives/basicauth)):
```sh
wh.<your.domain> {
        @block {
                not remote_ip forwarded {$WHITE_LIST}
        }
        handle @block {
                basicauth {
                        Bob $2a$14$Zkx19XLiW6VYouLHR5NmfOFU0z2GTNmpkT/5qqR7hx4IjWJPDhjvG
                }
                reverse_proxy localhost:50009 {
                        header_up X-Real-IP {remote_host}
                }
        }
        respond "IP already added! :)"
}
```

Пример поддомена, доступ до которого разрешён только с IP из whitelist:
```sh
sub.<your.domain> {
        @allow {
                remote_ip forwarded {$WHITE_LIST}
        }
        handle @allow {
                reverse_proxy localhost:50001
        }
        respond 403
}
```

В `caddy.service` необходимо добавить следующую строку в секцию `[Service]`:
```sh
Environment="WHITE_LIST=127.0.0.1/8"
```

Необходимо перезагрузить `caddy`:
```shell
systemctl daemon-reload
systemctl restart caddy
```  

### envip
Установка бинарника:
```shell
mkdir /opt/envip
wget https://github.com/fruworg/envip/raw/main/envip -O /opt/envip/envip
```

Установка systemd-демона `envip`:
```shell
wget https://raw.githubusercontent.com/fruworg/envip/main/envip.service -O /etc/systemd/system/envip.service
```

Запуск envip:
```shell
systemctl enable --now envip
```
