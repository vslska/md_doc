# Ansible: Обновление RHEL и Astra Linux (Async mode)

Этот плейбук предназначен для обновления систем в условиях нестабильного SSH-соединения (разрывы сессий ИБ). Использование `async` позволяет задаче выполняться на сервере даже после обрыва связи.

## Плейбук: update_systems.yml

```yaml
---
- name: OS Update (RHEL & Astra Linux)
  hosts: all
  become: true
  gather_facts: true

  tasks:
    # --- ПОДГОТОВКА (Особенно важно для Astra) ---
    - name: Fix broken dpkg (Astra Linux only)
      when: ansible_os_family == "Debian"
      ansible.builtin.command: dpkg --configure -a
      changed_when: false
      register: dpkg_fix

    # --- ОБНОВЛЕНИЕ ASTRA LINUX (ASYNC) ---
    - name: Run astra-update with async
      when: ansible_os_family == "Debian"
      ansible.builtin.command: astra-update -r -T -A
      async: 3600  # Ждем до 1 часа
      poll: 15     # Проверяем статус каждые 15 сек. Даже если SSH упадет, процесс на сервере выживет.
      register: astra_result

    # --- ОБНОВЛЕНИЕ RHEL (DNF) ---
    - name: Upgrade all packages (RHEL)
      when: ansible_os_family == "RedHat"
      ansible.builtin.dnf:
        name: "*"
        state: latest
      async: 3600
      poll: 15
      register: rhel_result

    # --- ПРОВЕРКА REBOOT (ASTRA/DEBIAN) ---
    - name: Check if reboot required (Astra)
      when: ansible_os_family == "Debian"
      ansible.builtin.stat:
        path: /var/run/reboot-required
      register: reboot_needed

    # --- ПЕРЕЗАГРУЗКА ---
    - name: Reboot if necessary
      ansible.builtin.reboot:
        msg: "System update applied, rebooting..."
        reboot_timeout: 600
      when: >
        (ansible_os_family == "Debian" and reboot_needed.stat.exists) or
        (ansible_os_family == "RedHat" and (rhel_result.changed or astra_result.changed))
```

## Разбор ключевых параметров для Obsidian:

- **`async: 3600`**: Ansible запускает задачу в фоновом режиме на стороне целевого сервера. Если твой SSH-сеанс закроется, процесс `astra-update` или `dnf` продолжит работать до завершения (лимит 1 час).
- **`poll: 15`**: Ansible будет переподключаться каждые 15 секунд, чтобы забрать вывод (stdout). Если связь пропадет, Ansible выдаст ошибку у тебя на экране, но **на сервере задача не прервется**.
- **`dpkg --configure -a`**: Гарантирует, что предыдущие неудачные попытки не заблокируют новый запуск.
- **`astra-update -r -T -A`**: Родная утилита Астры. Флаг `-A` (auto) критичен для автоматизации, чтобы скрипт не замер в ожидании нажатия клавиши.

## Методология "Дерзости":
Если хочешь запустить и сразу закрыть терминал (**Fire and Forget**), поменяй `poll: 15` на `poll: 0`. Тогда Ansible просто "кинет" задачу в сервер и сразу перейдет к следующей, не дожидаясь ответа.
