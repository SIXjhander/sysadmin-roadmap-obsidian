# Системы печати Windows

> Централизованное управление печатью в Windows инфраструктуре

## 🎯 Почему это важно

Управление принтерами - одна из частых задач системного администратора. Правильно настроенный Print Server экономит часы времени и упрощает поддержку.

## 📋 Необходимые знания

- [[Windows Server]] - Print Server роль
- [[Active Directory]] - для deployment через GPO
- [[PowerShell]] - для автоматизации

## 📊 Уровни владения

### [[Junior SysAdmin]] уровень

#### Базовая настройка принтеров

**Установка Print Server роли:**
```powershell
Install-WindowsFeature Print-Services -IncludeManagementTools
```

**Добавление локального принтера:**
1. Server Manager → Print Management
2. Print Servers → Add Printer
3. Выбрать тип подключения (Network, Local, IP)
4. Установить драйвер
5. Share принтер

**Добавление через PowerShell:**
```powershell
# Добавить TCP/IP принтер
Add-PrinterPort -Name "IP_192.168.1.100" -PrinterHostAddress "192.168.1.100"

# Установить драйвер
Add-PrinterDriver -Name "HP Universal Printing PCL 6"

# Добавить принтер
Add-Printer -Name "Office-Printer-01" `
    -DriverName "HP Universal Printing PCL 6" `
    -PortName "IP_192.168.1.100" `
    -Shared `
    -ShareName "Office-Printer-01"
```

**Подключение к сетевому принтеру (клиент):**
```powershell
# Через PowerShell
Add-Printer -ConnectionName "\\PrintServer\Office-Printer-01"

# Через GUI
\\PrintServer → двойной клик на принтер
```

#### Управление правами доступа

```powershell
# Назначить права на принтер
$printer = Get-Printer -Name "Office-Printer-01"
Set-PrinterPermission -PrinterName $printer.Name `
    -UserName "CONTOSO\Domain Users" `
    -AccessMask Print

# Типы прав:
# Print - печать
# Manage Documents - управление очередью
# Manage Printers - полный контроль
```

#### Troubleshooting базовых проблем

**Типичные проблемы:**
1. **Принтер offline**
   ```powershell
   # Проверить статус
   Get-Printer -Name "Office-Printer-01"
   
   # Restart spooler service
   Restart-Service Spooler
   ```

2. **Застрявшая очередь**
   ```powershell
   # Очистить очередь
   $printer = Get-Printer -Name "Office-Printer-01"
   Get-PrintJob -PrinterObject $printer | Remove-PrintJob
   ```

3. **Драйвер не работает**
   - Переустановить драйвер
   - Проверить совместимость с Windows версией

### [[Middle SysAdmin]] уровень

#### Print Server (централизованное управление)

**Архитектура Print Server:**
```
Клиенты → Print Server (драйверы, очереди) → Физические принтеры
```

**Преимущества:**
- Централизованное управление драйверами
- Единая точка для настройки
- Логирование печати
- Follow-me printing
- Квоты на печать

**Настройка Print Server:**
```powershell
# Добавить несколько принтеров массово
$printers = Import-Csv "C:\printers.csv"
# CSV: Name,IP,Driver,Location

foreach ($p in $printers) {
    Add-PrinterPort -Name "IP_$($p.IP)" -PrinterHostAddress $p.IP
    
    Add-Printer -Name $p.Name `
        -DriverName $p.Driver `
        -PortName "IP_$($p.IP)" `
        -Location $p.Location `
        -Shared `
        -ShareName $p.Name
}
```

#### Deployment через Group Policy

**Создание GPO для принтеров:**
1. Group Policy Management
2. Create new GPO: "Printer Deployment"
3. Edit GPO:
   - User Configuration → Preferences → Control Panel Settings → Printers
   - New → Shared Printer
   - Action: Create/Update
   - Share path: `\\PrintServer\Office-Printer-01`
   - Default: Yes (опционально)
   - Item-level targeting (по location, по группе)

**PowerShell deployment:**
```powershell
# Создать GPO с принтером
New-GPO -Name "Printer-Moscow" -Comment "Printers for Moscow office"

# Configure через XML (требует настройки)
$gpo = Get-GPO -Name "Printer-Moscow"
# Import-GPO или Set-GPRegistryValue
```

**Item-Level Targeting примеры:**
- Компьютер в определенной OU
- Член определенной Security группы
- IP адрес в определенном subnet
- Site (AD Sites and Services)

#### Driver Management

**Проблемы с драйверами:**
- x86 vs x64 драйверы
- Windows версии (7/10/11)
- Type 3 (user-mode) vs Type 4 (v4 print driver)

**Установка дополнительных драйверов:**
```powershell
# Добавить x86 драйвер на x64 сервер
Add-PrinterDriver -Name "HP Universal Printing PCL 6" -InfPath "C:\Drivers\hpcu223u.inf"
```

**Driver Isolation:**
```powershell
# Изолировать драйвер (безопасность)
Set-PrinterDriver -Name "HP Universal Printing PCL 6" -PrinterDriverAttributes Shared
```

#### Printer Pooling

**Пул принтеров** (несколько физических = один логический):
```powershell
# Создать несколько портов
Add-PrinterPort -Name "IP_192.168.1.100"
Add-PrinterPort -Name "IP_192.168.1.101"

