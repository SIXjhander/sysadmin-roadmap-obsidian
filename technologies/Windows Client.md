# Windows Client

> Администрирование пользовательских версий Windows в корпоративной среде

## 🎯 Почему это важно

Системные администраторы должны уметь управлять десятками, сотнями или тысячами пользовательских компьютеров с Windows. Понимание Windows Client критически важно для поддержки пользователей и интеграции с [[Windows Server]] и [[Active Directory]].

## 📋 Версии Windows Client

**Современные версии:**
- Windows 10 (21H2, 22H2)
- Windows 11 (21H2, 22H2, 23H2)

**Редакции:**
- Home - домашняя, без domain join
- Pro - профессиональная, domain join, BitLocker
- Enterprise - корпоративная, дополнительные функции
- Education - для образовательных учреждений

## 📊 Уровни владения

### [[Junior SysAdmin]] уровень

#### Базовое администрирование

**Установка и настройка:**
- Чистая установка Windows
- Создание загрузочной флешки (Rufus, Media Creation Tool)
- Windows Out-of-Box Experience (OOBE)
- Sysprep для подготовки образа
- Unattended installation (answer files)

**Управление пользователями (локальные):**
- Local Users and Groups (lusrmgr.msc)
- User Account Control (UAC)
- Local Administrator
- Standard User
```powershell
# PowerShell управление
New-LocalUser -Name "testuser" -Password (ConvertTo-SecureString "Pass123!" -AsPlainText -Force)
Add-LocalGroupMember -Group "Users" -Member "testuser"
```

**Настройки системы:**
- Control Panel
- Settings app
- System Properties (sysdm.cpl)
- Device Manager (devmgmt.msc)
- Services (services.msc)
- Task Manager

**Обновления Windows:**
- Windows Update
- Update Assistant
- Feature updates vs Quality updates
- Update rings
- Troubleshooting обновлений

**Драйверы:**
- Automatic driver installation
- Manual driver installation
- Driver rollback
- Device Manager
- Windows Update для драйверов

**Troubleshooting основные проблемы:**
- Blue Screen of Death (BSOD)
- Event Viewer (eventvwr.msc)
- Reliability Monitor
- Performance issues
- Startup problems (Safe Mode, Startup Repair)

**Backup и восстановление:**
- File History
- System Restore
- System Image Backup
- Reset This PC
- Windows Recovery Environment (WinRE)

**Безопасность (базовый уровень):**
- Windows Defender Antivirus
- Windows Firewall
- User Account Control (UAC)
- Windows Security Center

### [[Middle SysAdmin]] уровень

#### Domain-joined системы

**Active Directory Integration:**
- Связано с: [[Active Directory]]
- Domain join
```powershell
# Присоединение к домену
Add-Computer -DomainName "company.local" -Credential (Get-Credential) -Restart
```
- Domain user logon
- Roaming profiles
- Folder redirection
- Cached credentials

**Group Policy применение:**
- Связано с: [[Windows Server]]
- gpupdate /force
- gpresult для диагностики
- rsop.msc (Resultant Set of Policy)
- Troubleshooting GPO применения

**Software Deployment:**
- Group Policy Software Installation (GPSI)
- Chocolatey для package management
- WinGet (Windows Package Manager)
```powershell
# WinGet examples
winget search chrome
winget install Google.Chrome
winget upgrade --all
```
- MSI installations
- MSIX packages

**Windows 10/11 специфичные функции:**

**Windows Update for Business:**
- Deferral settings
- Quality updates
- Feature updates
- Servicing channels (Semi-Annual Channel)

**Windows AutoPilot:**
- Zero-touch deployment
- Device registration
- Profile assignment
- Связано с: [[Microsoft 365]], [[Azure]]

**Microsoft Intune (MDM):**
- Mobile Device Management
- Связано с: [[Microsoft 365]]
- Configuration profiles
- Application deployment
- Compliance policies
- Remote wipe

**BitLocker Drive Encryption:**
```powershell
# Включить BitLocker
Enable-BitLocker -MountPoint "C:" -EncryptionMethod XtsAes256 -UsedSpaceOnly -TpmProtector

# Recovery key в AD
Backup-BitLockerKeyProtector -MountPoint "C:" -KeyProtectorId "{ID}"
```
- TPM (Trusted Platform Module)
- Recovery keys в AD
- Network unlock

**Windows To Go:**
- Portable Windows workspace
- USB drive с полной Windows

**Hyper-V Client:**
- Client Hyper-V (Windows Pro+)
- Виртуальные машины на рабочих станциях
- WSL 2 (Windows Subsystem for Linux)

**Remote Management:**
- Remote Desktop
- Remote Assistance
- PowerShell Remoting
```powershell
# PSRemoting
Enter-PSSession -ComputerName PC01
Invoke-Command -ComputerName PC01 -ScriptBlock { Get-EventLog -LogName System -Newest 10 }
```
- WinRM (Windows Remote Management)
- WMI (Windows Management Instrumentation)

