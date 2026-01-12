# Active Directory

> Служба каталогов Microsoft для управления сетевыми ресурсами

## 🎯 Почему это критически важно

Active Directory (AD) - это сердце Windows инфраструктуры в enterprise окружении. AD обеспечивает централизованную аутентификацию, авторизацию и управление политиками для всей сети. Без понимания AD невозможно эффективно работать в Windows инфраструктуре.

## 📋 Необходимые знания перед изучением

**Обязательно:**
- [[Windows Server]] - AD является ролью Windows Server
- [[DNS]] - AD полностью зависит от DNS
- [[Основы сетей]] - понимание сетевой инфраструктуры

**Полезно:**
- [[PowerShell]] - для автоматизации AD задач
- [[LDAP]] - протокол доступа к каталогу

## 📊 Основные концепции

### Что такое Active Directory?

**Active Directory Domain Services (AD DS)** - это:
- Централизованная база данных пользователей, компьютеров, групп
- Служба аутентификации (Kerberos, NTLM)
- Система управления политиками (Group Policy)
- Иерархическая структура организации (OU)
- DNS-based domain structure

### Ключевые компоненты:

**1. Domain (Домен)**
- Административная и security граница
- Единое пространство имен (contoso.local)
- Общая база данных (NTDS.DIT)
- Общие policies и trusts

**2. Domain Controller (DC)**
- Сервер с ролью AD DS
- Хранит копию AD базы данных
- Обрабатывает аутентификацию
- Реплицирует изменения между DC
- **Минимум 2 DC для redundancy!**

**3. Forest (Лес)**
- Коллекция доменов
- Общая schema
- Общий Global Catalog
- Автоматические trusts внутри forest

**4. Tree (Дерево)**
- Иерархия доменов с общим namespace
- contoso.com → eu.contoso.com → uk.eu.contoso.com

**5. Organizational Unit (OU)**
- Контейнер для объектов
- Используется для делегирования и применения GPO
- НЕ security граница (в отличие от domain)

## 📊 Уровни владения

### [[Junior SysAdmin]] уровень

#### Установка и базовая настройка

**Установка AD DS:**
```powershell
# 1. Установить роль
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools

# 2. Повысить до Domain Controller (новый forest)
Install-ADDSForest `
    -DomainName "contoso.local" `
    -DomainNetbiosName "CONTOSO" `
    -ForestMode "WinThreshold" `
    -DomainMode "WinThreshold" `
    -InstallDns:$true `
    -SafeModeAdministratorPassword (ConvertTo-SecureString "P@ssw0rd!" -AsPlainText -Force) `
    -Force:$true

# 3. Добавить дополнительный DC к существующему домену
Install-ADDSDomainController `
    -DomainName "contoso.local" `
    -InstallDns:$true `
    -Credential (Get-Credential) `
    -SafeModeAdministratorPassword (ConvertTo-SecureString "P@ssw0rd!" -AsPlainText -Force)
```

**Требования для AD:**
- Static IP адрес на DC
- DNS правильно настроен
- Правильное время (NTP)
- Firewall ports открыты

#### Создание и управление пользователями

**Через PowerShell:**
```powershell
# Создать пользователя
New-ADUser -Name "John Smith" `
    -GivenName "John" `
    -Surname "Smith" `
    -SamAccountName "jsmith" `
    -UserPrincipalName "jsmith@contoso.local" `
    -Path "OU=Users,OU=Employees,DC=contoso,DC=local" `
    -AccountPassword (ConvertTo-SecureString "TempPass123!" -AsPlainText -Force) `
    -Enabled $true `
    -ChangePasswordAtLogon $true

# Получить информацию о пользователе
Get-ADUser -Identity "jsmith" -Properties *

# Изменить атрибуты
Set-ADUser -Identity "jsmith" `
    -Title "IT Administrator" `
    -Department "IT" `
    -Office "Moscow" `
    -EmailAddress "jsmith@contoso.com"

# Деактивировать пользователя
Disable-ADAccount -Identity "jsmith"

