Данной статьи будет достаточно для того, чтобы сформировать у начинающих администраторов базовое понимание и методики использования users в UNIX системах.

Информационные  команды
```bash
id user_name
cat /etc/passwd | grep user_name # Информацией о пользователях
users # список активных пользователей
tail -n 1 /etc/passwd # последняя строчка с добевленным пользов.
ls -la /home/user # Вывести содержимое директорий с отображением прав на файлы
```
Авторизация под пользователем
```bash
# По ssh
ssh -p 22 root@xx.xx.xx.xx
ssh -h
# Находять уже в терминале под одним из пользователей
su -
sudo -i
su - user_name
```
# Users, зачем нужны?
Есть два главных юзкейса в исользовании users.
1. Для адинистраторов, с правами `sudo`
2. Для остальных пользователей системы, с ограниченными правами.

Разберем подробнее каждый вариант.

##  Использование учетных записей для адинистраторов, с правами `sudo`.
С точки зрения безопасности и управляемости системой, есть ряд преимуществ использовать пользователей с правами администратора через `sudo`, вместо прямого входа под суперпользователем `root`:
1. **Отслеживать действия на сервере.**
   
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
Ссылки по rsyslog:
- [Логирование rsyslog](https://habr.com/ru/articles/321262/)
- [rsyslog github](https://github.com/rsyslog/rsyslog)
- [Настройка rsyslog](https://blog.sedicomm.com/2018/11/11/kak-nastroit-tsentralnyj-server-vedeniya-logov-s-pomoshhyu-rsyslog-v-linux/), [Видео](https://www.youtube.com/watch?v=Os-rBk1-ogU)
- [Просмотр логов](https://profitserver.ru/knowledge-base/linux-logs/)
- [Ссылка на проблему упорядочивания логов](https://github.com/rsyslog/rsyslog/issues/1400)

2. **Минимизировать риски своего поведения**

   Из за того что работа под пользователем sudo, вместо root, требует явного указания права sudo в административных командах, прийдется более сознательно подходить к своим действиям 
3. **Гибко настроить права доступа**

   Можно настроить файл конфигурации `/etc/sudoers` так,
чтобы давать разным пользователям/группам повышенные привилегии на исполнение разных команд.
```bash
sudo visudo # откроется файл /etc/sudoers в режиме безопасного редактирования

# Далее варианты редактирования.
# Пользователь может выполнять все команды без ввода пароля:
     user_name ALL=(ALL) NOPASSWD:ALL

# Пользователь может выполнять только определенные команды:
     user_name ALL=(ALL) /usr/bin/apt, /usr/bin/systemctl

# Группа пользователей admin может выполнять команды без пароля:
     %admin ALL=(ALL) NOPASSWD:ALL
     
# Конкретный пользователь может перезагружать систему без запроса пароля:
     user_name ALL=(ALL)
NOPASSWD:/sbin/reboot,/sbin/shutdown
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
passwd - модиф. пароля
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
```
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
```
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
Настройку проводите с дополнительно открытой сессией на случай если ssh будет настроин некорректно и будет отказ при логине в новой сессии.

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

## Для примера настроим вход user_1 c паролем+2FA, а user_2 c ssh pubkwy + 2FA
Создадим пользователей, и начнем настройку с `user_1`
```bash
sudo useradd -m -G sudo -s /bin/bash user_1 && sudo passwd user_1
sudo useradd -m -G sudo -s /bin/bash user_2 && sudo passwd user_2

su - user_1
google-authenticator # сгенерируйте или вставьте уже исользуемый .google_authenticator от другого поьзователя
ls -la /home/user_1/.google_authenticator
# stdout
-r-------- 1 user_1 user_1 119 Oct  9 07:39 /home/user_1/.google_authenticator
```

#### edit  /etc/pam.d/sshd for user_1
Вставим две строки вконец файла `sshd`
```
cp /etc/pam.d/sshd /etc/pam.d/sshd_backup
cp /etc/ssh/sshd_config /etc/ssh/sshd_config_backup
echo 'auth    required      pam_unix.so     no_warn try_first_pass
auth    required      pam_google_authenticator.so nullok' >> /etc/pam.d/sshd
```
После успешной настройки всех пользователей убирем/закомментируем паратетр `nullok` чтобы 2FA была обязательным шагом авторизации. С `nullok` 2FA требуется только от пользователей, настроивших этот этап авторизации.
```
sed -i '/auth    required      pam_google_authenticator.so/c\auth    required      pam_google_authenticator.so # nullok' /etc/pam.d/sshd
```

#### edit /etc/ssh/sshd_config for user_1
Строка ниже актуальна как для входа по паролю так и по ssh key. По дэфолту он выставлен правильно `yes`
```
KbdInteractiveAuthentication yes
# Либо устаревший вариант этого парамета
ChallengeResponseAuthentication yes
```
Теперь надо определить порядок авторизации. Вставим в конец файла условия конкретно для user_1:
```
sudo tee -a /etc/ssh/sshd_config > /dev/null <<'EOF'
Match User user_1
    AuthenticationMethods keyboard-interactive
EOF
```
Стоит упомянуть что перенаправление (к примеру, с использованием `>>`) происходит в контексте пользователя, который выполняет команду, а не в контексте `sudo`. Это означает, что хотя `sudo echo` выполняется с правами `root`, сам `>> /etc/ssh/sshd_config` выполняется с правами `user_1`, и не имеет прав для записи.
```
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
Рестарт сервиса ssh и проверим вход, если не работает решаем проблемы и переходим к настройки user_2
```
sudo systemctl restart sshd
```
### Продолжение - настройка user_2
Сдесь мы создадим ssh-key конкретно для user_2 и зададим вход по ssh-key+2FA
```
su - user_2
google-authenticator
ls -la /home/user_1/.google_authenticator

ssh-keygen
# Файлы сохранены в /home/user_2/.ssh
# ~/.ssh/id_rsa - private key закачайте на ПК для входа по ssh.
# ~/.ssh/id_rsa.pub - public key. Остается на сервере.
```
ЕСЛИ НАДО конвертируйте ключ с помошью  [puttykeygen](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html). Подробне о ssh key login [тут](https://github.com/AlexToTheSun/Validator_Activity/blob/main/Mainnet-Guides/Minimum-server-protection.md#ssh-key-login) 
Выберите RSA 2048
```
sudo tee -a /etc/ssh/sshd_config > /dev/null <<'EOF'
Match User user_2
    AuthenticationMethods publickey,keyboard-interactive
EOF

sudo systemctl restart sshd
```
Ссылки:
1 https://www.linode.com/docs/guides/secure-ssh-access-with-2fa/
2 https://docs.vultr.com/how-to-use-two-factor-authentication-with-sudo-and-ssh-on-linux-with-google-authenticator




Далее:
-- почему иногда (shell) не пишется: приглашение в sell и в bash


