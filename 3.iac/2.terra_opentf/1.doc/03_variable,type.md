Каждая переменная должна быть объявлена с помощью специального блока `variable`:

```hcl
variable "image_id" {
  type = string
}

variable "security_group_ids" {
  description = "(Optional) - List of security group IDs."
  type        = list(string)
  default     = []
}
```

После ключевого слова `variable` идёт уникальное в рамках манифеста название переменной. Это имя используется для указания значения переменной и обращения к ней в коде.

> [!Важно] Важно
> Нельзя  назначить переменной зарезервированные имена: **source, version, providers, count, for_each, lifecycle, depends_on, locals**.

### Аргументы  для объявления переменных
##### **Аргумент `description`**
Служит для описания переменной в документации.

Описание `description` должно кратко объяснять цель переменной и ожидаемый тип значения. Эта строка может быть включена в документацию о модуле, поэтому она должна быть написана с точки зрения пользователя модуля, а не его автора. Также можно включить информацию о том, обязательно ли указывать переменную.

Пример написания:
```hcl
variable "security_groups_ids" {
  description = "(Optional) - List of security group IDs."
  type        = list(string)
  default     = []
}
```

Из примера видно, что переменная необязательная, от пользователя ожидается список идентификаторов Security Group.

#### **Аргумент `type`**

Определяет, какой тип данных принимается для переменной. Например, **string** для строкового значения или **number** — для числового.

Пример написания:

```hcl
variable "pg_version" {
  description = "PostgreSQL version"
  type        = number
  default     = 15
}
```

Тип данных переменной — **string**.

#### **Аргумент `default`**

Значение переменной по умолчанию, которое делает её необязательной и будет использовано, если не дать значение переменной.

Пример написания:
```hcl
variable "pg_version" {
  description = "PostgreSQL version"
  type        = string
  default     = "15"
}
```

Переменная получит значение **"15"**, если пользователь не задаст её значение.
#### **Аргумент `validation`**

Блок для определения правил проверки, обычно в дополнение к ограничениям типа данных.

Пример написания:
```hcl
variable "pg_version" {
  description = "PostgreSQL version"
  type        = string
  default     = "15"
  validation {
    condition     = contains(["12", "12-1c", "13", "13-1c", "14", "14-1c", "15", "15-1c", "16"], var.pg_version)
    error_message = "Allowed PostgreSQL versions are 12, 12-1c, 13, 13-1c, 14, 14-1c, 15, 15-1c, 16."
  }
}
```
Разрешены только определённые значения переменной **pg_version**, соответствующие доступным версиям PostgreSQL. При использовании иного значения будет сообщение об ошибке, указанное в аргументе `error_message`.

#### **Аргумент `sensitive`**

Ограничивает вывод интерфейса Terraform при использовании переменной в конфигурации. Объявление переменной с аргументом `sensitive = true` заменит её значение в выводе команд `terraform plan` и `terraform apply` на `(sensitive value)`.
```hcl
variable "user_information" {
  type = object({
    name    = string
    address = string
  })
  sensitive = true
}

resource "some_resource" "a" {
  name    = var.user_information.name
  address = var.user_information.address
}

Terraform will perform the following actions:

  # some_resource.a will be created
  + resource "some_resource" "a" {
      + name    = (sensitive value)
      + address = (sensitive value)
    }

Plan: 1 to add, 0 to change, 0 to destroy.
```

#### **Аргумент `nullable`**

Контролирует, может ли переменной быть присвоено значение **null**. По умолчанию аргументу присвоено значение **true**.

Пример написания:
```hcl
variable "example" {
  type     = string
  nullable = false
}
```
У переменной не задано значение аргумента `default` и указано `nullable = false`. Таким образом, это обязательная переменная, которой необходимо указать некоторое значение, отличное от **null**.

Аргумент `type` в блоке переменных позволяет ограничить тип значения переменной. Если ограничение типа не установлено, то принимается значение любого типа — `any`.

> [!NOTE] Важно
> Ограничения типа данных (type constraints) необязательны, но мы рекомендуем указывать их. Они служат полезными напоминаниями для пользователей модуля и возвращают сообщение об ошибке, если используется неправильный тип данных.

Ограничения типа данных создаются из ключевых слов типа и конструкторов типов, позволяя комбинировать различные типы данных для описания сложных структур.

