

> *«Terraform — не инструмент для облаков.  
> Это — договор с хаосом:  
> “Я опишу желаемое состояние.  
> Ты — приведи реальность к нему.  
> Без шума. Без боли. Без императива.”»*  
> — technology-dragon, 2025

---

###  1. Основные понятия (по-моему)

| Термин                          | Что это                                          | Аналогия                                                 |
| ------------------------------- | ------------------------------------------------ | -------------------------------------------------------- |
| **Provider**                    | Драйвер для облака (AWS, YC, GCP)                | *«Переводчик: я говорю на HCL — облако понимает на API»* |
| **Resource**                    | Объект в облаке (ВМ, bucket, сеть)               | *«Строка в партитуре: “Здесь — nginx. Здесь — база”»*    |
| **State (`terraform.tfstate`)** | Снимок текущего состояния инфраструктуры         | *«Память системы: “Я помню, что построил”»*              |
| **Plan / Apply**                | `plan` — показать изменения, `apply` — применить | *«Сначала — репетиция. Потом — спектакль»*               |
| **Module**                      | Повторно используемый блок (как Role в Ansible)  | *«Готовый модуль урока: “Docker в YC за 5 минут”»*       |

---

### 2.  Минимальный рабочий цикл

```bash
# 1. Инициализация (скачает provider)
terraform init

# 2. Просмотр изменений — ДЕЛАЙ ВСЕГДА
terraform plan

# 3. Применить (с подтверждением)
terraform apply

# 4. Уничтожить — осторожно!
terraform destroy
```

В CI/CD можно добавить:
```bash
terraform plan -out=tfplan && terraform apply tfplan
```

### 3. Структура проекта (минимализм)
```bash
yc-vm/
├── main.tf          # ресурсы
├── variables.tf     # входные параметры
├── outputs.tf       # что вернуть (IP, ID…)
├── terraform.tfvars # значения переменных (не в Git!)
└── .gitignore

```
#### `main.tf` — пример: ВМ в Yandex Cloud
```hcl
provider "yandex" {
  token     = var.yc_token
  cloud_id  = var.yc_cloud_id
  folder_id = var.yc_folder_id
  zone      = "ru-central1-a"
}

resource "yandex_compute_instance" "vm" {
  name = "dragon-vm"

  resources {
    cores  = 2
    memory = 2
  }

  boot_disk {
    initialize_params {
      image_id = "fd8iv5v47ss5p3s1vkgc"  # Ubuntu 22.04 LTS
      size     = 20
    }
  }

  network_interface {
    subnet_id = yandex_vpc_subnet.subnet.id
    nat       = true
  }

  metadata = {
    ssh-keys = "ubuntu:${file("~/.ssh/id_ed25519.pub")}"
  }
}

# Сеть и подсеть — вынесены отдельно (чистота!)
resource "yandex_vpc_network" "net" {
  name = "dragon-net"
}

resource "yandex_vpc_subnet" "subnet" {
  name       = "dragon-subnet"
  zone       = "ru-central1-a"
  network_id = yandex_vpc_network.net.id
  v4_cidr_blocks = ["10.0.1.0/24"]
}
```

### 4. Безопасность и стиль
| Совет                             | Почему важно!                                                                  |
| --------------------------------- | ------------------------------------------------------------------------------ |
| ❌ Не хранить `token` в коде       | → Используй `variables.tf` + `terraform.tfvars` (в `.gitignore`)               |
| ✅ Использовать `sensitive = true` | Для паролей, ключей: `variable "yc_token" { type = string; sensitive = true }` |
| ✅ Внешний state — только в проде  | Локально — `local`, в команде — `yc s3 bucket` или `terraform cloud`           |
| ✅ `terraform fmt` → всегда        | Как `black` для Python — красота через единообразие                            |
	Изспользовать `locals` для  производных  значений:
```hcl
locals {
  instance_name = "dragon-${var.env}-${var.app}"
  tags = {
    owner = "technology-dragon"
    env   = var.env
  }
}
```

- Не бояться `terraform console` — там можно проверить выражения _до_ `apply`.
- Не делать `apply` без `plan` → это как `rm -rf /` без `ls`.
- Использовать`terraform graph | dot -Tpng > graph.png` — чтобы _увидеть_ зависимости.
- Писать `outputs.tf` — даже если сейчас не нужно. Через месяц ты _благодарен_ себе.

