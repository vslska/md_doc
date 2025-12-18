
Вам потребуется скрипт, который запускается на **удаленном** сервере (через `Invoke-Command` или `Enter-PSSession`) и взаимодействует с локальным клиентом SCCM через WMI.

### Предварительное условие

Для выполнения этих действий у вашей учетной записи должны быть права локального администратора на целевом сервере.

### 1. Запуск цикла оценки развертывания обновлений

Первый шаг — убедиться, что клиент SCCM "знает" о новых доступных обновлениях. Вы можете принудительно запустить цикл оценки обновлений:

**Локально на сервере (или внутри `Enter-PSSession`):**
```powershell
# Запуск цикла оценки развертывания программных обновлений
Invoke-WmiMethod -Namespace root\ccm -Class SMS_CLIENT -Name TriggerSchedule "{00000000-0000-0000-0000-000000000108}"
Write-Host "Запущен цикл оценки обновлений SCCM. Обновления скоро появятся как доступные."
```

**Удаленно с вашего рабочего места (используя `Invoke-Command`):**
```powershell
Invoke-Command -ComputerName ИМЯ_ВАШЕГО_СЕРВЕРА -ScriptBlock {
    Invoke-WmiMethod -Namespace root\ccm -Class SMS_CLIENT -Name TriggerSchedule "{00000000-0000-0000-0000-000000000108}"
}
```

### 2. Скрипт для установки всех доступных обновлений

После того как клиент SCCM обнаружил обновления, вы можете использовать скрипт, который просматривает все доступные обновления и инициирует их установку через WMI-класс `CCM_SoftwareUpdatesManager`. 

Этот скрипт нужно запускать на **удаленном** хосте (через `Invoke-Command`):
```powershell
$ComputerName = "ИМЯ_ВАШЕГО_СЕРВЕРА"

Invoke-Command -ComputerName $ComputerName -ScriptBlock {
    Write-Host "Поиск и запуск установки доступных обновлений SCCM..." -ForegroundColor Cyan
    
    # Получаем все обновления со статусом EvaluationState = 0 (Неизвестно) или 1 (Доступно)
    $Updates = Get-WmiObject -Namespace "root\ccm\clientSDK" -Class CCM_SoftwareUpdate | Where-Object { $_.EvaluationState -eq 0 -or $_.EvaluationState -eq 1 }

    if ($Updates) {
        Write-Host "Найдено $($Updates.Count) обновлений для установки." -ForegroundColor Green
        
        # Запуск процесса установки через WMI метод InstallUpdates
        # Аргумент (,$Updates) передает массив объектов WMI в метод
        Invoke-WmiMethod -Class CCM_SoftwareUpdatesManager -Name InstallUpdates -ArgumentList (,$Updates) -Namespace root\ccm\clientsdk | Out-Null
        
        Write-Host "Процесс установки запущен." -ForegroundColor Green
    } else {
        Write-Host "Обновлений, готовых к установке, не найдено." -ForegroundColor Yellow
    }
}
```

### 3. Проверка статуса

Вы можете проверить текущий статус установки обновлений, посмотрев журнал клиента SCCM (`C:\Windows\CCM\Logs\UpdatesDeployment.log`) или запросив WMI-класс `CCM_SoftwareUpdate` снова, наблюдая за изменением `EvaluationState` (например, на `5` – Установка завершена, или `7` – Требуется перезагрузка).
Используя эти WMI-вызовы через PowerShell, вы полностью обходите графический интерфейс Software Center и автоматизируете процесс установки обновлений в духе Linux/командной строки.