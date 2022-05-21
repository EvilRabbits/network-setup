# Network setup



Настройка сетки с помощью wireguard.



Установка происходит на Debian 11.



## Wireguard setup

```
apt update && apt upgrade -y
apt install -y wireguard
```

Генерируем ключи:

```
wg genkey | tee /etc/wireguard/privatekey | wg pubkey | tee /etc/wireguard/publickey
```


