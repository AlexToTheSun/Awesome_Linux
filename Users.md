Данной статьи будет достаточно для того, чтобы сформировать у начинающих администраторов базовое понимание и методики использования users в UNIX системах.


```bash
# Хранилища учетных записей
/etc/passwd
/etc/shadow
/etc/group
/etc/gshadow
man [passwd|shadow|group|gshadow]

# Информационные  команды
id user_name
cat /etc/passwd | grep user_name # Информацией о пользователях
tail -n 1 /etc/passwd # последняя строчка с добевленным пользов.
users # список активных пользователей
group user # список групп конкретного пользователя
ls -la /home/user # Вывести содержимое директорий с отображением прав на файлы

# Авторизация под пользователем
# По ssh
ssh -p 22 -id /path/on/your/compute/privat_key user_name@xx.xx.xx.xx
ssh --help

# Находять уже на сервере под одним из пользователей, в сессии терминала
# Разница в su -l и sudo -i -u в том что sodo явно указывает что кроме switch user еще выдает пользователю права root
su -
su - user_name
su -l user_name
sudo -i
sudo -i -u user_name
```
# Users, зачем нужны?
Есть два главных юзкейса в исользовании users.
1. Для адинистраторов, с правами `sudo`
2. Для остальных пользователей системы, с ограниченными правами.

Разберем подробнее каждый вариант.

##  Использование учетных записей для адинистраторов, с правами `sudo`.
С точки зрения безопасности и управляемости системой, есть ряд преимуществ использовать пользователей с правами администратора через `sudo`, вместо прямого входа под суперпользователем `root`:
1. **Отслеживать действия на сервере.**

   Ссылки по rsyslog:
- [Логирование rsyslog](https://habr.com/ru/articles/321262/)
- [rsyslog github](https://github.com/rsyslog/rsyslog)
- [Настройка rsyslog](https://blog.sedicomm.com/2018/11/11/kak-nastroit-tsentralnyj-server-vedeniya-logov-s-pomoshhyu-rsyslog-v-linux/), [Видео](https://www.youtube.com/watch?v=Os-rBk1-ogU)
- [Просмотр логов](https://profitserver.ru/knowledge-base/linux-logs/)
- [Ссылка на проблему упорядочивания логов](https://github.com/rsyslog/rsyslog/issues/1400)
   
   Каждая команда sudo регистрируется в логах и легко сделать аудит. Простейший запуск логирования rsyslog, без доп настройки приведен ниже. Команды с запросом прав sudo будут сохраняться в файл `/var/log/auth.log`:
```bash
sudo apt update
sudo apt install rsyslog
sudo systemctl enable rsyslog
sudo systemctl start rsyslog
sudo systemctl status rsyslog
```
   Будут созадны файлы:
```bash
#  Файлы конфигурации:
/etc/rsyslog.conf # Основной файл конфигурации для rsyslog. В этом файле определяются правила, куда и какие логи записывать.
/etc/rsyslog.d/ # Каталог для дополнительных конфигурационных файлов. Сюда можно добавлять файлы для специфичной конфигурации логирования.

# Каталоги и файлы журналов:
/var/log/syslog # Основной системный журнал, куда записываются сообщения о различных системных событиях.
/var/log/auth.log # Журнал, в который записываются события, связанные с аутентификацией, такие как использование sudo.
/var/log/kern.log # Журнал сообщений ядра.
/var/log/daemon.log # Журнал для сообщений служб (демонов).
```


2. **Минимизировать риски своего поведения**

   Из за того что работа под пользователем sudo, вместо root, требует явного указания права sudo в административных командах, прийдется более сознательно подходить к своим действиям 
3. **Гибко настроить права доступа**

- [Youtube Д.Кетов - пользовательские учетные записи](https://www.youtube.com/watch?v=xHCxaF2qbyQ)
- [Youtube Д.Кетов - подробнее про деленирование прав доступа](https://www.youtube.com/watch?v=NePIcJ1cZ-w)

   Можно настроить файл конфигурации `/etc/sudoers` так,
чтобы давать разным пользователям/группам повышенные привилегии на исполнение разных команд. При редактировании файла `etc/sudoers` надо использовать оболочку `visudo` для безопасного редактирования (проверяет целостность синтаксиса). Редактор для `visudo` берется системой автоматически и можно изменить задав переменную `${editor}`.
```bash
sudo visudo # откроется файл /etc/sudoers в режиме безопасного редактирования

# Далее варианты редактирования.
# Пользователь может выполнять все команды без ввода пароля:
# Синтаксис: <user / %group>   <на каком хосте>=(<От чьего лица>)   <на какие программы>
     user_name ALL=(ALL) NOPASSWD:ALL

# Пользователь может выполнять только команды apt systemctl:
     user_name ALL=(ALL) /usr/bin/apt, /usr/bin/systemctl
# При таком четком указании доступных комманд от лица супер пользователя 
№ мы не сможем войти в интерактивный режим командного интерпритатора через sudo -i ,  так как нужен доступ к /bin/bash от лица root 

# Группа пользователей admin может выполнять команды без пароля:
     %admin ALL=(ALL) NOPASSWD:ALL
     %sudo ALL=(ALL) ALL
     
# Пользователь user_name может перезагружать систему без запроса пароля на всех узлах от любого лица:
     user_name ALL=(ALL)  NOPASSWD:/sbin/reboot,/sbin/shutdown
```
Так же, чтобы управлять правами для отдельных пользователей или групп, вместо редактирования главного файла `/etc/sudoers`, можнно создавать отдельные файлы в каталоге `/etc/sudoers.d`. В этих файлах синтаксис такой же как и в главном файле, разница лишь в том что главный файл остается неизменным. Пример:
```bash
sudo visudo -f /etc/sudoers.d/file_name # записываем нужные права и сохраняем файл
sudo visudo -c # проверяем корректность файлов
```

4. **Бытрое реагирование, если user_sudo скомрометирован**

   В случае если пользователь группы `sudo` скомпрометирован - можно быстро отозвать права администратора удалив его из групы `sudo`.
```bash
deluser <user> <group>
gpasswd -d <user> <group>
```
##  Использование учетных записей для остальных пользователей.
Можно гибко ограничивать права доступа, выбирать рабочую оболочку, способ авторизации в систему и т.д. 


# Создание и редактирование
Необходимые команды для создания и редактирования пользвателей
```bash
useradd
groupadd
usermod  # usermod -aG sudo user_name
userdel
passwd - модиф. пароля , -l заблокировать , -u разблокировать
chpasswd - модиф. пароля без прав суперпользователя
chage - изм. срока действия пароля
users - список активных пользователей
chsh - изм. оболочки пользователя
chfn - изм. инф. о пользователе
deluser user group - удаление пользов. из группы
gpasswd -d user group- удаление пользов. из группы
```

Пример создания пользователя:
```bash
useradd --help
# -d, --home-dir
# -m, --create-home — создаёт домашнюю директорию в /home
# -M, --no-create-home — не создаёт домашнюю директорию для пользователя.
# -u, --uid — требуется для того, чтобы вручную задать uid пользователя
# -r, -system
# -h, -help
# -G, --groups
# -p, --password
# -s, --shell — если не указать то оболочка по умолчанию /bin/sh. все оболочки: cat /etc/shells

sudo useradd -m -p 123123 -G sudo -s /bin/bash user_name
```
Пароль лучше здавать в интерактивном режиме либо файлом, чтобы в истории не сохрнялись команды с паролями от созданных пользователей:
```bash
sudo useradd -m -G sudo -s /bin/bash user_name && sudo passwd user_name
```
### Домашний каталог пользователя
Если при создании пользователя не применили флаг `-m` то у ползователя отсутствует домашняя дректория и им не получится полноценно пользоваться. Двайте создадим её:
```bash
sudo mkdir /home/username
```
Скопируем фалы скелета. Обычно новые домашние каталоги инициализируются с набором стандартных файлов, хранящихся в `/etc/skel`
```bash
sudo cp -r /etc/skel/. /home/username
```
Установим пользователя и группу владельцем нового домашнего каталога:
```bash
sudo chown -R username:username /home/username
```
В файле `/etc/passwd` надо записать созданный каталог как домашний каталог пользователя.
```bash
# Либо вручную
nano /etc/passwd

# Либо командой usermod
sudo usermod -d /home/username username
```
# Настройка безопасности
Ссылки:
- [Статья linode: Use 2FA (Two-Factor Authentication) with SSH](https://www.linode.com/docs/guides/secure-ssh-access-with-2fa/)
- [Статья Vultr: How to Use Two-Factor Authentication with Sudo and SSH on Linux with Google Authenticator](https://docs.vultr.com/how-to-use-two-factor-authentication-with-sudo-and-ssh-on-linux-with-google-authenticator)
- [Статья DigitalOcean: How To Configure SSH Key-Based Authentication on a Linux Server](https://www.digitalocean.com/community/tutorials/how-to-configure-ssh-key-based-authentication-on-a-linux-server)
- [Статья базовой безопасности от techdocs!](https://techdocs.akamai.com/cloud-computing/docs/set-up-and-secure-a-compute-instance)
- [Ссылка для скачивания puttygen](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)
- [About no /etc/nologin](https://unix.stackexchange.com/questions/17906/can-i-allow-a-non-root-user-to-log-in-when-etc-nologin-exists)
- [Замена SSH порта](https://www.hostinger.com/tutorials/how-to-change-ssh-port-vps?utm_campaign=Generic-Tutorials-DSA|NT:Se|LO:NL&utm_medium=ppc&gad_source=1&gclid=CjwKCAiAmfq6BhAsEiwAX1jsZ4QYcxOlVtKoKhMhtiGaiA0i_SDMV9or0y6mZQiYPU0ey8T0LKfZsBoCBo8QAvD_BwE)
 
Настройку проводите с дополнительно открытой сессией на случай если ssh будет настроен некорректно и произойдет отказ при авторизации в новой сессии.

### Замена SSH порта
```
sudo nano /etc/ssh/sshd_config
```
Заскомментировать строчку `#Port 22` и вместо 22 поставить своё число. 

Далее - рестарт ssh сервиса. Вот варианты рестарта:
```
sudo systemctl restart sshd
sudo systemctl restart ssh
sudo service ssh restart

sudo systemctl status ssh
```


### Этапы настройки 2FA для new_user:
1. Логин под new_user
2. Генерация 2ФА
3. Корректировка файлов `/etc/pam.d/sshd` и `/etc/ssh/sshd_config`
4. Рестрарт `sshd`

Заранее замечу, что в файле `/etc/ssh/sshd_config` имеется параматр `AuthenticationMethods` для глобальной настройки его используют вне блока Match User. Но если надо настроить исключение из глобальной настройки `AuthenticationMethods` для отдельных пользователей то применяют , как в примере ниже 
```bash
Match User new_user_1
    AuthenticationMethods publickey,keyboard-interactive
Match User new_user_2
    AuthenticationMethods publickey
```

## Пример
Для примера настроим вход user_1 c `паролем+2FA`, а user_2 c  `паролем+2FA+ssh_key`.

Создадим пользователей, и сделаем бэкап файлов конфигурации, для возможного восстановления в случае ошибки. 
```bash
sudo useradd -m -G sudo -s /bin/bash user_1 && sudo passwd user_1
sudo useradd -m -G sudo -s /bin/bash user_2 && sudo passwd user_2
sudo cp /etc/pam.d/sshd /etc/pam.d/sshd_backup
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config_backup
```
### Базовая настройка конфигурации ssh
Корректируем  `/etc/pam.d/sshd`. Это требуется для обоих пользователей. Вставим две строки вконец файла.
```bash
sudo tee -a /etc/pam.d/sshd > /dev/null <<'EOF'
auth    required      pam_unix.so     no_warn try_first_pass
auth    required      pam_google_authenticator.so nullok
EOF
# После успешной настройки пользователей мы должны убрать параметр `nullok`. Он нужен, чтобы разрешить вход только по паролю пользователям, которые еще не анстроили google 2fa.
```
Корректируем `/etc/ssh/sshd_config`
По дэфолту он выставлен правильно `yes`, проверим `sudoedit /etc/ssh/sshd_config`
```bash
KbdInteractiveAuthentication yes
# Либо устаревший вариант этого парамета:
ChallengeResponseAuthentication yes
```
### Настройка user_1 (passwd+2fa)
Сгенерируем google 2fa.
```bash
su - user_1
google-authenticator # сгенерируйте или вставьте уже исользуемый .google_authenticator от другого поьзователя
ls -la /home/user_1/.google_authenticator
# stdout
-r-------- 1 user_1 user_1 119 Oct  9 07:39 /home/user_1/.google_authenticator
```
Корректируем `/etc/ssh/sshd_config` для того чтобы определить авторизацию `user_1` - условие `keyboard-interactive` включает в себя как пароль так и google-2fa:
```bash
sudo tee -a /etc/ssh/sshd_config > /dev/null <<'EOF'
Match User user_1
    AuthenticationMethods keyboard-interactive
EOF
```
> Стоит упомянуть что перенаправление (к примеру, с использованием `>>`) происходит в контексте пользователя, который выполняет команду, а не в контексте `sudo`. Это означает, что хотя `sudo echo` выполняется с правами `root`, сам `>> /etc/ssh/sshd_config` выполняется с правами `user_1`, и не имеет прав для записи.
```bash
# Пример перенаправление с правами user_1
sudo echo 'Match User user_1
    AuthenticationMethods keyboard-interactive' >> /etc/ssh/sshd_config

sudo cat << 'EOF' >> /etc/ssh/sshd_config
Match User user_1
    AuthenticationMethods keyboard-interactive
EOF

# Пример с командой tee. Строки вставляются вконец файла
echo 'Match User user_1
    AuthenticationMethods keyboard-interactive' | sudo tee -a /etc/ssh/sshd_config > /dev/null
```
Рестарт сервиса ssh и проверим вход, если не работает решаем проблемы.
```bash
sudo systemctl restart sshd
```
### Настройка user_2 (ssh_key+passwd+2fa)
Сгенерируем 2fa
```bash
su - user_2
google-authenticator
ls -la /home/user_1/.google_authenticator
```
Cоздадим ssh-key для user_2

```bash
ssh-keygen
# Созданы файлы /home/user_2/.ssh/id_rsa , /home/user_2/.ssh/id_rsa.pub
# ~/.ssh/id_rsa.pub - public key. Остается на сервере.
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys

# ~/.ssh/id_rsa - private key. Закачайте на ПК.

```
Если вы хотите не создавать а скопировать из home дирктории другого пользователя:
```bash
mkdir .ssh
sudo cp  /home/user_2/.ssh/id_rsa.pub  /home/user_1/.ssh/id_rsa.pub
sudo cp  /home/user_2/.ssh/id_rsa  /home/user_1/.ssh/id_rsa
sudo cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys

# Команда ниже нужна для того чтобы перекинуть id_rsa.pub со стороннего сервера на настраиваемый
sudo cat ~/.ssh/id_rsa.pub | sudo ssh -p 22 -i /path/on/your/compute/privat_key username@remote_host "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
```

Используйте [puttykeygen](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) для конвертации id_rsa в файл id_rsa.ppk (надо выбрать RSA 2048) для авторизации через терминалы вроде putty. Либо используйте исходный файл id_rsa для входа через консоль сервера/ПК на unix.

Корректируем `/etc/ssh/sshd_config` для того чтобы определить авторизацию `user_2` - условие `publickey,keyboard-interactive` будет запрашивать ssh-key, passwd, google-2fa:
```bash
sudo tee -a /etc/ssh/sshd_config > /dev/null <<'EOF'
Match User user_2
    AuthenticationMethods publickey,keyboard-interactive
EOF
```
Рестарт сервиса sshd
```bash
sudo systemctl restart sshd
```
После того как мы проверили что настроили успешно всех пользователей убирем/закомментируем параметр `nullok` чтобы 2FA была обязательным шагом авторизации. С `nullok` 2FA требуется только от пользователей, настроивших этот этап авторизации.
```bash
sed -i '/auth    required      pam_google_authenticator.so/c\auth    required      pam_google_authenticator.so # nullok' /etc/pam.d/sshd
```


# Заметки
### Приглашение командного интерпритатора
Если оболочка командного интерпритатора /bin/shell,  то приглашение будет классическим
```bash
$ -символ классического приглашения пользователя
# -для суперпользователя 
```
Если выбрать для пользователя /bin/bash, то увидим более информативное приглашение
```bash
user_name@computer_name:~$
# ~ это символ условного имени домашнего каталога пользователя. Команда `echo ~` покажет домашнюю директорию пользователя
# computer_name - собственное имя компьютера
```
### Использование id_rsa
При генерации ssh_key на сервере, создается файл приватного ключа id_rsa который подходит для использования на UNIX системах, допусти чтобы залогинеться на сервер через ком. строку MacBook или ПК на Ubuntu (через встроенную командную строку).

Используйте [puttykeygen](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) для конвертации id_rsa в файл id_rsa.ppk (выбрать RSA 2048) для авторизации через терминалы вроде putty.