# Удалить пользователя
Remove-ADUser -Identity "jsmith" -Confirm:$false

# Сброс пароля
Set-ADAccountPassword -Identity "jsmith" `
    -Reset `
    -NewPassword (ConvertTo-SecureString "NewPass123!" -AsPlainText -Force)

# Unlock account
Unlock-ADAccount -Identity "jsmith"
```

**Массовое создание из CSV:**
```powershell
# users.csv:
# FirstName,LastName,Username,Department,Title
# John,Smith,jsmith,IT,Administrator

Import-Csv "C:\users.csv" | ForEach-Object {
    New-ADUser -Name "$($_.FirstName) $($_.LastName)" `
        -GivenName $_.FirstName `
        -Surname $_.LastName `
        -SamAccountName $_.Username `
        -UserPrincipalName "$($_.Username)@contoso.local" `
        -Department $_.Department `
        -Title $_.Title `
        -Path "OU=Users,OU=Employees,DC=contoso,DC=local" `
        -AccountPassword (ConvertTo-SecureString "TempPass123!" -AsPlainText -Force) `
        -Enabled $true `
        -ChangePasswordAtLogon $true
}
```

#### Группы

**Типы групп:**
- **Security Groups** - для назначения прав
- **Distribution Groups** - для email списков (Exchange)

**Scopes:**
- **Domain Local** - для прав на ресурсы в домене
- **Global** - для пользователей одного домена
- **Universal** - для multi-domain окружений

**AGDLP/AGUDLP стратегия:**
```
Accounts → Global groups → Domain Local groups → Permissions
```

**Примеры:**
```powershell
# Создать Security группу
New-ADGroup -Name "IT Department" `
    -GroupScope Global `
    -GroupCategory Security `
    -Path "OU=Groups,DC=contoso,DC=local"

# Добавить пользователя в группу
Add-ADGroupMember -Identity "IT Department" -Members "jsmith"

# Получить членов группы
Get-ADGroupMember -Identity "IT Department"

# Получить группы пользователя
Get-ADPrincipalGroupMembership -Identity "jsmith"

# Удалить из группы
Remove-ADGroupMember -Identity "IT Department" -Members "jsmith" -Confirm:$false
```

#### Organizational Units (OU)

```powershell
# Создать OU
New-ADOrganizationalUnit -Name "Employees" `
    -Path "DC=contoso,DC=local" `
    -ProtectedFromAccidentalDeletion $true

# Создать вложенную OU
New-ADOrganizationalUnit -Name "IT" `
    -Path "OU=Employees,DC=contoso,DC=local"

# Переместить объект между OU
Move-ADObject -Identity "CN=John Smith,OU=Users,DC=contoso,DC=local" `
    -TargetPath "OU=IT,OU=Employees,DC=contoso,DC=local"

# Удалить OU (нужно снять защиту)
Set-ADOrganizationalUnit -Identity "OU=IT,DC=contoso,DC=local" `
    -ProtectedFromAccidentalDeletion $false
Remove-ADOrganizationalUnit -Identity "OU=IT,DC=contoso,DC=local" -Confirm:$false
```

#### Компьютерные аккаунты

```powershell
# Получить компьютеры
Get-ADComputer -Filter * -Properties OperatingSystem, LastLogonDate

# Переместить компьютер в OU
Move-ADObject -Identity "CN=PC001,CN=Computers,DC=contoso,DC=local" `
    -TargetPath "OU=Workstations,OU=Computers,DC=contoso,DC=local"

# Удалить неактивные компьютеры (не логинились 90 дней)
$date = (Get-Date).AddDays(-90)
Get-ADComputer -Filter {LastLogonDate -lt $date} -Properties LastLogonDate | 
    Disable-ADAccount
```

### [[Middle SysAdmin]] уровень

#### Group Policy Objects (GPO)

