## Очистка prod 
#### 1. Безопасная очистка (Рекомендуется для PROD)

Эти команды удаляют только те компоненты, которые уже не нужны, но **сохраняют возможность удаления последних установленных обновлений**.

- **Стандартная очистка компонентов:**  
    `dism /online /cleanup-image /startcomponentcleanup`  
    Она удаляет замещенные компоненты немедленно, не дожидаясь 30-дневного автоматического срока. При этом **возможность отката сохраняется** для актуальных патчей.
- **Использование штатной «Очистки диска»:**  
    Запустите `cleanmgr.exe` от имени администратора. Выберите пункт **«Очистка обновлений Windows»**(Windows Update Cleanup). Это самый мягкий способ, официально поддерживаемый Microsoft для серверов.
- **Сжатие ОС (Compact OS):**  
    `compact /compactos:always`  
    Это сжимает системные файлы с помощью алгоритмов, которые не мешают работе обновлений и не блокируют откаты. Это безопасно для производительности на современном железе. 

#### 2. Рискованная очистка (Не рекомендуется для PROD без бэкапа)

Команда, которую вы упоминали ранее с параметром `/ResetBase`, является «точкой невозврата».

- **Команда:** `dism /online /cleanup-image /startcomponentcleanup /resetbase`
- **Последствия для прода:** Она «запечатывает» текущее состояние системы как новое базовое. После этого **удаление любых ранее установленных обновлений станет невозможным**.
- **Риск:** Если после очистки вы обнаружите критическую ошибку в работе ПО, вызванную прошлым обновлением, вы не сможете его деинсталлировать штатно. 

#### 3. Очистка старых сервис-паков

Если сервер обновлялся давно, можно безопасно удалить файлы старых пакетов обновлений:  
`dism /online /cleanup-image /spsuperseded`  
Это также сделает текущий Service Pack (если он есть) неудаляемым, но освободит место. 

#### Резюме для  случая:

1. Сначала выполните **анализ**: `dism /online /cleanup-image /analyzecomponentstore`.
2. Используйте **`cleanmgr`** или **`startcomponentcleanup`** (БЕЗ `/resetbase`).
3. Если места все еще критически мало, примените **`compact /compactos:always`**.
4. Используйте `/resetbase` только если у вас есть **полный бэкап (Snapshot/Image)** всей системы и вы уверены в стабильности последних обновлений.
---

## Для безопасной очистки 
Windows Server 2019 в **продуктивной среде** (без использования `/ResetBase`, чтобы сохранить возможность отката), используйте этот PowerShell-скрипт. Он выполняет анализ, базовую очистку компонентов, сжатие системных файлов и удаление временных данных.

#### PowerShell-скрипт для безопасной очистки.
```powershell
# 1. Анализ хранилища компонентов (WinSxS)
Write-Host "--- Анализ хранилища компонентов ---" -ForegroundColor Cyan
dism /online /cleanup-image /analyzecomponentstore

# 2. Безопасная очистка (без /ResetBase)
# Удаляет только те компоненты, которые уже замещены новыми, 
# сохраняя возможность отката последних патчей.
Write-Host "`n--- Запуск безопасной очистки компонентов ---" -ForegroundColor Cyan
dism /online /cleanup-image /startcomponentcleanup

# 3. Сжатие бинарных файлов ОС (Compact OS)
# Безопасно для PROD, не влияет на возможность удаления обновлений.
Write-Host "`n--- Проверка и применение сжатия ОС ---" -ForegroundColor Cyan
compact /compactos:always

# 4. Очистка временных файлов и логов
Write-Host "`n--- Очистка временных системных файлов ---" -ForegroundColor Cyan
$TempPaths = @(
    "$env:SystemRoot\Temp\*",
    "$env:LOCALAPPDATA\Temp\*"
)

foreach ($Path in $TempPaths) {
    Get-ChildItem -Path $Path -Recurse -ErrorAction SilentlyContinue | 
    Remove-Item -Force -Recurse -ErrorAction SilentlyContinue
}

# 5. Очистка кэша обновлений (SoftwareDistribution)
# Только если служба обновлений не занята в данный момент.
Write-Host "`n--- Очистка кэша обновлений (Download) ---" -ForegroundColor Cyan
Stop-Service -Name wuauserv -Force -ErrorAction SilentlyContinue
Get-ChildItem -Path "$env:SystemRoot\SoftwareDistribution\Download\*" -Recurse | 
Remove-Item -Force -Recurse -ErrorAction SilentlyContinue
Start-Service -Name wuauserv -ErrorAction SilentlyContinue

Write-Host "`nГотово! Система очищена безопасными методами." -ForegroundColor Green
```

##### Что делает этот скрипт:

- **`dism ... /startcomponentcleanup`**: Выполняет немедленную очистку устаревших версий компонентов [1.3.1](https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/clean-up-the-winsxs-folder?view=windows-11). В отличие от версии с `/ResetBase`, этот метод **оставляет пути для отката**установленных обновлений [1.1.2](https://www.reddit.com/r/msp/comments/ldfskc/safe_disk_cleanup_automation_for_servers/).
- **`compact /compactos:always`**: Сжимает файлы операционной системы. Это современный механизм Windows Server 2019, который экономит место на диске практически без ущерба для производительности [1.2.6](https://bobcares.com/blog/clean-up-and-compress-winsxs-folder-on-windows-server-windows-10/).
- **Очистка `SoftwareDistribution\Download`**: Удаляет уже скачанные и установленные файлы обновлений, которые часто "застревают" и занимают гигабайты.
- **Безопасность**: Скрипт не затрагивает критические системные файлы и не делает обновления "неудаляемыми".

Перед запуском в **PROD** обязательно проверьте наличие **актуального бэкапа**системы, так как любые операции с системными файлами несут минимальный риск.