# Microsoft 365

> Облачная экосистема Microsoft (бывший Office 365)

## 🎯 Что это такое

Microsoft 365 - это набор облачных сервисов Microsoft, включающий Office приложения, Exchange Online, SharePoint, Teams, OneDrive и многое другое. Это критически важная платформа для современных организаций.

## 📋 Компоненты Microsoft 365

### Основные сервисы

**Office Applications:**
- Word, Excel, PowerPoint, Outlook
- Desktop приложения (Microsoft 365 Apps)
- Web версии (Office Online)
- Mobile приложения

**Exchange Online:**
- Связано с: [[Exchange Server]]
- Облачная почта
- Calendar и Contacts
- 50-100 GB mailbox
- Anti-spam и anti-malware
- DLP (Data Loss Prevention)

**SharePoint Online:**
- Связано с: [[SharePoint]]
- Document management
- Intranet sites
- Team sites
- Document libraries
- Workflow automation (Power Automate)

**OneDrive for Business:**
- Cloud storage (1 TB+ per user)
- File sync
- File sharing
- Known Folder Move (Desktop, Documents, Pictures)
- Version history
- Backup для Windows

**Microsoft Teams:**
- Chat и messaging
- Video meetings
- Team channels
- File sharing
- App integration
- Phone system (Teams Calling)

**Azure Active Directory (AAD):**
- Связано с: [[Azure]]
- Cloud identity management
- Single Sign-On (SSO)
- Multi-Factor Authentication (MFA)
- Conditional Access
- Identity Protection

### Дополнительные сервисы

**Power Platform:**
- Power BI - analytics и dashboards
- Power Apps - low-code app development
- Power Automate - workflow automation
- Power Virtual Agents - chatbots

**Microsoft Intune:**
- Связано с: [[Windows Client]]
- Mobile Device Management (MDM)
- Mobile Application Management (MAM)
- Endpoint protection
- Configuration profiles

**Compliance Center:**
- Data Loss Prevention (DLP)
- Retention policies
- eDiscovery
- Compliance Manager

## 📊 Уровни владения

### [[Middle SysAdmin]] уровень

#### Базовое администрирование

**Microsoft 365 Admin Center:**
- admin.microsoft.com
- User management
- License assignment
- Service health
- Message center
- Support requests

**User Management:**
```powershell
# Подключение к Microsoft 365
Connect-MsolService

# Создание пользователя
New-MsolUser -UserPrincipalName "john.doe@company.com" `
    -DisplayName "John Doe" `
    -FirstName "John" `
    -LastName "Doe"

# Назначение лицензии
Set-MsolUserLicense -UserPrincipalName "john.doe@company.com" `
    -AddLicenses "company:ENTERPRISEPACK"

# Включение MFA
Set-MsolUser -UserPrincipalName "john.doe@company.com" `
    -StrongAuthenticationRequirements @($true)
```

**Exchange Online Administration:**
```powershell
# Подключение
Connect-ExchangeOnline

# Создание mailbox
New-Mailbox -UserPrincipalName "mailbox@company.com" -DisplayName "Shared Mailbox" -Shared

# Настройка forwarding
Set-Mailbox -Identity "user@company.com" -ForwardingAddress "other@company.com"

# Mail flow rules
New-TransportRule -Name "Block external forwarding" -FromScope InOrganization -SentToScope NotInOrganization -MessageTypeMatches AutoForward -RejectMessageEnhancedStatusCode 5.7.1
```

**SharePoint Online Management:**
```powershell
# Подключение
Connect-SPOService -Url https://company-admin.sharepoint.com

# Создание site collection
New-SPOSite -Url https://company.sharepoint.com/sites/teamsite -Owner admin@company.com -StorageQuota 1000 -Title "Team Site"

# Управление правами
Set-SPOSite -Identity https://company.sharepoint.com/sites/teamsite -SharingCapability ExternalUserSharingOnly
```

**Teams Administration:**
- Teams admin center (admin.teams.microsoft.com)
- Teams policies
- Calling policies
- Meeting policies
- Guest access configuration

**Security Baselines:**
- Baseline security recommendations
- Security Score (secure.microsoft.com)
- Identity Secure Score
- Compliance Score

### [[Senior SysAdmin]] уровень

#### Advanced Configuration

**Azure AD Connect (Hybrid Identity):**
- Связано с: [[Active Directory]], [[Azure]]
- On-premises AD sync с Azure AD
- Password Hash Sync (PHS)
- Pass-through Authentication (PTA)
- Federation (AD FS)
- Device sync
```powershell
# Запуск sync вручную
Start-ADSyncSyncCycle -PolicyType Delta
```

**Conditional Access:**
```
IF user = member of "Executives" 
AND location = not trusted 
AND device = not compliant 
THEN require = MFA + compliant device
```
- Location-based policies
- Device compliance policies
- Risk-based policies
- App protection policies

**Microsoft Defender for Office 365:**
- ATP (Advanced Threat Protection)
- Safe Links
- Safe Attachments
- Anti-phishing policies
- Threat Explorer

**Information Protection:**
- Sensitivity labels
- DLP policies across services
- Encryption (Azure Information Protection)
- Rights Management (RMS)

**Compliance:**
- Retention policies
- eDiscovery для legal holds
- Audit logging
- Litigation hold
- GDPR compliance

**Microsoft Intune (MDM/MAM):**
```powershell
# Подключение к Intune
Connect-MSGraph