**Performance Optimization:**
- Startup programs management
- Disk Cleanup
- Storage Sense
- Defragmentation (хотя SSD не нуждаются)
- Resource Monitor
- Performance Monitor

**Troubleshooting инструменты:**
- Windows Assessment and Deployment Kit (ADK)
- DISM (Deployment Image Servicing and Management)
```powershell
# Проверка и восстановление системных файлов
DISM /Online /Cleanup-Image /CheckHealth
DISM /Online /Cleanup-Image /RestoreHealth
sfc /scannow
```
- Windows Performance Toolkit
- Process Monitor (Sysinternals)
- Autoruns (Sysinternals)

### [[Senior SysAdmin]] уровень

#### Enterprise Deployment и Management

**MDT (Microsoft Deployment Toolkit):**
- Создание deployment shares
- Task sequences
- Boot images (WinPE)
- Lite Touch Installation
- Связано с: [[SCCM]]

**SCCM / Microsoft Endpoint Configuration Manager:**
- Operating System Deployment (OSD)
- Software distribution
- Patch management
- Compliance settings
- Software metering
- Remote control
- Reporting

**Windows Assessment and Deployment Kit:**
- Windows PE (Preinstallation Environment)
- User State Migration Tool (USMT)
- Volume Activation Management Tool (VAMT)
- Application Compatibility Toolkit

**Master Image Creation:**
- Sysprep и generalization
- Audit mode
- Answer files (unattend.xml)
- Driver injection
- Application layering

**Azure AD Join vs Hybrid Join:**
- Pure cloud management
- Связано с: [[Azure]], [[Microsoft 365]]
- Hybrid scenarios
- Conditional Access

**Windows Analytics:**
- Upgrade Readiness
- Update Compliance
- Device Health

**Advanced Security:**

**Windows Defender Advanced Threat Protection (ATP):**
- Endpoint Detection and Response (EDR)
- Threat analytics
- Automated investigation

**AppLocker:**
```powershell
# Создание AppLocker правила
New-AppLockerPolicy -Path "C:\temp\applocker.xml" -RuleType Publisher -User Everyone -RuleNamePrefix "Allow"
```
- Application whitelisting
- Executable rules
- Script rules

**Device Guard / Credential Guard:**
- Virtualization-based security
- Code integrity policies

**Windows Information Protection (WIP):**
- Data loss prevention
- App-level encryption

**Exploit Protection:**
- Attack surface reduction
- Controlled folder access

**Advanced Troubleshooting:**

**Windows Error Reporting:**
- Error reporting analysis
- Minidump analysis

**Performance Tuning:**
- Registry optimization
- Services optimization
- Group Policy optimization
- Boot time optimization

**Migration и Upgrade:**
- In-place upgrade
- USMT (User State Migration Tool)
- Windows Easy Transfer (deprecated)
- Feature update deployment strategies
- Compatibility testing

**PowerShell для массовых операций:**
```powershell
# Получить список всех PC в домене
Get-ADComputer -Filter * -Properties OperatingSystem, LastLogonDate

# Удаленная установка обновлений
Invoke-Command -ComputerName (Get-ADComputer -Filter *).Name -ScriptBlock {
    Install-WindowsUpdate -AcceptAll -AutoReboot
}

# Сбор инвентаризации
$computers = Get-ADComputer -Filter * -Properties Name
$inventory = foreach ($computer in $computers) {
    Invoke-Command -ComputerName $computer.Name -ScriptBlock {
        [PSCustomObject]@{
            ComputerName = $env:COMPUTERNAME
            RAM = (Get-WmiObject Win32_ComputerSystem).TotalPhysicalMemory / 1GB
            DiskSpace = (Get-WmiObject Win32_LogicalDisk -Filter "DeviceID='C:'").Size / 1GB
            OS = (Get-WmiObject Win32_OperatingSystem).Caption
        }
    }
}
```

**Compliance и Auditing:**
- Security baselines
- CIS benchmarks
- NIST guidelines
- Audit policies
- Security logs analysis

## 🔗 Связанные технологии

### Обязательные для Windows Client Admin
- [[Windows Server]] - серверная инфраструктура
- [[Active Directory]] - domain management
- [[PowerShell]] - автоматизация

### Microsoft экосистема
- [[Microsoft 365]] - облачные сервисы и Intune
- [[Azure]] - Azure AD, Autopilot
- [[SCCM]] - enterprise management

### Дополнительные
- [[Системы печати]] - client printing
- [[VPN]] - удаленный доступ

## 📚 Ресурсы для изучения

### Официальная документация
- Microsoft Learn
- Windows IT Pro documentation
- Windows 10/11 deployment guide

