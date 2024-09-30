# Aliases. Настрой взаимодействие с CLI под себя
Aliases - псевдонимы комманд. На любой символ или слово можно настроить вызов нужной для нас комманды с нужными опциями.
# Как пользоваться
Заданные `alias` сохраняются только до закрытия терминала.
```bash
# Проверка заданных alias
alias

# Задать alias
alias alias_name='команда аргумент ...'
alias alias_name="/путь/к/исполняемому/файлу"

# Удалить alias
unalias alias_name
```
# Как пользоваться
Чтобы не задавать alias в каждой новой сессии, можно прописать их в файле `~/.bash_aliases` и сделать ссылку на него в файле `~/.bashrc`. Пример:
```bash
# При входе в терминал исполняется файл $HOME/.bash_profile поэтому добавим туда исполнение файла с записанными aliases:
cat <<'EOF' >> $HOME/.bash_profile
if [ -f $HOME/.bash_aliases ]; then
    source $HOME/.bash_aliases
fi
EOF

# На случай если вам надо восстановить файл .bashrc по умолчанию скопируйте его из резервной директории коммандой cp /etc/skel/.bashrc ~/
# 

# Зададим для примера пару aliases:
cat <<'EOF' >> $HOME/.bash_aliases
alias c='clear'
alias ls='ls --color=auto'
EOF

# Проверим работу заданных псевдонимов после перезагрузки или исполнения файла $HOME/.bashrc
source $HOME/.bash_profile
c
```
# Ресурсы 
1. https://askubuntu.com/questions/161249/bashrc-not-executed-when-opening-new-terminal
2. https://redos.red-soft.ru/base/redos-7_3/7_3-administation/7_3-console/7_3-alias/?nocache=1727692862370
3. https://dev.to/siddharthshyniben/create-bash-aliases-to-work-faster-l91
4. https://losst.pro/poleznye-alias-linux
5. `alias --help`
6. `man aliases`