# Добавить принтер с pooling
Add-Printer -Name "Pool-Printer" `
    -DriverName "HP Universal Printing PCL 6" `
    -PortName "IP_192.168.1.100","IP_192.168.1.101"
```

#### Branch Office Printing

**Проблема:** Печать через WAN медленная

**Решение 1: Branch Office Direct Printing**
- Клиенты печатают напрямую на локальный принтер
- Но драйверы берутся с центрального Print Server

**Решение 2: Local Print Server в каждом офисе**

#### Мониторинг печати

```powershell
# Event Log для печати
Get-WinEvent -LogName "Microsoft-Windows-PrintService/Operational" -MaxEvents 100

# Audit printing (через GPO):
# Computer Config → Windows Settings → Security Settings → 
# → Advanced Audit Policy → Object Access → Audit File Share

# Просмотр активных jobs
Get-PrintJob -ComputerName PrintServer -PrinterName * | 
    Format-Table PrinterName, UserName, Size, Submitted
```

### [[Senior SysAdmin]] уровень

#### Enterprise Print Management

**Print Management Console:**
- Custom Filters для мониторинга
- Automated Printer Migration
- Printer Forms management
- Прав centralized management

**Print Migration:**
```powershell
# Export конфигурации
Export-WindowsFeature Print-Services -Path "C:\PrintBackup" -Computer PrintServer01

# Import на новый сервер
Import-WindowsFeature -Path "C:\PrintBackup" -Computer PrintServer02
```

**PrintBrm.exe (Print Backup and Restore):**
```cmd
# Backup всех принтеров
PrintBrm.exe -b -s \\PrintServer -f C:\Backup\printers.printerExport

# Restore
PrintBrm.exe -r -s \\NewPrintServer -f C:\Backup\printers.printerExport
```

#### Follow-Me Printing (Pull Printing)

**Концепция:**
1. Пользователь печатает
2. Job удерживается на сервере
3. Пользователь подходит к любому принтеру
4. Аутентифицируется (карта/PIN)
5. Забирает свои jobs

**Решения:**
- PaperCut
- PrinterLogic
- UniFlow
- Follow-Me-Printing в Xerox/Canon

#### Secure Printing

```powershell
# Require PIN для печати
Set-Printer -Name "Secure-Printer" `
    -KeepPrintedJobs $true `
    -EnablePinPrinting $true
```

#### Print Quotas

**Контроль стоимости печати:**
- Tracking кто сколько печатает
- Limits на количество страниц
- Color vs B&W pricing
- Department chargebacks

**Решения:**
- Windows Server Print Accounting (базовое)
- PaperCut (popular)
- PrinterLogic
- SafeCom

#### Cloud Printing Integration

**Гибридная печать:**
- On-premise Print Server
- + Universal Print (Microsoft 365)
- + Google Cloud Print (deprecated)

**Universal Print (Microsoft):**
```powershell
# Требует:
# - Microsoft 365 E3/E5
# - Azure AD Premium
# - Print Server Connector

# Register printer to Universal Print
Register-UpPrinter -PrinterName "Office-Printer" `
    -ShareName "Office-Printer" `
    -UniversalPrintUrl "https://print.microsoft.com"
```

#### Advanced Troubleshooting

**Common Issues:**

1. **Slow printing**
   - Network bandwidth
   - Spooler bottleneck
   - Printer processing
   - Driver issues

2. **Random failures**
   - Check Event Logs
   - Driver isolation issues
   - Memory leaks in spooler
   - Corrupted spool files

3. **Authentication issues**
   - Kerberos delegation
   - Constrained delegation for Print Server

**Debugging:**
```powershell
# Enable Print Service Operational log
wevtutil sl Microsoft-Windows-PrintService/Operational /e:true

# Set spooler tracing
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Print" /v LogLevel /t REG_DWORD /d 0xFFFFFFFF

# Logs location:
# C:\Windows\System32\spool\PRINTERS\*.SHD
# C:\Windows\System32\spool\PRINTERS\*.SPL
```

## 🔗 Связанные технологии

- [[Windows Server]] - платформа
- [[Active Directory]] - user/computer management
- [[PowerShell]] - автоматизация
- [[Основы сетей]] - TCP/IP printing

## 📚 Ресурсы

- Microsoft Print Management documentation
- PaperCut documentation (for enterprise)
- TechNet Print Services articles

## 💡 Best Practices

1. **Централизуй Print Server** - не 100 принтеров на клиентах
2. **Используй GPO** для deployment
3. **Type 4 драйверы** где возможно (более стабильные)
4. **Мониторь очереди** - automated alerts
5. **Regular maintenance** - clear old jobs
6. **Document printer locations** - в AD атрибутах
7. **Backup конфигурации** - monthly PrintBrm backups

## 🚨 Типичные ошибки

1. **Установка драйверов на клиентах** - используй Print Server!
2. **Нет резервного Print Server** - SPOF
3. **Плохие naming conventions** - "Printer1", "HP2"
4. **Не используется GPO** - ручная настройка на каждом ПК
5. **Игнорирование monitoring** - узнаешь о проблеме когда позвонят
6. **Прямая печать на принтер** минуя сервер - нет централизованного управления

---

**Связанные уровни:**
- 🎓 [[Junior SysAdmin]] - базовая настройка
- 📈 [[Middle SysAdmin]] - Print Server и GPO
- 🚀 [[Senior SysAdmin]] - enterprise решения

#технология #печать #windows #принтеры
