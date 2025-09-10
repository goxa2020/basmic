# Разработка системы автономной ориентации и посадки БВС на маркированную площадку

## Использованные технологии
- Компьютерное зрение: OpenCV для обработки изображений
- ArUco – позиционирования робототехнических систем с использованием компьютерного зрения
- Алгоритм управления: PID-регуляторы для точного удержания позиции и коррекции траектории.

## Оборудование и комплектующие
- Orange PI
- Веб-камера низкого разрешения Live ACM138
- Сервопривод TowerPro SG92r

## Программное обеспечение
- Языки: Python, C++
- Фреймворки: Visual Studio Code
- Моделирование: Gazebo9

### Как работает
Универсальная система посадки состоит из вебкамеры и микрокомпьютер Orange Pi. Камера установлена на гиростабилизированный подвес, который необходим для стабилизации линии визирования в диапазоне от 0 до 45 градусов относительно вертикали вне зависимости от маневров носителя.  Поиск метки осуществляется в положении линии визирования 45 градусов. После обнаружения метки осуществляется движение в ее сторону с удержанием метки в центре поля зрения и плавным изменением угла наклона камеры. При достижении вертикального положения линии визирования зависаем над центром метки и выполняем коррекцию курса носителя относительно площадки с меткой. Далее выполняем процедуру снижения. Для увеличения точности посадки используется маленький маркер внутри основного. Система посадки работает независимо от автопилота носителя, что позволяет считать данное решение универсальным.


# Установка
Этот проект был реализован на компьютере с Ubuntu 18.04 с использованием ROS, Gazebo и OpenCV. Для его корректной работы необходимо установить следующее программное обеспечение. 

Вы можете либо загрузить программное обеспечение, либо клонировать его непосредственно со своего терминала. Чтобы использовать последний, необходимо установить [Git](https://git-scm.com) с помощью командной строки.:
```bash
#Установка Git
sudo apt install git

#Клонирование проекта
git clone https://github.com/Kenil16/master_project.git
```

Теперь проект можно установить со всеми зависимостями, запустив скрипт [init_setup.bash](https://github.com/goxa2020/basmic/blob/master/software/ros_workspace/init_setup.bash). Теперь смените каталог и запустите скрипт инициализации:
```bash
#Change directory and execute the initialization script
cd software/ros_workspace/
. ./init_setup.bash
```

Установка и настройка системы завершены. Если вы хотите установить необходимое программное обеспечение по отдельности, выполните следующие шаги.

Установка зависимостей PX4:
```bash
#Установка пакетов через apt
sudo apt install astyle build-essential ccache clang clang-tidy cmake cppcheck doxygen file g++ gcc gdb git lcov make ninja-build python3 python3-dev python3-pip python3-setuptools python3-wheel rsync shellcheck unzip xsltproc zip libeigen3-dev libopencv-dev libroscpp-dev protobuf-compiler python-pip python3-pip ninja-build gstreamer1.0-plugins-bad gstreamer1.0-plugins-base gstreamer10-plugins-good gstreamer1.0-plugins-ugly libgstreamer-plugins-base1.0-dev libgstrtspserver-1.0-dev xvfb python-rosdep2

#Установка пакетов через pip
pip install --user argparse cerberus empy jinja2 numpy packaging pandas psutil pygments pyros-genmsg pyserial pyulog pyyaml setuptools six toml wheel rosdep

#Установка пакетов через pip3
pip3 install --user --upgrade empy jinja2 numpy packaging pyros-genmsg toml pyyaml pymavlink
```

[ROS](http://wiki.ros.org/Installation/Ubuntu) melodic используется в этом проекте и может быть установлен вместе с Gazebo:
```bash
#инициализация рабочего пространства
sudo rosdep init
sudo rosdep update

#Настройка вашего sources.list
$ sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'

#Настройка ключей
$ sudo apt-key adv --keyserver 'hkp://keyserver.ubuntu.com:80' --recv-key C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654

#Установка
$ sudo apt update
$ sudo apt install ros-melodic-desktop-full
```

[Mavros](http://wiki.ros.org/mavros) можно установить с помощью:
```bash
#Установить mavros вместе с дополнительными пакетами
sudo apt install ros-melodic-mavros ros-melodic-mavros-extras -y
```

Установка зависимостей для запуска программных скриптов:
```bash
sudo apt update

sudo apt install ros-melodic-hector-gazebo-plugins dvipng

pip install pathlib numpy pandas opencv-contrib-python scipy pymavlink matplotlib
```

Установка [Gazebo](http://gazebosim.org/tutorials?cat=install&tut=install_ubuntu&ver=9.0)  (протестировано на версии 9.16.0)
```bash
#Настройте свой компьютер для приема программного обеспечения с packages.osrfoundation.org.
sudo sh -c 'echo "deb http://packages.osrfoundation.org/gazebo/ubuntu-stable `lsb_release -cs` main" > /etc/apt/sources.list.d/gazebo-stable.list'

#Настройка ключей
wget https://packages.osrfoundation.org/gazebo.key -O - | sudo apt-key add -

#Установка
sudo apt-get update
sudo apt-get install gazebo9

#В противном случае Gazebo не будет работать.
sudo apt upgrade libignition-math2
```

Необходимо установить [PX4](https://docs.px4.io/master/en/dev_setup/dev_env_linux_ubuntu.html), и эта установка предполагает, что PX4 SITL будет установлен в каталог /home. Можно легко создать другой каталог установки, но тогда необходимо будет соответствующим образом изменить символические ссылки:
```bash
#Инициализируйте среду PX4 SITL из вашей домашней папки
cd ~
mkdir PX4_SITL

cd PX4_SITL
git clone https://github.com/PX4/Firmware.git --branch v1.11.0

cd Firmware
git submodule update --init --recursive

#Создать символическую ссылку на модели пользовательского ПО в PX4 SITL
cd software/ros_workspace/
ln -s $PWD/PX4-software/init.d-posix/* /home/$USER/PX4_SITL/Firmware/ROMFS/px4fmu_common/init.d-posix/
ln -s $PWD/PX4-software/mixers/* /home/$USER/PX4_SITL/Firmware/ROMFS/px4fmu_common/mixers/
ln -s $PWD/PX4-software/models/* /home/$USER/PX4_SITL/Firmware/Tools/sitl_gazebo/models/
ln -s $PWD/PX4-software/worlds/* /home/$USER/PX4_SITL/Firmware/Tools/sitl_gazebo/worlds/
ln -s $PWD/PX4-software/launch/* /home/$USER/PX4_SITL/Firmware/launch/

#Сборка прошивки PX4 SITL
cd /home/$USER/PX4_SITL/Firmware
DONT_RUN=1 make px4_sitl_default gazebo
```

# Как использовать
Чтобы настроить систему, необходимо запустить следующий файл конфигурации с терминала, чтобы подключить PX4 к среде ROS:
```bash
cd software/ros_workspace/
. ./setup_gazebo.bash
```

Для запуска программы необходимо выполнить следующую команду с вашего терминала:
```bash
#Run roslaunch which starts all the nodes in the system 
roslaunch px4 gazebo_sim_v1.0.launch worlds:=optitrack_big_board_onepattern_missing_markers.world drone_control_args:="idle" x:=-21.0 y:=0
```
Здесь вы можете определить мир из нескольких возможных [настроек](https://github.com/Kenil16/master_project/tree/master/software/ros_workspace/PX4-software/worlds). По умолчанию БПЛА будет находиться в состоянии ожидания, но при желании его можно изменить на какое-либо задание.
