Зачем иметь своё? Очень просто.
Чтобы разбираться в вопросе и контролировать ситуацию со своим доступом к сети.

### Настройка сервера
Установка зависимостей
```sh
sudo apt install -y wireguard qrencode curl git
```

Генерируем ключи
```sh
wg genkey | tee /etc/wireguard/privatekey | wg pubkey | tee /etc/wireguard/publickey
```

Права на приватный ключ сервера
```sh
sudo chmod 600 /etc/wireguard/privatekey
```

Редактируем файл */etc/wireguard/wg0.conf*, добавляем следующее содержимое:
```conf
[Interface]
PrivateKey = <privatekey>
Address = 10.10.0.1/24
ListenPort = 51832
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
```
Интерфейс *eth0* в качестве примера.

Если __НЕ__ нужно заворачивать весь трафик через сервер, то настраивать форвардинг не нужно.
Настраиваем IP-Forwarding
```sh
echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf
```
И применяем настройки
```sh
sysctl -p
```

Включаем, стратуем, и проверяем статус сервиса *wg0.service*
```sh
systemctl enable wg-quick@wg0.service
systemctl start wg-quick@wg0.service
systemctl status wg-quick@wg0.service
```

##### Создаем клиента
Генерируем ключи
```sh
wg genkey | tee /etc/wireguard/peer0_privatekey | wg pubkey | tee /etc/wireguard/peer0_publickey
```

Добавляем в конфигурационный файл сервера клиента:
```conf
[Peer]
PublicKey = <peer0_publickey>
AllowedIPs = 10.10.0.2/32
```

Перезапускаем и проверяем статус службы
```sh
systemctl restart wg-quick@wg0
systemctl status wg-quick@wg0
```

### Настройка клиента
Установка
Для Mac OS [пакет](https://apps.apple.com/ru/app/wireguard/id1451685025?mt=12) из AppStore.
Для [Windows](https://download.wireguard.com/windows-client/wireguard-installer.exe)
Для единственно нормальных операционных систем:
```sh
sudo apt install -y wireguard-tools
```

Создаем конфигурационный файл peer0.conf
```conf
[Interface]
PrivateKey = <CLIENT-PRIVATE-KEY>
Address = 10.10.0.2/32
DNS = 8.8.8.8

[Peer]
PublicKey = <SERVER-PUBKEY>
Endpoint = <SERVER-IP>:51832
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 20
```

Подключиться:
```sh
wg-quick up wg0
```

Отключиться:
```sh
wg-quick down wg0
```

### Дополнительно
Мобильное приложение на [IOS](https://apps.apple.com/ru/app/wireguard/id1441195209) и [Android](https://play.google.com/store/apps/details?id=com.wireguard.android&hl=ru&gl=US&pli=1) позволяет отсканировать QR-Код и быстро настроить клиент.

Устанавливаем пакет
```sh
sudo apt install -y qrencode
```

Команда для генерации QR-кода
```sh
qrencode -t ansiutf8 -r peer0.conf
```