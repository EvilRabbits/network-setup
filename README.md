# Network setup



Настройка сетки с помощью wireguard.



Установка происходит на Debian 11.

## Wireguard setup

```
apt-get update && apt-get upgrade
apt-get install wireguard
```

Генерируем ключи для сервера:

```
sudo -i
cd /etc/wireguard
umask 077
wg genkey > server.key
wg pubkey < server.key > server.pub
```

Создаём конфиг **wg0.conf** и добавляем следующие строки:

```
vim /etc/wireguard/wg0.conf
```

```
[Interface]
Address = 10.254.0.1/24
ListenPort = 49999
```

Добавляем в **wg0.conf** приватный ключ сервера:

```
echo "PrivateKey = $(cat server.key)" >> /etc/wireguard/wg0.conf
# Для fish
echo "PrivateKey = "(cat server.key) >> /etc/wireguard/wg0.conf
```

Включаем **NAT**:

```
PostUp = iptables -I FORWARD 1 -i wg0 -j ACCEPT; iptables -t nat -I POSTROUTING 1 -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
```

Если сервер не использует **eth0**, то вместо **eth0** вставляем название, которое можно получить из вывода:

```
ip -o -4 route list default | cut -d" " -f5
```

Включаем **IPv4 Forwarding**:

Раскомментируем ручками строку с **net.ipv4.ip_forward=1** в **/etc/sysctl.conf** или с помощью комманды:

```
sed -i 's/#net.ipv4.ip_forward=1/net.ipv4.ip_forward=1/' /etc/sysctl.conf
sysctl -p
```

Запускаем Wireguard-server и влючаем демона:

```
wg-quick up wg0
systemctl enable wg-quick@wg0
exit
```

Создаем директорию для конфига клиента:

```
cd ~
umask 077
mkdir wireguard && cd wireguard
mkdir peer1 && cd peer1
```

Генерируем ключи для клиента:

```
wg genkey > peer1.key
wg pubkey < peer1.key > peer1.pub
wg genpsk > peer1.psk
```

Создаём конфиг **peer1.conf** и добавляем следующие строки:

```
vim peer1.conf
```

Добавляем в **peer1.conf** приватный ключ клиента:

```
echo "PrivateKey = $(cat peer1.key)" >> peer1.conf
# Для fish
echo "PrivateKey = "(cat server.key) >> peer1.conf
```

Также добавляем детали подключения к серверу:

```
echo "[Peer]" >> peer1.conf
# IP - адрес сервера  
echo "Endpoint = IP" >> peer1.conf
echo "AllowedIPs = 10.254.0.0/24" >> peer1.conf
echo "PublicKey = $(sudo cat /etc/wireguard/server.pub)" >> peer1.conf
# Для fish
echo "PublicKey = "(sudo cat /etc/wireguard/server.pub) >> peer1.conf

echo "PresharedKey = $(cat peer1.psk)" >> peer1.conf
# Для fish
echo "PresharedKey = "(cat peer1.psk) >> peer1.conf
```

Добавляем клиента в конфиг сервера:

```
sudo -i
cd /etc/wireguard
echo >> wg0.conf && echo "[Peer]" >> wg0.conf
echo "AllowedIPs = 10.254.0.2/32" >> wg0.conf

# user - название пользователя
echo "PublicKey = $(cat /home/user/wireguard/peer1/peer1.pub)" >> wg0.conf
# Для fish
echo "PublicKey = "(cat /home/user/wireguard/peer1/peer1.pub) >> wg0.conf

echo "PresharedKey = $(cat /home/user/wireguard/peer1/peer1.psk)" >> wg0.conf
# Для fish
echo "PresharedKey = "(cat /home/user/wireguard/peer1/peer1.psk) >> wg0.conf
```

Перезапускаем Wireguard-server, чтобы применить изменения конфига:

```
wg-quick down wg0
wg-quick up wg0
exit
```

Изменяем конфиг клиента **peer1.conf**. Добавляем приватный ключ клиента **peer1.key**, порт 49999 и меням **AllowedIPs** на 0.0.0.0/0:

```
[Interface]
Address = 10.254.0.2/32
PrivateKey = CFQjjugFZybM1df10mKva2wYjLRe4PhghitZMVRTTh4Tlw=

[Peer]
Endpoint = 123.45.67.89:49999
AllowedIPs = 0.0.0.0/0
PublicKey = gu/22X5fwJ123sd8u2ropqvR3sSlsdfl2342mn1FDf=
```






