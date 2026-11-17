# Аварийная загрузка Linux через консоль GRUB

> Инструкция применима к любому Linux-дистрибутиву, когда система не загружается из-за сбоя `systemd`, повреждения конфигов или критических сервисов. Рассматриваются оба варианта: корень в LVM и `/boot` отдельным разделом, и корень с `/boot` внутри одного тома.

---

## 1. Подготовка модулей GRUB

В консоли GRUB (`grub>` или `grub rescue>`) включите модули, необходимые для чтения разделов и файловых систем:

```bash
insmod lvm          # если корень или /boot в LVM
insmod xfs          # если используется XFS
insmod ext2         # покрывает ext2/ext3/ext4 (частый выбор для /boot)
insmod part_gpt     # для GPT-разметки
insmod part_msdos   # для MBR-разметки
insmod linux        # чтобы работала команда linux
```

Если не уверены в файловой системе, можно попробовать `insmod ext2`, `insmod xfs`, `insmod btrfs` по очереди — лишние модули не повредят.

---

## 2. Поиск разделов и ядер

### 2.1. Список устройств

```bash
ls
```

Устройства LVM отображаются как `(lvm/ГРУППА_ТОМОВ-ЛОГИЧЕСКИЙ_ТОМ)`, обычные разделы — как `(hd0,gpt1)`, `(hd0,msdos1)` и т.п.

### 2.2. Поиск раздела с ядрами

Переберите разделы и тома, пока не найдёте тот, где лежат `vmlinuz-*` и `initrd*`/`initramfs*`:

```bash
ls (hd0,gpt1)/
ls (hd0,gpt2)/
ls (hd0,msdos1)/
ls (lvm/vg0-lv_root)/
ls (lvm/vg0-lv_boot)/
```

Возможны два сценария:

- **`/boot` — отдельный раздел** (не в LVM): ядра лежат прямо в корне этого раздела, например `(hd0,gpt2)/vmlinuz-...`. Тогда `set root=(hd0,gpt2)` и пути к ядрам — `/vmlinuz-...`.
- **`/boot` — каталог внутри корня** (в LVM или на разделе): ядра лежат по пути `/boot/vmlinuz-...`. Тогда `set root=` указывает на корень системы, а пути — `/boot/vmlinuz-...`.

### 2.3. Указание корня для GRUB

```bash
set root=(hd0,gpt2)          # раздел с /boot
# или
set root=(lvm/vg0-lv_root)   # корень системы, если /boot внутри него
```

Проверьте, что файлы видны:

```bash
ls /
# или
ls /boot
```

> **Важно:** `set root=...` в GRUB — это раздел, откуда GRUB читает файлы ядра и initrd (пути в `linux` и `initrd` считаются от него). Параметр `root=...` в строке `linux` — это корень уже загружаемой системы для ядра. Это разные вещи, и они могут указывать на разные устройства.

---

## 3. Передача управления в Bash (в обход systemd)

Для автозаполнения длинных имён ядер нажимайте **Tab**. История команд — стрелки вверх/вниз.

### 3.1. Загрузка ядра

```bash
linux /vmlinuz-[ВЕРСИЯ] root=/dev/mapper/vg0-lv_root ro init=/bin/bash
```

или, если `/boot` внутри корня:

```bash
linux /boot/vmlinuz-[ВЕРСИЯ] root=/dev/mapper/vg0-lv_root ro init=/bin/bash
```

Параметры:

- `root=` — корневой раздел системы: `/dev/mapper/vg0-lv_root`, `/dev/vg0/lv_root`, `/dev/sda2`, UUID (`root=UUID=...`) и т.п.
- `ro` — монтирование только для чтения (безопаснее для ФС); запись включается позже командой `mount -o remount,rw /`.
- `rw` — сразу на запись (если уверены в целостности ФС).
- `init=/bin/bash` — ядро запускает bash вместо systemd.
- `selinux=0` — если RHEL/CentOS/Fedora и SELinux может блокировать вход.
- `enforcing=0` — альтернатива `selinux=0` (SELinux в режиме permissive).

