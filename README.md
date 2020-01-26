# **Инициализация системы. Systemd и SysV**
### **Написать service, который будет раз в 30 секунд мониторить лог на предмет наличия ключевого слова (файл лога и ключевое слово должны задаваться в /etc/sysconfig)**

Для начала создаём файл с конфигурацией для сервиса в директории /etc/sysconfig - из неё сервис будет брать необходимые переменные.

![network](https://github.com/vasiliev-an/lesson_5/blob/master/img/1.png)


Затем создаем /var/log/watchlog.log и пишем туда строки на своё усмотрение, плюс ключевое слово ‘ALERT’

![network](https://github.com/vasiliev-an/lesson_5/blob/master/img/2.png)


Создадим скрипт:

![network](https://github.com/vasiliev-an/lesson_5/blob/master/img/3.png)

Команда logger отправляет лог в системный журнал

Разрешаем выполнение скрипта:
```
chmod +x watchlog.sh 
```

Создадим юнит для сервиса:

![network](https://github.com/vasiliev-an/lesson_5/blob/master/img/4.png)


Создадим юнит для таймера:

![network](https://github.com/vasiliev-an/lesson_5/blob/master/img/5.png)

Убираем исполняемый бит у юнитов:

```
chmod 0644 /etc/systemd/system/watchlog.service
chmod 0644 /etc/systemd/system/watchlog.timer
```

Затем запускаем службу timer и смотрим логи:

![network](https://github.com/vasiliev-an/lesson_5/blob/master/img/6.png)



### **Из epel установить spawn-fcgi и переписать init-скрипт на unit-файл. Имя сервиса должно также называться.**


Устанавливаем spawn-fcgi и необходимые для него пакеты:

```
yum install epel-release -y && yum install spawn-fcgi php php-cli mod_fcgid httpd -y
```

![network](https://github.com/vasiliev-an/lesson_5/blob/master/img/7.png)

etc/rc.d/init.d/spawn-fcgi - cам Init скрипт, который будем переписывать

Но перед этим необходимо раскомментировать строки с переменными в /etc/sysconfig/spawn-fcgi

![network](https://github.com/vasiliev-an/lesson_5/blob/master/img/8.png)

А сам юнит файл будет примерно следующего вида:


![network](https://github.com/vasiliev-an/lesson_5/blob/master/img/9.png)


Убеждаемся что все успешно работает:

![network](https://github.com/vasiliev-an/lesson_5/blob/master/img/10.png)


### **Дополнить юнит-файл apache httpd возможностью запустить несколько инстансов сервера с разными конфигами**

Для запуска нескольких экземпляров сервиса будем использовать шаблон в конфигурации файла окружения /usr/lib/systemd/system/httpd.service.
Создадим httpd@first.service и httpd@second.service и добавим параметр %I в EnvironmentFile:

![network](https://github.com/vasiliev-an/lesson_5/blob/master/img/11.png)


![network](https://github.com/vasiliev-an/lesson_5/blob/master/img/12.png)

В самом файле окружения (которых будет два) задается опция для запуска веб-сервера с необходимым конфигурационным файлом:

![network](https://github.com/vasiliev-an/lesson_5/blob/master/img/13.png)

Соответственно в директории с конфигами httpd должны лежать два конфига, в нашем случае это будут first.conf и second.conf

Для удачного запуска, в конфигурационных файлах должны быть указаны уникальные  для  каждого  экземпляра  опции  Listen  и  PidFile.  Конфиги  можно скопировать и поправить только второй, в нем должны быть след опции:

```
PidFile /var/run/httpd-second.pid
Listen 8080
```

Добавляем два новых конфига:
```
cp httpd.conf first.conf
cp httpd.conf second.conf
```
Редактируем second.conf:

![network](https://github.com/vasiliev-an/lesson_5/blob/master/img/14.png)

Запускаем и проверяем, какие порты слушаются:

![network](https://github.com/vasiliev-an/lesson_5/blob/master/img/15.png)
