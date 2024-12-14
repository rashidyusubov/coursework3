# Гайд по 3 части курсовой работы
### Необходимые инструменты для выполнения курсовой работы:

1. Multi Image Kitchen (MIK) (СДО)
2. Apktool: https://apktool.org/docs/install
3. Signapk (СДО)
4. Amlogic USB Burning Tool (СДО)

### 1) Скачать образ с СДО, соответствующий вашей версии операционной системы: Android 9, Android 10, Android 11 и Android 12, вне зависимости от ревизии.

Исходные образы доступны по ссылке: https://disk.yandex.ru/d/-nC5ApvTkqHvbA

Альтернативные ссылки на скачивание образов, соответствующих выбранному варианту: https://cloud.mirea.ru/index.php/s/HitLmw7DdQbdK8w

### 2) Разборка образа с помощью утилиты Multi Image Kitchen (MIK).
Как пользоваться:
1. Разместите папку программы в любое место, ближе к корню диска, путь не должен содержать кириллицу и пробелы.
2. Запустите программу, перейдите в пункт меню: Вид и нажмите на пункт: Создать ярлык на столе, появится ярлык на Рабочем столе.

На значок программы или в её окно можно переносить образ(ы) прошивки (*.img), файл(ы) обновления (*.zip), образ(ы) раздела(ов) (*.img, *.PARTITION, *.fex, *.new.dat, *.new.dat.br) или папку с распакованной прошивкой, папку с распакованным обновлением, папку с распакованным образом раздела.

**Образы и папки должны располагаться ближе к корню раздела диска, в пути не должно быть пробелов и кириллицы**

![image_2024-12-09_22-56-10](https://github.com/user-attachments/assets/2888e9f2-4096-4e82-ab67-ae5df20f44b0)

![image_2024-12-09_22-55-46](https://github.com/user-attachments/assets/38e6a7fc-4687-472f-93b4-cd0f2b5ff3d3)

![image_2024-12-09_22-58-48](https://github.com/user-attachments/assets/c0f5353b-ad77-47dd-ba26-fab03de507b0)

### 3) Замена в образе анимации загрузки на свою
Необходимо включить проверку фоновых рисунков при загрузке устройства. Данная настройка активируется с помощью значения true переменной ```config_checkWallpaperAtBoot```, определенной внутри файла ```framework-res.apk```. Данный файл находится в папке /system/framework (для MIK ```/system/system/system/framework```, для adb pull - /system/framework) и для его модификации необходимо декомпилировать приложение с помощью apktool. После декомпиляции в файле ```res/values/bools.xml``` будет требуемая настройка. Необходимо установить значение в true. После этого пересобрать apk, подписать и заменить на устройстве.

![image_2024-12-09_22-55-46](https://github.com/user-attachments/assets/57868d52-b56f-47b5-92e1-57c703b360f7)

Анимация загрузки представлена в виде файла ```bootanimation.zip```, размещенного в ``` /system/system/system/media/bootanimation.zip ```

Содержимое zip архива представляет собой файл ```desc.txt``` с описанием поведения и папки с изображениями. Рекомендуется использовать изображения в формате png, размером до 500 КБ каждый.

Внимание! При запаковке файлов обратно в bootanimation.zip необходимо отключить сжатие в архиваторе, иначе анимация может работать некорректно. Делается это путем выбора настройки «Store» в архиваторе

1. Заменить bootanimation.zip на свою анимацию 
Пример desc.txt
```
1920 1080 30
c 1 0 android
```
### 4) Добавить 2-3 приложения через папку preinstallApp в разделе system и активировать автоматическую установку приложений из этой папки при запуске системы.

1. Добавить в конец файла installd.rc ```/system/system/system/etc/init/installd.rc``` следующие строки:
```
service preinstallApp /system/bin/sh /bin/preinstallApp.sh
    class main
    user root
    group root
    disabled
    oneshot
    seclabel u:r:shell:s0

on property:sys.boot_completed=1
    start preinstallApp
```

![image_2024-12-09_23-44-17](https://github.com/user-attachments/assets/77b7d063-8480-44f7-813e-15fb70203f68)

2. Заменить содержимое скрипта preinstall.sh ```/system/system/system/bin``` на следующий код:

```
#!/system/bin/sh

MARK=/data/local/thirdpart_apks_installed
PKGS=/system/preinstallApp
LOGTEXT=/data/local/log.txt

if [ ! -e $MARK ]; then
    touch $LOGTEXT
    echo "booting the first time, so pre-install some APKs." >> $LOGTEXT

    APKLIST=$PKGS/*.apk
    for INFILES in $APKLIST
    do
        echo $INFILES >> $LOGTEXT
        /system/bin/pm install -r $INFILES >> $LOGTEXT
    done
    echo "OK, installation complete." >> $LOGTEXT
    touch $MARK
fi
```
3. Создать файл preinstallApp.sh в ```/system/system/system/bin``` и вписать в этот файл следующий код:

```
#!/system/bin/sh

MARK=/data/local/thirdpart_apks_installed
PKGS=/system/preinstallApp
LOGTEXT=/data/local/log.txt

if [ ! -e $MARK ]; then
    touch $LOGTEXT
    echo "booting the first time, so pre-install some APKs." >> $LOGTEXT

    APKLIST=$PKGS/*.apk
    for INFILES in $APKLIST
    do
        echo $INFILES >> $LOGTEXT
        /system/bin/pm install -r $INFILES >> $LOGTEXT
    done
    echo "OK, installation complete." >> $LOGTEXT
    touch $MARK
fi
```

![image_2024-12-09_23-47-59](https://github.com/user-attachments/assets/d6e19581-c92c-48b0-80ea-1f70cd9fdfdf)

4. Cоздать папку **preinstallApp** в ```system/system/system/```
5. Переместить в папку **preinstallApp** ```system/system/system/preinstallApp``` 2-3 apk файлов приложений

![image_2024-12-09_23-49-58](https://github.com/user-attachments/assets/1d311420-aebb-411f-ab8c-e70747557f53)

**Обратите внимание, что объем свободного места в разделе system составляет ~ 300 МБ**

### 5) Установить созданную прошивку на устройство.

1. Открыть Amlogic USB Burning Tool
2. 3 раза нажать на кнопку function (которая по центру) на устройстве Khadas
3. Дожадться появления в спике устройства
4. Нажать на кнопку File и выбрать образ который хотите установить
5. Нажать на кнопку Start
6. После достижения 100% нажать на кнопку Stop
7. Закрыть программу

### 6) Сделать фото (Лучше снимать на видео) созданной анимации при загрузке устройства Khadas.

### 7) Добавить в отчет фотографию вывода списка установленных приложений с выделением тех, которые были установлены через папку preinstallApp, где видно дату и время с компьютера.

Вывода списка установленных сторонних приложений.

```
adb shell pm -l
```

![image](https://github.com/user-attachments/assets/adc4f7cd-c72f-4b60-afa7-80597e797f06)

![image](https://github.com/user-attachments/assets/02a1517e-499d-4493-9f15-2af6cb6040b6)

![image](https://github.com/user-attachments/assets/32266984-fa0d-4df2-bf76-a09e647f3bbb)

### 8) Добавить в отчет фотографию вывода скрипта установки где видно дату и время с компьютера.

Вывода списка установленных сторонних приложений.

```
adb shell vi /system/bin/preinstallApp.sh
```

![image](https://github.com/user-attachments/assets/59b6bc55-bd43-468f-867d-b0671f846af0)

### 9) Вывести информацию getprop и добавить листингом в отчёт (Примечание: все свойства, которые были получены при выполнении команды getprop).

```
getprop
```