# Создание configuration profile
$profile = New-IntuneDeviceConfigurationPolicy -DisplayName "Corporate Wi-Fi"
```
- Device enrollment
- App deployment
- Configuration profiles
- Compliance policies
- Conditional Access integration

**Power Platform Governance:**
- Data Loss Prevention для Power Apps
- Environment management
- Connector policies
- Power BI tenant settings

**Migration to Microsoft 365:**
- Email migration (IMAP, PST, cutover, staged, hybrid)
- SharePoint migration
- File server to OneDrive/SharePoint
- Migration tools (SharePoint Migration Tool, Migration Manager)

**Automation:**
```powershell
# Массовое создание пользователей из CSV
Import-Csv users.csv | ForEach-Object {
    New-MsolUser -UserPrincipalName $_.Email `
        -DisplayName $_.Name `
        -FirstName $_.FirstName `
        -LastName $_.LastName `
        -UsageLocation "US" `
        -LicenseAssignment "company:ENTERPRISEPACK"
}
```

## 🔗 Связанные технологии

### On-premises Integration
- [[Active Directory]] - hybrid identity
- [[Exchange Server]] - hybrid Exchange
- [[SharePoint]] - hybrid SharePoint
- [[Windows Server]] - AD Connect server

### Cloud Services
- [[Azure]] - underlying platform
- [[Windows Client]] - Microsoft 365 Apps deployment
- [[Intune]] - device management

## 📚 Ресурсы для изучения

### Официальная документация
- Microsoft 365 documentation (docs.microsoft.com)
- Microsoft 365 roadmap
- Message Center updates

### Сертификации
- **MS-900**: Microsoft 365 Fundamentals
- **MS-102**: Microsoft 365 Administrator
- **MS-500**: Microsoft 365 Security Administrator
- **MS-700**: Managing Microsoft Teams

### Практика
- Microsoft 365 Developer Program (бесплатный tenant на 90 дней)
- Trial licenses

## 💡 Best Practices

### Licensing
1. **Правильно выбирай планы** - не переплачивай
2. **Используй группы для назначения** - automation
3. **Мониторь unused licenses**
4. **Понимай разницу** E3 vs E5

### Security
1. **Обязательно MFA** для всех пользователей
2. **Conditional Access** для критичных приложений
3. **DLP policies** для защиты данных
4. **Regular security audits**
5. **Enable audit logging**

### Governance
1. **Naming conventions** для Teams/Groups
2. **Lifecycle policies** - expiration и archive
3. **Guest access policies**
4. **Sharing policies** - internal vs external
5. **Data classification**

### Performance
1. **OneDrive Known Folder Move** - для backup
2. **Teams best practices** - архитектура каналов
3. **SharePoint limits** - размер библиотек
4. **Exchange limits** - mailbox quotas

## 🎯 Практические задачи

### Middle уровень
- [ ] Создать и настроить Microsoft 365 tenant
- [ ] Добавить и верифицировать домен
- [ ] Создать 10+ пользователей с лицензиями
- [ ] Настроить Exchange Online
- [ ] Создать Teams для отдела
- [ ] Настроить SharePoint site
- [ ] Включить MFA для пользователей
- [ ] Настроить mail flow rules

### Senior уровень
- [ ] Настроить Azure AD Connect (hybrid)
- [ ] Реализовать Conditional Access policies
- [ ] Настроить DLP across services
- [ ] Провести migration из Exchange on-prem
- [ ] Настроить Microsoft Defender for Office 365
- [ ] Внедрить Intune для device management
- [ ] Создать compliance policies
- [ ] Автоматизировать user provisioning

## 🚨 Типичные ошибки

1. **Не включать MFA** - major security risk
2. **Over-licensing** - платить за неиспользуемые license
3. **Слабые пароли** - нет password policy
4. **Неправильный external sharing** - data leakage
5. **Не использовать groups** - управление по одному пользователю
6. **Не мониторить service health**
7. **Не тестировать изменения** - особенно mail flow
8. **Игнорировать audit logs**
9. **Нет backup стратегии** - хотя Microsoft не отвечает за backup
10. **Не обучать пользователей** - security awareness

## 📈 Plans Comparison

| Feature | Business Basic | Business Standard | E3 | E5 |
|---------|---------------|-------------------|----|----|
| Office Apps | Web only | Desktop + Web | Desktop + Web | Desktop + Web |
| Exchange | 50 GB | 50 GB | 100 GB | 100 GB |
| OneDrive | 1 TB | 1 TB | Unlimited* | Unlimited* |
| Teams | ✓ | ✓ | ✓ | ✓ + Phone System |
| Azure AD P1 | - | - | ✓ | ✓ |
| Azure AD P2 | - | - | - | ✓ |
| Defender for Office 365 | - | - | - | ✓ (Plan 2) |
| Advanced Compliance | - | - | - | ✓ |

*Unlimited with 5+ users

---

**Связанные уровни:**
- 📈 [[Middle SysAdmin]] - базовое администрирование
- 🚀 [[Senior SysAdmin]] - advanced configuration и hybrid

**Требует знания:**
- [[Azure]] - cloud platform
- [[Active Directory]] - для hybrid
- [[PowerShell]] - automation

#технология #microsoft #microsoft365 #cloud #saas
