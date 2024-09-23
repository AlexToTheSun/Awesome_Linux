# Описание
Создадим загрузочную флешку с различными образами ОС. Использовать будем [ventoy](https://ventoy.net/en/index.html)
# Ссылки
- [Ventoy](https://ventoy.net/en/index.html)
- [Windows 11 isp file](https://www.microsoft.com/ru-ru/software-download/windows11)
- [Ubuntu 22.04.5](https://releases.ubuntu.com/jammy/?_ga=2.64417327.1666800236.1725435531-688160996.1725105188&_gl=1*1ncjt6f*_gcl_au*NDM1NzUzMTY4LjE3MjUxMDUxOTM)
- [Ubuntu 24.04.1](https://releases.ubuntu.com/noble/)
# Процесс
Ventoy упрощает процесс создания закрузочной флешки:
- Утанавливаем Ventoy на usb flash drive.
- Скачиваем ISO/WIM/IMG/VHD(x)/EFI (в нашем случае iso) файлы необходимых нам систем. Проверяем их [на совместимостить с ventoy](https://ventoy.net/en/isolist.html).
- Перекидываем iso на флешку.

Теперь по порядку.
## Установка Ventoy на флешку (используем пк с Windows).
Установить ventoy надо только на флешку. Мы установим используя windows пк. 

Переходим на сайт https://ventoy.net/en/download.html и скачиваем архив `ventoy-1.0.99-windows.zip`. 

После скачивания проверим совпадает ли SHA-256 скаченного файла/архива с [указанным на сайте](https://ventoy.net/en/download.html)
```
File                      | SHA-256                                                          | Released   | Size
ventoy-1.0.99-windows.zip | 578993fcf8d57252bc34536c01dc853a374e60def68f60b2826c3de0826ea00c | 2024-06-08 | 15 MB
```
Для этого откроем PowerShell и вычислим хэш-значение для скаченного ISO-файла с помощью командлета Get-FileHash . Пример:
```
Get-FileHash D:\ventoy-1.0.99-windows.zip
```
![Снимок111](https://github.com/user-attachments/assets/a5b32c4e-c666-4418-9d46-f953c40fc4eb)

Подключим флешку к пк. Запустим `Ventoy2Disk.exe` и выберим нужную нам флешку для установки. 

## Скачиваем ISO образы систем.
Мы для примера скачали [Windows 11](https://www.microsoft.com/ru-ru/software-download/windows11), [Ubuntu 22.04.5 desktop/server](https://releases.ubuntu.com/jammy), [Ubuntu 24.04.1 desktop/server](https://releases.ubuntu.com/noble) iso файлы.

После скачивания обязательно сверяем SHA-256 каждого файла с хэшем, указанным разработчиками. Если все верно - просто скидываем файлы на нашу флешку, на которой мы установили Ventoy. Для удобства на флешке можно создавать директории.

Готово, можно пользоваться.