Типы данных можно разделить на primitive (простые) и complex (сложные):
![[Pasted image 20250923104530.png]]
## Простые типы данных
**String** — последовательность символов Unicode, представляющая текст (например, hello).
**Number** — числовое значение, которое может представлять целые числа (например, 15) и дробные значения (например, 6,283185).
**Bool** — логическое значение (**true** либо **false**), которое может использоваться в условной логике
## Сложные типы данных
**Коллекции** — для группировки схожих значений.
**Структурные типы** — для группировки потенциально несхожих значений.

Сложный тип данных группирует несколько значений в одно и представлен конструкторами типов. У некоторых конструкторов есть укороченные ключевые версии.
#### **Коллекции, или Collection**
Коллекции позволяют группировать несколько значений **одного** типа данных как одно значение. У всех типов коллекций должен быть элементный тип, который предоставляется как аргумент их конструктору. Например, `list(strings)` означает список строк, `list(numbers)` — список числовых значений.

> [!NOTE] Важно
Все элементы коллекции должны иметь один и тот же тип данных.
> 

Коллекции делятся на 3 группы

`list (...)` последовательность значений, которая идентифицируется последовательными целыми числами, начиная с 0.
- `list(string)` — список строк, например `["ru-central1-a", "ru-central-1b"]`.
- `list(number)` — список числовых значений, например `[4, 8, 15, 16, 23, 42]`.
- `list` или `list(any)` — принимает любой тип элемента, но все элементы должны иметь один тип данных. Например, вы можете присвоить значение `["abc", "def"]`, но не можете `["abc", 123]`.

`map (...)` коллекция значений, в которой каждое идентифицируется названием.
- `map(string)` — коллекция **ключ-значение строк**, например `{owner = "DevOps", project = "infra"}`.
- `map(number)` — коллекция **ключ-значение чисел**, например `{from_port: 8090, to_port = 8099}`.
- `map` или `map(any)` — принимает любой тип элемента, но все элементы должны иметь один тип данных. Например, вы можете присвоить значение `{abc = "abc", def = "def"}`, но не можете `{abc = "abc", def = 123}`.

`set (...)` коллекция уникальных значений, у которой нет последовательности или идентификации.
- `set(string)` — коллекция **строк**, например `["apple", "cherry", "banana"]`.
- `set(number)` — коллекция **чисел**, например `[101, 202, 303]`.
- `set` или `set(any)` — принимает любой тип элемента, но у всех элементов должен быть один тип данных. Например, вы можете присвоить значение `["abc", "def"]`, но не можете `["abc", 123]`.

#### **Структурные типы данных, или Structural**
Структурный тип позволяет группировать несколько значений различных типов данных в одно. Для структурных типов требуется схема в качестве аргумента, чтобы указать, какие типы данных разрешены для каждого элемента.

Структурные типы данных делятся на два вида:

`Object (...)`
Коллекция именованных атрибутов, у каждого есть свой тип данных. Схема объекта задаётся в виде `{KEY1 = TYPE, KEY2 = TYPE, ...}` — пар **ключ-значение**, разделённых запятой. Значение переменой должно содержать все указанные ключи, и значение для каждого ключа должно соответствовать его указанному типу.
```hcl
type = object({
    cloud_name  = string
    name        = string
    description = string
    labels      = map(string)
  })
  
value = {
  cloud_name  = "Yandex Cloud"
  name        = "infra-folder"
  description = "Infrastracture"
  labels      = {
    environment = "production"
    owner       = "DevOps"
  }
}  
```

**Опциональные атрибуты типа данных `object(…)`**

Terraform возвращает ошибку, когда не получает значение всех атрибутов объекта. Это можно изменить, используя опциональные атрибуты.
Чтобы сделать атрибут объекта опциональным, используется конструкция `optional(<TYPE>, <DEFAULT>)`.

Она принимает два аргумента:
- `<TYPE>` — тип данных атрибута, обязательный аргумент.
- `<DEFAULT>` — значение по умолчанию, которое будет использоваться, если пользователь не передал атрибут или передал **null**. Если не указывать этот аргумент, будет использовано значение **null**.

В примере описана переменная для настроек PostgreSQL-кластера, все атрибуты объекта в ней опциональные и у них указано значение по умолчанию:
```hcl
variable "performance_diagnostics" {
  description = "(Optional) - PostgreSQL cluster performance diagnostics settings."
  type = object({
    enabled                      = optional(bool, null)
    sessions_sampling_interval   = optional(number, 60)
    statements_sampling_interval = optional(number, 600)
  })
  default = {}
}
```

