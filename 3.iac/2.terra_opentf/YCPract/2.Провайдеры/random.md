### Генерация случайных значений

Важные цели:

1. **Определение случайных значений для тестирования**
2.  **Генерация уникальных имён ресурсо**
3.  **Генерация секретов**
4.  **Создание недетерминированных конфигураций
5.  **Обход ограничений**

Представьте, что нужно создать бакет с именем **bucket**. Скорее всего, это имя уже используется, а оно [должно быть уникальным](https://cloud.yandex.com/ru/docs/storage/concepts/bucket#naming).
```
╷
│ Error: error creating Storage S3 Bucket: BucketAlreadyExists: The requested bucket name is not available. The bucket namespace is shared by all users of the system. Please select a different name and try again.
│       status code: 409, request id: 8499ce68263482a2, host id: 
│
│   with yandex_storage_bucket.this,
│   on main.tf line 65, in resource "yandex_storage_bucket" "this":
│   65: resource "yandex_storage_bucket" "this" {
│
```
Random это логический провайдер — он не обращается к внешнему сервису.

| **Наименование**                                                                                           | **Применение**                                              | **Содержание**                                                           | **Пример**                                                                                                                                                                                               |
| ---------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------- | ------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [random_id](https://registry.terraform.io/providers/hashicorp/random/latest/docs/resources/id)             | Для создания уникальных идентификаторов ресурсов.           | Случайные цифры и их hex-значение, трансформированное в base64.          | "b64" = "ciNq9AnC8c8"  <br>"b64_std" = "ciNq9AnC8c8="  <br>"b64_url" = "ciNq9AnC8c8"  <br>"byte_length" = 8  <br>"dec" = "8224534940876992975"  <br>"hex" = "72236af409c2f1cf"  <br>"id" = "ciNq9AnC8c8" |
| [random_bytes](https://registry.terraform.io/providers/hashicorp/random/latest/docs/resources/bytes)       | Для секретов и ключей.                                      | Случайные байты.                                                         | "0xfe3a7b1d8c29f0643a5e97136bdc7b9a"                                                                                                                                                                     |
| [random_integer](https://registry.terraform.io/providers/hashicorp/random/latest/docs/resources/integer)   | Для задания значения приоритета для listener.               | Случайное цифровое значение, определённое с помощью атрибутов min и max. | 12345                                                                                                                                                                                                    |
| [random_password](https://registry.terraform.io/providers/hashicorp/random/latest/docs/resources/password) | Для создания паролей.                                       | Случайная последовательность букв, цифр и специальных символов.          | "ib}oQ8:r[UbaTzS9"                                                                                                                                                                                       |
| [random_pet](https://registry.terraform.io/providers/hashicorp/random/latest/docs/resources/pet)           | Для создания уникальных идентификаторов ресурсов.           | Случайные имена животных.                                                | "curious-koala"                                                                                                                                                                                          |
| [random_shuffle](https://registry.terraform.io/providers/hashicorp/random/latest/docs/resources/shuffle)   | Для списка случайных зон доступности и подсетей.            | Случайная выборка списка строк из строк, заданных атрибутом input.       | ["ru-central1-b", "ru-central1-d"]                                                                                                                                                                       |
| [random_string](https://registry.terraform.io/providers/hashicorp/random/latest/docs/resources/string)     | Для случайных строковых значений.                           | Случайная последовательность букв, цифр и специальных символов.          | "K8s#2jP!L"                                                                                                                                                                                              |
| [random_uuid](https://registry.terraform.io/providers/hashicorp/random/latest/docs/resources/uuid)         | Для сервисов, требующих уникальный строковый идентификатор. | Строка в UUID-формате.                                                   | "aabbccdd-eeff-0011-2233-445566778899"                                                                                                                                                                   |
Пример  на `random_sting` генерации случайного значения и его использование на примере бакета.
```
resource "yandex_storage_bucket" "this" {
  bucket     = "bucket-${random_string.bucket_name.result}"
  access_key = yandex_iam_service_account_static_access_key.this.access_key
  secret_key = yandex_iam_service_account_static_access_key.this.secret_key
}

resource "random_string" "bucket_name" {
  length  = 8
  special = false
  upper   = false
}
```
С помощью ресурса `random_string.bucket_name` генерируется случайный идентификатор. При создании бакета указывается его имя — `"bucket-${random_string.bucket_name.result}"`, где:

- `bucket-` — константа;
- `${}` — конструкция для константы (используется для обращения к значению переменной);
- `random_string.bucket_name` — значение ресурса;
- `result` — значение атрибута.

Случайное значение не является уникальным. Если такой бакет уже существует, вы получите ошибку уникальности имени бакета. В таком случае можно пересоздать случайное значение и применить изменения инфраструктуры с помощью команды:
```
$ terraform apply -replace="random_string.bucket_name"
```

Здесь с помощью опции `-replace` передаётся требование заменить ресурс, что ведёт к генерации нового случайного значения.