### 3.2. Загрузка initrd

Имя должно **строго совпадать** с версией ядра и с тем, что вы увидели в `ls`:

```bash
initrd /initramfs-[ВЕРСИЯ].img
# или
initrd /initrd.img-[ВЕРСИЯ]
```

Типичные имена по дистрибутивам:

- Debian/Ubuntu: `initrd.img-XXXX`
- RHEL/CentOS/Fedora: `initramfs-XXXX.img`
- Arch: `initramfs-linux.img`

### 3.3. Старт

```bash
boot
```

---

## 4. Действия в аварийной консоли `#`

Вы окажетесь в `bash` с правами `root` без ввода пароля. `systemd` и его службы не запущены.

### 4.1. Монтирование файловой системы на запись

```bash
mount -o remount,rw /
```

### 4.2. Подключение остальных дисков

```bash
mount -a -v
```

`-v` показывает, что именно монтируется. Если `mount -a` падает или зависает (например, из-за ошибки в `/etc/fstab` — частой причины сбоя загрузки), монтируйте точки по одной вручную, начиная с `/`:

```bash
mount /dev/mapper/vg0-lv_var /var
mount /dev/mapper/vg0-lv_home /home
```

### 4.3. Исправление системы

- Сброс забытого пароля: `passwd root` (или `passwd пользователь`).
- Редактирование конфигов: `nano /etc/fstab`, `vi /etc/default/grub` и т.д.
- Проверка `fstab` перед правкой: `cat /etc/fstab`.
- Отключение сбойного сервиса вручную: `rm /etc/systemd/system/multi-user.target.wants/СБОЙНЫЙ_СЕРВИС.service`.
- Восстановление загрузчика:
  - Debian/Ubuntu: `update-grub`
  - RHEL/CentOS/Fedora: `grub2-mkconfig -o /boot/grub2/grub.cfg`
  - Установка в MBR/GPT: `grub-install /dev/sda` (Debian/Ubuntu) или `grub2-install /dev/sda` (RHEL/CentOS/Fedora).
- Переустановка ядра, если файлы повреждены:
  - Debian/Ubuntu: `apt install --reinstall linux-image-$(uname -r)`
  - RHEL/CentOS/Fedora: `dnf reinstall kernel`

### 4.4. Перезагрузка и выключение

Службы управления питанием не запущены, поэтому обычные `reboot` и `poweroff` не сработают. Используйте:

```bash
reboot -f
poweroff -f
```

Альтернатива через sysrq:

```bash
echo b > /proc/sysrq-trigger   # перезагрузка
echo o > /proc/sysrq-trigger   # выключение
```

---

## 5. Важные замечания по безопасности

- Любой, кто имеет **физический доступ** к машине и знает GRUB, может использовать этот метод для получения root-доступа.
- Меры защиты:
  - **Пароль на GRUB.** Настройка в `/etc/grub.d/40_custom`:
    ```
    set superusers="admin"
    password_pbkdf2 admin grub.pbkdf2.sha512....
    ```
    Хеш получается командой `grub-mkpasswd-pbkdf2` (Debian/Ubuntu) или `grub2-mkpasswd-pbkdf2` (RHEL/CentOS/Fedora). После правки — `update-grub` / `grub2-mkconfig`.
  - **Шифрование корневого раздела LUKS.** При аварийной загрузке зашифрованного корня перед `linux`/`initrd` нужны модули и разблокировка:
    ```bash
    insmod cryptodisk
    insmod luks
    cryptomount -a
    ```
    После этого появится устройство `(crypto0)`, и пути к ядрам/initrd указываются относительно него (если `/boot` тоже зашифрован) либо `/boot` остаётся незашифрованным отдельным разделом.
  - **Пароль BIOS/UEFI** и **Secure Boot** — затрудняют подмену загрузчика.
- Init-shell (`init=/bin/bash`) минует все проверки PAM и паролей — это и есть суть метода, поэтому физическая безопасность машины критична.