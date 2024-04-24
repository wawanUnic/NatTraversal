# NatTraversal

Код отрабатывался на платформе Raspberry. Порядок установки:

Форматируем флешку в Windows после Linux: diskpart, list disk, select disk X, clean, create partition primary, format quick, assign

Ставим обычную Raspbian 32bit (от 2020-02-13, но не от 2022!). Логин/Пароль: pi, raspberry (можно и без раб.стола) В новой версии PiImager необходимо сразу задавать первичные параметры (SSH, username и проч.). И если там задать Логин/Пароль, то сразу можно подключиться удаленно, а иначе придется подключать клаву/мышь для задания Логина/Пароля.

Передергиваем карту памяти для перезапуска

Создаем файл ssh без расширения в разделе boot. После работы Raspbian этот файл исчезнет. В новых версиях это можно не далать, а задать при накатывании образа.

У плат Rpi1, Rpi2 и др. возможен обмен serial-115200-8n1. Пины 4 и 5. Правка config.txt (необязательно): enable_uart=1. Потом через Putty.

У старых обрзов Raspbian обновление делается через sudo apt update, sudo apt upgrade

Статический IP-адрес (только в Linux): sudo nano /etc/dhcpcd.conf в конце: nodhcp interface eth0 static ip_address=192.168.0.243/24 static routers=192.168.0.1 static domain_name_servers=192.168.0.1

Подключаем Вай-Фай (еще при создании SD-карты). Создаем файл: /boot/wpa_supplicant.conf. Пишем в него: ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev update_config=1 network={ ssid="Guest" psk="Nero12345" }

Подключаем Вай-Фай (уже в Linux): sudo nano /etc/wpa_supplicant/wpa_supplicant.conf Добавляем в конец: network={ ssid="..." psk="..." } Делаем sudo reboot

Ставим удаленный рабочий стол (только если установлена система без стола!): sudo apt-get install tightvncserver xrdp (Необходимо подключение к Интернет, работает по проту 3389, запускается через Win+R, mstsc) (Или воспользуемся уже имеющимся без установки: (через ssh) sudo raspi-config, включить vnc, потом в терминале vncserver)

Устанавливаем mDNS (после чего веб-визуализация будет по адресу - myname:8080) (В новых версиях это можно не далать, а задать при накатывании образа): sudo apt-get update (необходимо подключение к Интернет. Возможно со второго раза...) sudo apt-get install libnss-mdns - часто уже установлена sudo nano /etc/hostname - вписать новое имя вместо "raspberry" sudo nano /etc/hosts - вписать новое имя вместо "raspberry" в последней строке sudo /etc/init.d/hostname - обычно тут ошибка sudo reboot

Через приложение CodeSYS устанавливаем в систему Raspbian объект GateWay и агент RunTime (Можно через interScada). Если нет ответа от runTime, то sudo reboot и sudo service codesyscontrol restart. Необходимо будет создать своего пользователя (обычно pi, pi3)

Несколько прав меняем через ssh: Изменяем права доступа к файлу агента (0777?): sudo chmod 0777 /etc/CODESYSControl_User.cfg Добавляем в файл /etc/CODESYSControl_User.cfg: [SysProcess] Command=AllowAll (типа все команды на исполнение. А можно Command.0=reboot Command.1=echo)

Права доступа на папку исполнения CodeSys: sudo chmod -R 0777 /var/opt/codesys Изменить порт визуализации: /etc/CODESYSControl.cfg: [CmpWebServer] WebServerPortNr=80 Если используешь стандартный uart-RPi, то нужно запретить вход в оболочку в raspi-config (38600 б/с). А хардовый порт нужно оставить! Если включаешь файл Питона в проект, то ему надо дать права на запуск: sudo chmod 0777 /var/opt/codesys/PlcLogic/Application/file.py Необходима перезагрузка: sudo reboot (рабочая папка: /var/opt/codesys) (загруженные в проект файлы ложаться сюда: /var/opt/codesys/PlcLogic/Application) (фавикон для визуализации (можно положить уже в CodeSys): /var/opt/codesys/PlcLogic/visu/favicon.ico)

Автоматический перезапуск агента RunTime. Добавляем в файл /etc/CODESYSControl_User.cfg: [SysProcess] Command=AllowAll Добавляем в проект библиотеку SysProcess. Пишем в проекте: pResult: POINTER TO SysProcess.SysTypes.RTS_IEC_RESULT; SysProcess.SysProcessExecuteCommand('sudo service codesyscontrol restart', pResult);

// Устанавлием сервер доступа к файлам
sudo apt install vsftpd
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/vsftpd.pem -out /etc/ssl/private/vsftpd.pem
sudo nano /etc/vsftpd.conf
	anonymous_enable=NO
	local_enable=YES
	write_enable=YES
	rsa_cert_file=/etc/ssl/private/vsftpd.pem
	rsa_private_key_file=/etc/ssl/private/vsftpd.pem
	ssl_enable=YES
systemctl restart vsftpd
systemctl status vsftpd

Для версия 4 использовано определение глобального адреса. Для этого используется сторонний сайт http://httpbin.org/get. Который возвращает JSON сообщение в ответ:
{
    "args": {},
    "headers": {
        "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7",
        "Accept-Encoding": "gzip, deflate",
        "Accept-Language": "ru,en;q=0.9,en-GB;q=0.8,en-US;q=0.7",
        "Host": "httpbin.org",
        "Upgrade-Insecure-Requests": "1",
        "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36 Edg/124.0.0.0",
        "X-Amzn-Trace-Id": "Root=1-6628b5f4-330f7b457bad93857bf10cdf"
    },
    "origin": "78.10.222.186",
    "url": "http://httpbin.org/get"
}

Переключать направление движения пакетов можно через перечисление mode. Оно глобально для двух устройств и находится во вкладке POU (Устройства-POU-EdgeGateway-RaspberryPi-LinuxARM64).

Настройка HTTPS: /etc/CODESYSControl.cfg :[CmpWebServer] ConnectionType=3 WebServerPortNr=80 WebServerSecurePortNr=443. Сгенерировать сертификат через SequreAgent. в PLC Shell: cert-gendhparams 1024 Тогда он будет работать хотя бы с IE и Firefox. Об этом: https://forge.codesys.com/forge/talk/Runtime/thread/0451764381/