**Создание и управление:**
```powershell
# Создать GPO
New-GPO -Name "Security Baseline" -Comment "Company security settings"

# Связать с OU
New-GPLink -Name "Security Baseline" `
    -Target "OU=Computers,DC=contoso,DC=local" `
    -LinkEnabled Yes

# Получить GPO
Get-GPO -Name "Security Baseline"

# Получить GPO Report
Get-GPOReport -Name "Security Baseline" -ReportType Html -Path "C:\report.html"

# Backup GPO
Backup-GPO -Name "Security Baseline" -Path "C:\GPOBackups"

# Restore GPO
Restore-GPO -Name "Security Baseline" -Path "C:\GPOBackups" -BackupId <GUID>

# Import settings from another GPO
Import-GPO -BackupId <GUID> -TargetName "New GPO" -Path "C:\GPOBackups"
```

**Типичные GPO настройки:**

**1. Password Policy:**
- Computer Configuration → Policies → Windows Settings → Security Settings → Account Policies → Password Policy
  - Minimum password length: 12
  - Password complexity: Enabled
  - Maximum password age: 90 days
  - Password history: 24

**2. Account Lockout:**
  - Account lockout threshold: 5 attempts
  - Account lockout duration: 30 minutes
  - Reset lockout counter: 30 minutes

**3. Software Installation:**
```powershell
# Computer Configuration → Policies → Software Settings → Software Installation
# Right-click → New → Package
# Выбрать .msi файл с сетевой папки
```

**4. Folder Redirection:**
- User Configuration → Policies → Windows Settings → Folder Redirection
  - Documents → \\server\users$\%username%\Documents
  - Desktop → \\server\users$\%username%\Desktop

**5. Drive Mapping:**
```powershell
# User Configuration → Preferences → Windows Settings → Drive Maps
# Create → Mapped Drive
# Location: \\server\share
# Drive Letter: Z:
# Reconnect: Yes
```

**6. Printer Deployment:**
- User Configuration → Preferences → Control Panel Settings → Printers
- Deploy shared printer: \\printserver\printername

**7. Security Settings:**
- Disable USB storage
- Windows Firewall rules
- Local admin restrictions
- BitLocker enforcement

**8. Windows Update Settings:**
```
Computer Configuration → Policies → Administrative Templates → 
Windows Components → Windows Update
```

#### Sites and Services

**AD Sites** - физическая топология сети:
- Определяют репликацию между DC
- Оптимизируют аутентификацию (closest DC)
- Контролируют репликационный трафик

```powershell
# Создать Site
New-ADReplicationSite -Name "Moscow-Office"

# Создать Subnet
New-ADReplicationSubnet -Name "192.168.1.0/24" -Site "Moscow-Office"

# Создать Site Link
New-ADReplicationSiteLink -Name "Moscow-SPB" `
    -SitesIncluded "Moscow-Office", "SPB-Office" `
    -Cost 100 `
    -ReplicationFrequencyInMinutes 180
```

#### FSMO Roles (Flexible Single Master Operations)

**5 FSMO ролей:**

**Forest-wide (одна на forest):**
1. **Schema Master** - изменения схемы AD
2. **Domain Naming Master** - добавление/удаление доменов

**Domain-wide (одна на домен):**
3. **PDC Emulator** - основной DC для:
   - Time synchronization
   - Password changes
   - Account lockouts
   - Legacy compatibility
4. **RID Master** - выдает RID пулы для создания объектов
5. **Infrastructure Master** - обновляет cross-domain references

```powershell
# Посмотреть FSMO roles
Get-ADDomainController -Filter * | Select-Object Name, OperationMasterRoles

# Другой способ
netdom query fsmo

# Transfer FSMO роли (graceful)
Move-ADDirectoryServerOperationMasterRole -Identity "DC02" `
    -OperationMasterRole SchemaMaster, DomainNamingMaster, PDCEmulator, RIDMaster, InfrastructureMaster

# Seize FSMO роли (если DC мертв)
Move-ADDirectoryServerOperationMasterRole -Identity "DC02" `
    -OperationMasterRole PDCEmulator -Force
