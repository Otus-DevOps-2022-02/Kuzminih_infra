bastion_IP = 51.250.74.58  
someinternalhost_IP = 10.128.0.33

# Kuzminih_infra
Kuzminih Infra repository
## HW_5 connection via jumphost over ssh

### Пример с yandex cloud-VM

Созданы две ВМ в облаке яндекс.
- ВМ1 является jump host все подключения будут осущетсвлятся через неё, подключен внешний ip адрес (ip1) и подключен к внутреней сети
- ВМ2 имеет одно подключение внутреней сети (internal-ip)

### Настройка локальной ВМ с которой происходит подключение
Генерируем ключ
`ssh-keygen -t -f ~/.ssh/appuser -C appuser -P ""`
[-t dsa | ecdsa | ecdsa-sk | ed25519 | ed25519-sk | rsa]
[-f keyfile]
[-C comment]
[-P passphrase]

Проверям форвардинг
```
eval `ssh-agent -s`
```

Перечисляет параметры открытого ключа всех удостоверений, представленных в настоящее время агентом.
```
ssh-add -L
```

Добовляем ключ в форвардинг
```
ssh-add ~/.ssh/appuser
```
### Первый вариант подключения (ручной)
`ssh -i ~/.ssh/appuser -A appuser@<ip1>`
[-i identity_file]
[-A Enables forwarding of connections from an authentication agent such as ssh-agent(1). This can also be specified on a per-host basis in a configuration file]

`ssh <internal-ip>`
---
### Второй вариант
`ssh -v -i ~/.ssh/appuser -J appuser@<ip> appuser@<internal-ip>`
Указывается jumhost, через который происходит подключение к ВМ2. Без указания пользователя для ВМ2 подключение не удалось.
[-i identity_file]
[-v Verbose mode] Для дебага.
[-J destination]
---
### Рефакторинг
Добовляем конфиг для автоматического подключения по имени.
```
vi ~/.ssh/config
```
Вариант №1
```
host bastion
  Hostname 51.250.74.67
  User appuser
  IdentityFile /home/fedor/.ssh/appuser

host someinternalhost
  Hostname 10.128.0.33
  User appuser
  ProxyCommand ssh -W %h:%p bastion
  IdentityFile /home/fedor/.ssh/appuser
```
---
### Ссылки на доку ssh
https://en.wikibooks.org/wiki/OpenSSH/Cookbook/Proxies_and_Jump_Hosts
https://docs.rackspace.com/blog/speeding-up-ssh-session-creation/
VPN https://rtfm.co.ua/ssh-podklyuchenie-v-privatnuyu-set-cherez-bastion-i-nemnogo-pro-multiplexing/

### Установка
```
deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/4.4 multiverse
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/4.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.4.listcurl -fsSL https://www.mongodb.org/static/pgp/server-4.4.asc | sudo apt-key add -apt updatesudo apt update
ip r
echo "deb http://repo.pritunl.com/stable/apt focal main" | sudo tee /etc/apt/sources.list.d/pritunl.list
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com --recv 9DA31620334BD75D9DCB49F368818C72E52529D4
sudo apt update
gpg --keyserver hkp://keyserver.ubuntu.com --recv 7AE645C0CF8E292A
gpg --export --armor 7AE645C0CF8E292A | sudo apt-key add -
sudo apt update
sudo apt install pritunl mongodb-org
systemctl start pritunl mongod
sudo systemctl start pritunl mongod
sudo systemctl status pritunl.service
sudo ss -tlnup
sudo pritunl setup-key
sudo pritunl default-password
```