`tuple (...)`
Последовательность элементов, которая идентифицируется целыми числами, начиная с 0, при этом у каждого элемента свой тип данных. Схема кортежа задаётся в виде `[TYPE, TYPE, ...]` — списка типов, разделённых запятой. Значение переменной должно иметь то же количество и тип значений, что определены в схеме.
```hcl
type = tuple(string, number, bool)

value = ["example", 42, true]
```


Обратиться к переменной внутри Terraform-модуля можно с помощью выражения `var.<NAME>`, где `<NAME>` — объявленное в блоке `variable` название переменной.

```hcl
variable "network_name" {
  description = "(Optional) - Name of the network."
  type        = string
}

resource "yandex_vpc_network" "this" {
  name = var.network_name
}
```

К значениям элементов переменных с типом данных `map` и `object(…)` можно обращаться через ключи:

```hcl
variable "labels" {
  description = "(Optional) - Set of label pairs to assing to the PostgreSQL cluster. Include the 'name' label."
  type        = map(string)
  default     = {
    name = "pgsql-cluster", 
    owner = "DevOps"
  }
}

variable "access_policy" {
  description = "(Optional) - Access policy from other services to the PostgreSQL cluster."
  type = object({
    data_lens     = optional(bool, null)
    web_sql       = optional(bool, null)
    serverless    = optional(bool, null)
   = optional(bool, null)
  })  data_transfer
  default = {}
}

# PostgreSQL cluster
resource "yandex_mdb_postgresql_cluster" "this" {
name = var.labels["name"]
...
  config {
    access {
        data_lens     = var.access_policy.data_lens
        web_sql       = var.access_policy.web_sql
        serverless    = var.access_policy.serverless
        data_transfer = var.access_policy.data_transfer
    }
  }
  ...
}  
```

- Переменные позволяют переиспользовать код Terraform.
- Переменные объявляются с помощью блока `variable` и используются с ключевым словом `var`.
- Для переменной можно указать тип данных: простые для работы с символами, числами и логическими значениями или сложные для группировки значений.
- 
Когда вы обращаетесь к переменным с типом **object**, можно использовать формат `var.<VAR_NAME>.<ATTR_NAME>`. Например, `var.instance_resources.cores` в случае объявленной переменной.

Когда вы обращаетесь к переменной с типом **map,** необходимо указывать ключ элемента. Для объявленной выше переменной элемент будет указан через ключ с индексом **[0]**. Для получения имени ключа используйте функцию `keys`.

Обратиться к переменной внутри Terraform-модуля можно с помощью выражения `var.<NAME>`, где `<NAME>` — объявленное в блоке `variable` название переменной.

```hcl
variable "network_name" {
  description = "(Optional) - Name of the network."
  type        = string
}

resource "yandex_vpc_network" "this" {
  name = var.network_name
}
```

К значениям элементов переменных с типом данных `map` и `object(…)` можно обращаться через ключи:

```hcl
variable "labels" {
  description = "(Optional) - Set of label pairs to assing to the PostgreSQL cluster. Include the 'name' label."
  type        = map(string)
  default     = {
    name = "pgsql-cluster", 
    owner = "DevOps"
  }
}

variable "access_policy" {
  description = "(Optional) - Access policy from other services to the PostgreSQL cluster."
  type = object({
    data_lens     = optional(bool, null)
    web_sql       = optional(bool, null)
    serverless    = optional(bool, null)
   = optional(bool, null)
  })  data_transfer
  default = {}
}

# PostgreSQL cluster
resource "yandex_mdb_postgresql_cluster" "this" {
name = var.labels["name"]
...
  config {
    access {
        data_lens     = var.access_policy.data_lens
        web_sql       = var.access_policy.web_sql
        serverless    = var.access_policy.serverless
        data_transfer = var.access_policy.data_transfer
    }
  }
  ...
}  
```

- Переменные позволяют переиспользовать код Terraform.
- Переменные объявляются с помощью блока `variable` и используются с ключевым словом `var`.
- Для переменной можно указать тип данных: простые для работы с символами, числами и логическими значениями или сложные для группировки значений.
- 
Когда вы обращаетесь к переменным с типом **object**, можно использовать формат `var.<VAR_NAME>.<ATTR_NAME>`. Например, `var.instance_resources.cores` в случае объявленной переменной.

Когда вы обращаетесь к переменной с типом **map,** необходимо указывать ключ элемента. Для объявленной выше переменной элемент будет указан через ключ с индексом **[0]**. Для получения имени ключа используйте функцию `keys`.