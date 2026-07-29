# TorTTY
## проблема:
Когда обычный SSH ломают на уровне протокола:

	•	ping работает. 
	•	TCP 22 открыт. 
	•	но соединение рвётся. 

👉 значит фильтрация DPI. 

## Решение:

👉 спрятать SSH внутри Tor. 
👉 использовать .onion hidden service. 
## Архитектура  
[ ТЫ ]  
   ↓ (Tor SOCKS)  
127.0.0.1:9050  
   ↓  
[ Tor сеть ]  
   ↓  
.onion адрес  
   ↓  
[ SSH сервер ]  
👉 SSH больше не виден провайдеру вообще
👉 выглядит как обычный Tor-трафик
⸻
## реализация. 
### ⚙️ Часть 1 — сервер (самое важное)  

1. Установка Tor
```
sudo apt install tor -y
```
3. Настройка hidden service
```bash
sudo nano /etc/tor/torrc
```
Добавить:
```bash
HiddenServiceDir /var/lib/tor/ssh_hidden/
HiddenServicePort 22 127.0.0.1:22
```
4. Перезапуск
```bash
sudo systemctl restart tor
```
6. Получение onion-адреса
```bash
sudo cat /var/lib/tor/ssh_hidden/hostname
```
👉 получаешь:
```bash
xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx.onion
```
⚠️ Важный момент
	•	это НЕ DNS
	•	это НЕ IP
	•	это криптографический адрес

👉 его нельзя “угадать” или отсканировать

### Часть 2 — клиент (твоя машина)

1. Tor с мостами (обход блокировок)
```bash
sudo nano /etc/tor/torrcl
```
Пример:
```bash
UseBridges 1
ClientTransportPlugin obfs4 exec /usr/bin/obfs4proxy
```
```bash
Bridge obfs4 77.85.159.177:7000 ...
Bridge obfs4 207.58.153.218:48954 ...
SocksPort 127.0.0.1:9050  
```
2. Проверка
```bash
ss -tulnp | grep 9050
```
👉 должно быть:
```bash
127.0.0.1:9050
```
3. Запуск
```bash
sudo systemctl restart tor@default
```
Ждёшь:
```bash
Bootstrapped 100% 
```
### Часть 3 — подключение через SOCKS
Установка ncat
```bash
sudo apt install ncat -y
```
Команда подключения
```bash
ssh -o ProxyCommand="ncat --proxy 127.0.0.1:9050 --proxy-type socks5 %h %p" user@your.onion
```
### Часть 4 — нормализация
```bash
~/.ssh/config
nano ~/.ssh/config
Host myserver
    HostName your.onion
    User rick
    ProxyCommand ncat --proxy 127.0.0.1:9050 --proxy-type socks5 %h %p
    IdentityFile ~/.ssh/id_ed25519
    ObscureKeystrokeTiming no
```
### Использование
```bash
ssh myserver
```







