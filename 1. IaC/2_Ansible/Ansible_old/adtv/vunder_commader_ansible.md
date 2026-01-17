ad_hoc коммандос

Вывод  запроса  на  ввод пароля.
```
--ask-become-pass 
```

Проверить сетевую
```
ansible all -m ping
```

ansible servers -m shell -a "ls -la /var"  # Выполнить комманду оболочки на managed (В данном случае у группы servers выполняется комманда ls -ls /var ) # вывод можно перенаправить >>

 `present` — установить пакет, если его нет. 
 `latest` — установить последнюю версию пакета. 
 `absent` — удалить пакет. 
 `stress` - 
 

ansible servers -m copy -a "src=/home/user  dest=/home" -b  # src = path до объекта dest = # path где будет размещен объект 

ansible servers -m file -a "path=/home/filename state=absent" -b # удаляем  на managed 

ansible servers -m get_url -a "url= dest=/" -b  #  скачивание данных с url на managed servers

uri - ansible all -m uri -a "url=http://bbbb.net return_content=yes" # выводит данные с ulr, помимо return_content=yes # выводит данные контента

ansible servers -m apt -a "name= *name package* state=present" -b  # Установить пакет







