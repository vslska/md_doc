#### Создание директории:

```
- name
  file:
   path: /home/vm2/newfolder
   state: directory
   mode: 0755
```

#### Создание файла

```
- name:
  copy:
    dest: /home/vm2/newfolder/file1.txt
    content: |
       Text line 1 file1
       Text line 2 file2
```

Include: 
