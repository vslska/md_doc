```
---
- name: install default Nginx Web Server
  hosts: servers
  become: yes

  vars:
    source_file: ./garb/index.html
    destin_file: /var/www/html/

  tasks:
  - name: Check and print linux vervion
    debug:
     var: ansible_os_family

  - block: # =====block for  debian=====

    - name: Install Nginx WebServer
      apt:
       name: nginx
       state: present

    - name: started nginx web server
      systemd:
       name: nginx
       state: started
       enabled: yes

    - name: Copy index.html to Servers
      copy: src={{ source_file }} dest={{ destin_file }} mode=0555
      notify: Restart Nginx Debian

    when: ansible_os_family == "Debian"


  handlers:
  - name: Restart Nginx
    systemd:
      name: nginx
      state: restarted
```


