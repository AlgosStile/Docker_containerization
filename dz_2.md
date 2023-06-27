# Механизмы контрольных групп
## Задание 1:
## 1. запустить контейнер с ubuntu, используя механизм LXC
## 2. ограничить контейнер 256 Мб ОЗУ и проверить, что ограничение работает
## 3. добавить автозапуск контейнеру, перезагрузить ОС и убедиться, что контейнер действительно запустился самостоятельно
## 4. при создании указать файл, куда записывать логи
## 5. после перезагрузки проанализировать логи

## Задание 2*: настроить автоматическую маршрутизацию между контейнерами. Адреса можно взять: 10.0.12.0/24 и 10.0.13.0/24.

### Решение:
Установливаем LXC

Чтобы установить LXC на Ubuntu, открываем терминал и выполняем команду:

sudo apt-get update && sudo apt-get install lxc

Создаем контейнер с именем "ubuntu-container" с помощью следующей команды:

sudo lxc-create -n ubuntu-container -t ubuntu

Запускаем контейнер с помощью команды:

sudo lxc-start -n ubuntu-container

Ограничение памяти
Остановка контейнер с помощью команды:

sudo lxc-stop -n ubuntu-container

Редактируем файл конфигурации контейнера с помощью например nano, добавив следующую строку:

lxc.cgroup.memory.limit_in_bytes = 256M

Файл находится по пути /var/lib/lxc/ubuntu-container/config.

Запускаем контейнер снова с помощью команды:
sudo lxc-start -n ubuntu-container

Проверяем, что ограничение памяти работает, выполнив команду:
sudo lxc-info -n ubuntu-container

Видим вывод указывающий на ограничение памяти, которое установили.

Автозапуск контейнера
Останавливаем контейнер с помощью команды:

sudo lxc-stop -n ubuntu-container

Создаем файл конфигурации для автозапуска контейнера в директории /etc/lxc/auto.
Файл может иметь любое имя, например, ubuntu-container.conf. 
Добавляем любые строки в файл напрмиер:

lxc.start.auto = 1
lxc.start.delay = 5
lxc.autodev = 1
lxc.log.file = /var/log/my-container.log

Запускаем контейнер снова с помощью команды:
sudo lxc-start -n ubuntu-container -d

Перезагружаем систему и видим, что контейнер запустился автоматически.

Логирование
Останавливаем контейнер с помощью команды:

sudo lxc-stop -n ubuntu-container

Редактируем файл конфигурации контейнера, добавив следующую строку:
lxc.log.file = /var/log/lxc/ubuntu-container.log
Файл находится по пути /var/lib/lxc/ubuntu-container/config.

Запускаем контейнер снова с помощью команды:

sudo lxc-start -n ubuntu-container -d

После перезагрузки системы анализируем лог-файл, чтобы убедиться, что контейнер запускается автоматически и работает как ожидается. 
Лог-файл находится по пути /var/log/lxc/ubuntu-container.log.

Автоматическая маршрутизация между контейнерами: 
Создаем два новых контейнера с именами "ubuntu-container1" и "ubuntu-container2" с помощью следующих команд:
sudo lxc-create -n ubuntu-container1 -t ubuntu
sudo lxc-create -n ubuntu-container2 -t ubuntu

Запускаем оба контейнера:
sudo lxc-start -n ubuntu-container1
sudo lxc-start -n ubuntu-container2

Входиим в оба контейнера и настраиваем их сетевые интерфейсы:

sudo lxc-attach -n ubuntu-container1
echo "auto eth0\niface eth0 inet static\naddress 10.0.12.1\nnetmask 255.255.255.0\ngateway 10.0.12.254" > /etc/network/interfaces.d/eth0.cfg
service networking restart
exit

sudo lxc-attach -n ubuntu-container2
echo "auto eth0\niface eth0 inet static\naddress 10.0.13.1\nnetmask 255.255.255.0\ngateway 10.0.13.254" > /etc/network/interfaces.d/eth0.cfg
service networking restart
exit

Здесь мы прописываем статические IP-адреса для каждого контейнера. Gateway адреса указываются так, чтобы они направляли трафик на родительский хост.

Настраиваем IP-адрес для сетевого интерфейса родительского хоста, который будет использоваться в качестве шлюза по умолчанию для контейнеров:

sudo ifconfig lxcbr0 10.0.12.254/24 up
sudo ifconfig lxcbr1 10.0.13.254/24 up

Выполняем следующие команды на каждом контейнере, чтобы добавить маршрут до другого контейнера:
sudo ip route add 10.0.13.0/24 via 10.0.12.1 dev eth0
sudo ip route add 10.0.12.0/24 via 10.0.13.1 dev eth0

Тестим соединение между контейнерами, выполнив команду ping из одного контейнера в другой:
sudo lxc-attach -n ubuntu-container1
ping 10.0.13.1
Тут получаем ответ от второго контейнера. 
Вроде все).