```

#### Replication

**Мониторинг репликации:**
```powershell
# Проверить репликацию
repadmin /replsummary

# Показать партнеров репликации
repadmin /showrepl

# Форсировать репликацию
repadmin /syncall /AdeP

# Проверить metadata репликации
repadmin /showrepl * /csv > repl-status.csv
```

**Troubleshooting репликации:**
```powershell
# DCDiag - полная диагностика DC
dcdiag /v /c /d /e /s:DC01

# Тест репликации
dcdiag /test:replications

# Проверка DNS
dcdiag /test:dns

# Проверка FSMO
dcdiag /test:fsmocheck
```

#### Trust Relationships

**Типы trust:**
- **Parent-Child** - автоматический, transitive, two-way
- **Tree-Root** - автоматический в forest
- **External** - между доменами разных forests
- **Forest** - между forests
- **Shortcut** - оптимизация в complex forest
- **Realm** - с non-Windows (Kerberos)

```powershell
# Создать External Trust
netdom trust contoso.local /domain:fabrikam.com /add /twoway /realm

# Проверить trust
nltest /trusted_domains

# Проверить secure channel
Test-ComputerSecureChannel -Verbose

# Repair secure channel
Test-ComputerSecureChannel -Repair
```

### [[Senior SysAdmin]] уровень

#### Multi-Forest Architecture

**Проектирование AD инфраструктуры:**
- Resource forests
- Account forests
- Forest trusts
- Selective authentication
- SID filtering

#### Fine-Grained Password Policies (FGPP)

**Разные password policies для разных групп:**
```powershell
# Создать Password Settings Object
New-ADFineGrainedPasswordPolicy -Name "Admins-PSO" `
    -Precedence 10 `
    -ComplexityEnabled $true `
    -MinPasswordLength 15 `
    -MinPasswordAge "1.00:00:00" `
    -MaxPasswordAge "60.00:00:00" `
    -PasswordHistoryCount 24 `
    -LockoutThreshold 3 `
    -LockoutDuration "00:30:00" `
    -LockoutObservationWindow "00:30:00"

# Применить к группе
Add-ADFineGrainedPasswordPolicySubject -Identity "Admins-PSO" `
    -Subjects "Domain Admins"

# Проверить применение
Get-ADUserResultantPasswordPolicy -Identity "admin.user"
```

#### Read-Only Domain Controller (RODC)

**RODC для branch offices:**
- Только чтение AD базы
- Password caching (избирательный)
- Filtered attributes
- Admin role separation

```powershell
# Установить RODC
Install-ADDSDomainController `
    -DomainName "contoso.local" `
    -ReadOnlyReplica `
    -SiteName "Branch-Office" `
    -Credential (Get-Credential)
```

#### Active Directory Recycle Bin

**Восстановление удаленных объектов:**
```powershell
# Включить Recycle Bin (необратимо!)
Enable-ADOptionalFeature -Identity "Recycle Bin Feature" `
    -Scope ForestOrConfigurationSet `
    -Target "contoso.local"

# Посмотреть удаленные объекты
Get-ADObject -Filter {IsDeleted -eq $true} -IncludeDeletedObjects

# Восстановить объект
Get-ADObject -Filter {Name -eq "John Smith" -and IsDeleted -eq $true} `
    -IncludeDeletedObjects | Restore-ADObject
```

#### Schema Modifications

**Расширение AD схемы:**
```powershell
# Регистрация DLL для расширения схемы
regsvr32 schmmgmt.dll

# Подключение к Schema
# Active Directory Schema snap-in в MMC

# PowerShell для schema (осторожно!)
$schema = Get-ADObject -SearchBase (Get-ADRootDSE).schemaNamingContext `
    -Filter {lDAPDisplayName -eq "user"}
```

⚠️ **ВНИМАНИЕ**: Изменения schema необратимы и влияют на весь forest!

#### Disaster Recovery

**Backup AD:**
```powershell
# Windows Server Backup
wbadmin start systemstatebackup -backuptarget:D: -quiet

