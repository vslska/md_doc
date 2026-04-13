

Способ 1: Использование переменных окружения (Рекомендуемый для Terraform)

`Terraform Provider Yandex Cloud` умеет автоматически считывать учетные данные из переменных окружения без использования файла конфигурации `yc`.

Вам понадобится **JSON-файл ключа сервисного аккаунта** (например, `key.json`), который вы должны были создать ранее. Если у вас его нет, перейдите к Способу 2, чтобы создать новый.

1. **Установите переменные окружения** перед запуском Terraform:
```bash
   export YC_CLOUD_ID="b1g5mdflm8pbhnfrek05"
   export YC_FOLDER_ID="b1gqo6mejt29kur1ls2t"
   export YC_TOKEN="y0__xC9yNnCBxjB3RMgo9zhtRSnMixwlvmcg-p4Ke5VkHF8upaXOg" 
   # Примечание: Токен из вашего yc config list - это IAM-токен пользователя, 
   # а не сервисного аккаунта. Лучше использовать ключ сервисного аккаунта.
```
   
**ИЛИ (предпочтительнее)** если у вас есть JSON-ключ сервисного аккаунта:

```bash
export YC_CLOUD_ID="b1g5mdflm8pbhnfrek05"
export YC_FOLDER_ID="b1gqo6mejt29kur1ls2t"
export YC_SERVICE_ACCOUNT_KEY_FILE_PATH="./key.json"
```

- **Запустите Terraform** в папке с файлами `main.tf` и `variables.tf`, которые мы создали ранее:
 ```bash
terraform init
terraform apply
 ```


Способ 2: Восстановление/Пересоздание файла конфигурации `yc`

Если вам все же нужен рабочий CLI `yc`, вы можете восстановить его конфигурацию.

A. Если у вас есть **JSON**-ключ сервисного аккаунта:

1. Выполните команду аутентификации, указав путь к вашему файлу `key.json`:

```bash
yc config profile create sa-profile
yc iam key create --folder-id b1gqo6mejt29kur1ls2t --output key.json # Если ключа еще нет
yc config set service-account-key key.json 
yc config set cloud-id b1g5mdflm8pbhnfrek05
yc config set folder-id b1gqo6mejt29kur1ls2t
```

B. Если у вас нет **JSON-ключа** сервисного аккаунта (нужно создать новый):

Для этого вам потребуются права администратора в Yandex Cloud:

1. **Создайте новый ключ для сервисного аккаунта.**
    - Сначала снова авторизуйтесь как пользователь, используя свой Yandex ID (чтобы получить временный токен):

```bash
yc init 
# Следуйте инструкциям, введите свой токен y0__...
```

- Затем создайте новый JSON-файл ключа для вашего существующего сервисного аккаунта (замените `my-service-account-name` на реальное имя вашего аккаунта):

```bash
yc iam key create --folder-id b1gqo6mejt29kur1ls2t \
--service-account-name my-service-account-name \
-output sa-key.json
```

- **Переключите конфигурацию CLI на использование нового ключа сервисного аккаунта** (это более надежный способ аутентификации для автоматизации, чем пользовательский токен):
```bash
yc config profile create sa-profile
yc config set service-account-key sa-key.json
yc config set cloud-id b1g5mdflm8pbhnfrek05
yc config set folder-id b1gqo6mejt29kur1ls2t
```

После выполнения этих шагов CLI `yc` снова будет работать, и Terraform сможет использовать эти данные, если вы не устанавливали переменные окружения из Способа 1.