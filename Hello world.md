## Написать скрипт на bash для мониторинга процесса test в среде linux. ##

Скрипт должен отвечать следующим требованиям:

1. Запускаться при запуске системы (предпочтительно написать юнит systemd в дополнение к скрипту)
   
2.  Отрабатывать каждую минуту

3. Если процесс запущен, то стучаться(по https) на https://test.com/monitoring/test/api
    
4.  Если процесс был перезапущен, писать в лог /var/log/monitoring.log (если процесс не запущен, то ничего не делать) 

5.  Если сервер мониторинга не доступен, так же писать в лог.


## Создаем файл myscript.sh ##

# nano myscript.sh #

#!/bin/bash

while :
do
    cpuUsage=$(top -bn1 | awk '/Cpu/ { print $2}')
    memUsage=$(free -m | awk '/Mem/{print $3}')

    echo "CPU Usage: $cpuUsage%"
    echo "Memory Usage: $memUsage MB"

    if systemctl is-active --quiet docker.service; then
        curl -X GET https://test.com/monitoring/test/api
        echo "Docker is running. Sent request to https://test.com/monitoring/test/api"
    else
        echo "Docker is not running."
    fi
    sleep 60
done


## Переходим в папку /etc/systemd/system ##
## Создаем сервис myscript.service ##
[Unit]
Description=MyBashScript
After=syslog.target
[Service]
ExecStart=/bin/bash '/home/admin1/myscript.sh'
Type=forking
[Install]
WantedBy=multi-user.target
Alias=bash.service

# Обновите конфигурацию systemd: #

systemctl daemon-reload

# Добавьте сервис в автозагрузку и запустите его: #

systemctl enable myscript.service

# Запускаем: #
systemctl start myscript.service