# Через PowerShell
Backup-GPO -All -Path "C:\GPOBackups"
```

**Восстановление AD:**

**1. Authoritative Restore** (восстановить удаленный объект):
```cmd
# Загрузиться в DSRM (Directory Services Restore Mode)
# Восстановить System State
wbadmin start systemstaterecovery -version:<version>

# Authoritative restore
ntdsutil
activate instance ntds
authoritative restore
restore subtree "OU=Deleted,DC=contoso,DC=local"
```

**2. Forest Recovery** (полная катастрофа):
1. Восстановить первый DC из backup
2. Metadata cleanup для остальных DC
3. Promote новые DC
4. Restore остальные данные

#### Monitoring и Auditing

```powershell
# Включить Advanced Audit Policy
auditpol /set /category:"DS Access" /success:enable /failure:enable
auditpol /set /category:"Account Management" /success:enable /failure:enable

# Проверить audit settings
auditpol /get /category:*

# Мониторинг критичных событий
# Event IDs для AD:
# 4740 - Account lockout
# 4720 - Account created
# 4726 - Account deleted
# 4728 - User added to security group
# 4732 - User added to local group
# 4756 - User added to universal group
```

**AD Health Check Script:**
```powershell
# Comprehensive AD health check
dcdiag /v /c /d /e > dcdiag.txt
repadmin /showrepl * /csv > replication.csv
repadmin /replsummary > replsummary.txt
Get-ADReplicationFailure -Scope Forest
Get-ADDomainController -Filter * | Test-ComputerSecureChannel
```

## 🔗 Связанные технологии

### Критически важные зависимости
- [[Windows Server]] - платформа для AD
- [[DNS]] - **абсолютно критично** для AD
- [[PowerShell]] - основной инструмент управления

### Тесно связанные
- [[DHCP]] - часто интегрирован с AD
- [[Системы печати Windows]] - управление через AD/GPO
- [[Microsoft 365]] - гибридная идентификация
- [[Azure AD]] - облачное расширение AD

### Безопасность
- [[Основы информационной безопасности]] - AD security best practices
- PKI / Certificate Services - для аутентификации

## 📚 Ресурсы для изучения

### Книги
- "Active Directory" by Brian Desmond - библия AD
- "Active Directory Cookbook" by Robbie Allen
- "Mastering Active Directory" by Dishan Francis

### Онлайн
- Microsoft Learn - AD modules
- Microsoft Virtual Academy
- TechNet documentation

### Практика
- Домашняя лаборатория с 2+ DC
- Practice forest с child domains
- Настройка GPO и troubleshooting

## 💡 Best Practices

1. **Минимум 2 DC** для redundancy
2. **Proper DNS** - 99% проблем из-за DNS
3. **Regular backups** - System State на всех DC
4. **Document everything** - OU structure, GPO, trusts
5. **Least privilege** - не используй Domain Admin для всего
6. **Separate admin accounts** - admin.username для админских задач
7. **Test GPO** - перед применением на production
8. **Monitor replication** - ежедневно
9. **Time sync** - критично для Kerberos (max 5 min skew)
10. **Protected admin groups** - AdminSDHolder

## 🚨 Типичные ошибки

1. **Плохой DNS** - самая частая причина проблем
2. **Один DC** - single point of failure
3. **Неправильная OU structure** - сложно применять GPO
4. **Слишком много Domain Admins** - security risk
5. **Не тестировать восстановление** - backup без проверки бесполезен
6. **Игнорировать репликацию** - divergence между DC
7. **Изменения в production без теста** - может все сломать
8. **Нет документации** - через год не вспомнишь почему так сделано

---

**Связанные уровни:**
- 🎓 [[Junior SysAdmin]] - базовые навыки AD
- 📈 [[Middle SysAdmin]] - продвинутое администрирование
- 🚀 [[Senior SysAdmin]] - enterprise архитектура

#технология #active-directory #microsoft #identity #критический-навык