### Книги
- "Windows 10 and 11 for IT Professionals"
- "Mastering Windows Group Policy"
- "Windows PowerShell Cookbook"

### Сертификации
- **Microsoft 365 Certified: Modern Desktop Administrator Associate**
- **Microsoft Certified: Windows Client (MD-100, MD-101)**

### Практика
- Windows 10/11 Enterprise Evaluation
- Virtual machines для тестирования

## 💡 Best Practices

### Deployment
1. **Standardized images** - единый образ для схожих ролей
2. **Automated deployment** - минимум ручной работы
3. **Testing before rollout** - всегда тестируй
4. **Documentation** - документируй процесс

### Management
1. **Group Policy over local** - централизованное управление
2. **Least privilege principle** - минимальные права пользователям
3. **Regular patching** - обновления безопасности
4. **Monitoring** - отслеживай здоровье систем
5. **Backup user data** - OneDrive, File History, folder redirection

### Security
1. **BitLocker для ноутбуков** - обязательно
2. **Windows Defender** - не отключай без причины
3. **UAC enabled** - не отключай
4. **Local admin rights** - только когда необходимо
5. **LAPS для локальных админов**

### User Experience
1. **Fast logon** - оптимизируй GPO
2. **Roaming profiles** с осторожностью (большие)
3. **Folder redirection** - лучше чем roaming
4. **Self-service tools** - портал для пользователей

## 🎯 Практические задачи

### Junior уровень
- [ ] Чистая установка Windows 10/11
- [ ] Настройка Windows Update
- [ ] Установка драйверов вручную
- [ ] Создание локальных пользователей
- [ ] Настройка Windows Firewall
- [ ] Использование Event Viewer
- [ ] Backup и restore с File History

### Middle уровень
- [ ] Domain join компьютера
- [ ] Применение GPO к компьютеру
- [ ] Установка ПО через GPO
- [ ] Включение BitLocker
- [ ] Настройка folder redirection
- [ ] Удаленное управление через PSRemoting
- [ ] Troubleshooting GPO применения
- [ ] Создание WinPE boot media

### Senior уровень
- [ ] Создание master image с Sysprep
- [ ] Настройка MDT для deployment
- [ ] Настройка SCCM OSD
- [ ] Миграция с Windows 10 на 11 (in-place)
- [ ] Настройка Windows Autopilot
- [ ] Интеграция с Azure AD
- [ ] Создание AppLocker policies
- [ ] Написание PowerShell скриптов для массовых операций
- [ ] Compliance reporting

## 🚨 Типичные ошибки

1. **Давать всем local admin** - огромная security дыра
2. **Не тестировать обновления** - можешь сломать 1000 ПК
3. **Плохие образы** - bloatware и ненужное ПО
4. **Нет backup user data** - потеря данных
5. **Игнорировать Event Viewer** - пропускаешь проблемы
6. **Отключать UAC** - плохая идея
7. **Один master image для всех** - разные роли нужны разные настройки
8. **Не документировать changes** - не помнишь что менял
9. **Применять untested GPO к Production** - рискованно
10. **Игнорировать driver updates** - проблемы с оборудованием

## 🔧 Полезные инструменты

### Microsoft
- Windows Admin Center
- Sysinternals Suite
- Windows Performance Toolkit
- Windows ADK
- Microsoft Deployment Toolkit

### Third-party
- Chocolatey - package manager
- PDQ Deploy - software deployment
- PDQ Inventory - inventory management
- TeamViewer / AnyDesk - remote support

## 📈 Мониторинг Windows Clients

### Что мониторить
- Disk space
- Windows Update status
- Antivirus status
- Performance metrics
- Event logs errors
- Software compliance
- Security compliance

### Инструменты
- SCCM reporting
- Связано с: [[Системы мониторинга]]
- Azure Monitor (for Azure AD joined)
- Splunk / ELK для логов

## 🔄 Windows as a Service (WaaS)

**Servicing channels:**
- Semi-Annual Channel - feature updates 2 раза в год
- Long-Term Servicing Channel (LTSC) - для специальных систем

**Update types:**
- Feature updates - новые функции, 1-2 раза в год
- Quality updates - security и bug fixes, ежемесячно
- Driver updates - через Windows Update

**Deployment rings:**
1. IT test devices
2. Early adopters
3. General deployment
4. Critical devices (last)

---

**Связанные уровни:**
- 🎓 [[Junior SysAdmin]] - базовое администрирование
- 📈 [[Middle SysAdmin]] - domain integration
- 🚀 [[Senior SysAdmin]] - enterprise deployment

**Требует знания:**
- [[Windows Server]] - для domain join
- [[Active Directory]] - user management
- [[PowerShell]] - automation

#технология #windows #windows-client #microsoft #desktop-support
