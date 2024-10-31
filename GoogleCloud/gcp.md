# [←](./../README.md) <a id="home"></a> Google Cloud Virtual Machine

## Table of Contents:
- [Google Cloud Virtual Machine](#cloud)
- [JDK installation](#jdk)
- [Docker installation](#docker)
- [Linux SWAP settings](#swap)

----

## [↑](#home) <a id="cloud"></a> Google Cloud Virtual Machine
Создадим бесплатную виртуальную машину в **Google Cloud**.

Все манипуляции с Google Cloud выполняются при помощи **"[Google Cloud Console](https://console.cloud.google.com/)"**.\
В левом верхнем углу доступно меню, при помощи которого мы можем выполнять различные действия.

В самом начале использования Google Cloud необходимо настроить **Billing**.\
Для этого в меню выберем пункт **"Billing"**.\
В списке увидим один биллинг аккаунт (например, "My Billing Account"). Откроем его.\
В верхней части страницы нажмём кнопку **"Manage billing account"**.\
После чего появится кнопка **"Rename Billing Account"**. Нажмём на неё и выберем новое имя.\
Например: **"GCP Free Tier"**.

Далее, настроим **"Compute Engine"** (так называются VM в терминах Google Cloud).\
В верхнем левом углу раскроем меню и перейдём в раздел **"Compute Engine"**.

Перед первым использованием Compute Engine необходимо включить данный функционал.\
Для этого нужно нажать кнопку **"Enable"**.\
Необходимо связать Compute Engine с биллинг аккаунтом.
Выберем наш GCP Free Tier биллинг аккаунт.
После этого нужно будет подождать, когда Compute Engine активируется для нашего аккаунта.

После активации Compute Engine можно использовать кнопку **"Create Instance"** для создания новой VM.

При создании укажем имя для новой VM, например **"instance-free-tier"**.\
Далее настроим VM согласно условиям, указанным в документе **"[Google Cloud Features: Compute](https://cloud.google.com/free/docs/free-cloud-features#compute)"**:
- **"Region"** должен быть **ТОЛЬКО** из указанного перечня. Например: ``Iowa: us-central1``. 
- **"General Purpose Machine configuration"** согласно документу. Например: ``e2``.
- **"Machine type"** указать согласно документу. Например: ``e2-micro``.
- **"Availability policies"** оставляем в значении по умолчанию: ``Standard``.
- Для **"Boot disk"** нажать **Change** и выбрать размер согласно документу (например: ``30``) и **Boot Disk Type** (например, ``Standard Persistent Disk``)
- В разделе **Firewall** разрешить трафик для HTTP и HTTPS

Нажмём кнопку **Create** для завершения создания **VM**.

После создания в списке **VM instances** появится новая VM с указанным ранее названием.\
Для подключения к ней достаточно нажать на кнопку **"SSH"** в колонке **"Connect"**.

----

## [↑](#home) <a id="jdk"></a> JDK installation
Прежде всего необходимо обновить индекс доступных пакетов:
> sudo apt-get update

Для начала, установки **JDK**, т.к. Java будет часто нужна. Нужно определиться с версией.\
Например, проверим в **"[Kafka: Requirements](https://kafka.apache.org/documentation/#java)"** указана JDK 17.\
Для Gradle нам понадобится не ниже JDK 17, согласно **"[Gradle Compatibility Matrix](https://docs.gradle.org/current/userguide/compatibility.html#java_runtime)"**.

Проверим, какие пакеты JDK доступны после обновления индекса:
> apt-cache search openjdk | grep 17

Найдём в списке openjdk и выполним его установку:
> sudo apt install openjdk-17-jdk

После установки должна быть возможность узнать версию JDK:
> java --version

----

## [↑](#home) <a id="docker"></a> Docker installation
Установим Docker на нашу виртуальную машину.

Например, если для нашей VM был выбран **Debian**, будем следовать инструкциям по установке с официального сайта Docker: **"[Docker: Install using the apt repository](https://docs.docker.com/engine/install/debian/#install-using-the-repository)"**.

Проверим ещё раз, что инструкция была выполнена корректно и Docker доступен:
> docker --version

Наша VM в случае проблем может быть перезагружена. В этом случае необходимо чтобы Docker загружался при старте системы, т.е. добавить Docker в автозагрузку. Для этого добавим докер в **[systemd](https://wiki.debian.org/systemd)**.

Добавление Docker'а в systemd описано в официальной инструкции: **"[Configure Docker to start on boot with systemd](https://docs.docker.com/engine/install/linux-postinstall/#configure-docker-to-start-on-boot-with-systemd)"**:
```
sudo systemctl enable docker.service
sudo systemctl enable containerd.service
```

----

## [↑](#home) <a id="swap"></a> Linux SWAP settings
Т.к. у нашей VM очень мало памяти, нам понадобится настроить SWAP.

Откроем **[Google Cloud Console](https://console.cloud.google.com/)** и для нашей VM нажмём **SSH** для подключения к VM.

Для начала проверим текущую доступную память (в мегабайтах, для удобства):
> free -m

Мы увидим два важных момента:
- из гигабайта памяти почти половина уже занята
- по умолчанию нет swap'а

Т.к. нам этой памяти для большинства задач не хватит, придётся добавить swap.\
Воспользуемся инструкцией: **"[Digital Ocean: How To Add Swap Space on Debian](https://www.digitalocean.com/community/tutorials/how-to-add-swap-space-on-debian-11)"**.

Единственное, выберем размер файла больше, чем в статье:
> sudo fallocate -l 4G /swapfile

Как и сказано в статье, выдадим права к данному файлу только для root:
> sudo chmod 600 /swapfile

Помечаем файл как "файл подкачки" (он же "swap space"):
> sudo mkswap /swapfile

И активируем (swap on):
> sudo swapon /swapfile

Добавим данный swap в качестве постоянного:
```
sudo cp /etc/fstab /etc/fstab.bak
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
cat /etc/fstab
```

Теперь в ``free -m`` мы увидим, что SWAP появился.

Стоит отметить, что данную операцию мы выполнили только потому, что размер памяти у нас очень ограничен. Например, использование SWAP для приложений, которые должны по возможности меньше работать с диском (например, Kafka) нежелателен, т.к. это приводит к тому, что вместо быстрой записи в память происходит медленная запись на диск.