# Network setup



Настройка сетки с помощью wireguard.



Установка происходит на Debian 11.

## Wireguard setup

```
apt-get update && apt-get upgrade
apt-get install wireguard
```

Генерируем ключи:

```
sudo -i
cd /etc/wireguard
umask 077
wg genkey > server.key
wg pubkey < server.key > server.pub
```

Создаём конфиг **wg0.conf** и добавляем следующие строки:
vim /etc/wireguard/wg0.conf

```
[Interface]
Address = 10.254.0.1/24
ListenPort = 49999
SaveConfig = True
```


Добавляем в **wg0.conf** приватный ключ:

```
echo "PrivateKey = $(cat server.key)" >> /etc/wireguard/wg0.conf
#Для fish
echo "PrivateKey = "(cat server.key) >> /etc/wireguard/wg0.conf
```




