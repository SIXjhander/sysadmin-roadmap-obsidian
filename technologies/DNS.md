# DNS (Domain Name System)

> Система доменных имен - фундаментальный сервис интернета и корпоративных сетей

## 🎯 Почему это критически важно

DNS - это "телефонная книга интернета". Без DNS не будет работать практически ничего: веб-сайты, почта, Active Directory. **90% проблем с AD связаны с DNS!**

## 📋 Необходимые знания

- [[Основы сетей]] - TCP/IP, понимание портов
- [[Windows Server]] (для Windows DNS) или [[Linux]] (для BIND)

## 🌐 Как работает DNS

### Базовая концепция

```
Пользователь вводит: www.google.com
       ↓
DNS резолвинг → IP адрес: 142.250.185.36
       ↓
Браузер подключается к IP адресу
```

### DNS Hierarchy

```
. (Root)
    ↓
.com .net .org .ru (TLDs - Top Level Domains)
    ↓
google.com, microsoft.com (Second Level Domains)
    ↓
www.google.com, mail.google.com (Subdomains)
```

### DNS Resolution Process (полный)

```
1. Клиент → Local DNS Cache
2. Клиент → Configured DNS Server (рекурсивный запрос)
3. DNS Server → Root Server (. servers)
4. Root → TLD Server (.com servers)
5. TLD → Authoritative Name Server (google.com NS)
6. Authoritative → IP адрес обратно клиенту
```

## 📊 Типы DNS записей

### A Record (Address)
```
www.contoso.com → 192.168.1.100
```
Преобразование имени в IPv4 адрес

### AAAA Record
```
www.contoso.com → 2001:0db8:85a3::8a2e:0370:7334
```
Преобразование имени в IPv6 адрес

### CNAME (Canonical Name)
```
www.contoso.com → webserver.contoso.com
blog.contoso.com → webserver.contoso.com
```
Алиас одного имени на другое (не на IP!)

### MX (Mail Exchange)
```
contoso.com → mail.contoso.com (Priority: 10)
contoso.com → backup-mail.contoso.com (Priority: 20)
```
Почтовые серверы для домена

### TXT Record
```
contoso.com → "v=spf1 include:_spf.google.com ~all"
```
Текстовая информация (SPF, DKIM, verification)

### PTR (Pointer) - Reverse DNS
```
100.1.168.192.in-addr.arpa → www.contoso.com
```
Обратное преобразование IP в имя

### NS (Name Server)
```
contoso.com → ns1.contoso.com
contoso.com → ns2.contoso.com
```
Авторитативные DNS серверы для домена

### SRV (Service)
```
_ldap._tcp.contoso.com → dc01.contoso.com:389
```
Location of services (критично для AD!)

### SOA (Start of Authority)
```
contoso.com → Primary NS, Admin email, Serial, Refresh, Retry, Expire, TTL
```
Основная информация о зоне

## 📊 Уровни владения

### [[Junior SysAdmin]] уровень

#### Базовые концепции

**DNS на Windows Server:**
```powershell
# Установка DNS роли
Install-WindowsFeature -Name DNS -IncludeManagementTools

# Создание Forward Lookup Zone
Add-DnsServerPrimaryZone -Name "contoso.local" -ReplicationScope "Forest"

# Создание A записи
Add-DnsServerResourceRecordA -Name "www" `
    -ZoneName "contoso.local" `
    -IPv4Address "192.168.1.100"

# Создание CNAME
Add-DnsServerResourceRecordCName -Name "blog" `
    -ZoneName "contoso.local" `
    -HostNameAlias "www.contoso.local"

# Создание MX
Add-DnsServerResourceRecordMX -Name "." `
    -ZoneName "contoso.local" `
    -MailExchange "mail.contoso.local" `
    -Preference 10
```

**Просмотр DNS настроек (клиент):**
```powershell
# Windows
ipconfig /all
ipconfig /displaydns  # Посмотреть cache
ipconfig /flushdns    # Очистить cache

# Linux
cat /etc/resolv.conf
resolvectl status     # systemd-resolved
```

#### DNS Troubleshooting базовый

**nslookup:**
```cmd
# Простой запрос
nslookup www.google.com

# Запрос к конкретному DNS серверу
nslookup www.google.com 8.8.8.8

# Интерактивный режим
nslookup
> set type=MX
> gmail.com
> set type=A
> www.google.com
> exit
```

**dig (Linux):**
```bash
# Простой запрос
dig www.google.com

# Запрос определенного типа
dig MX gmail.com
dig AAAA www.google.com

# Trace full resolution
dig +trace www.google.com

# Reverse lookup
dig -x 8.8.8.8

# Short answer
dig +short www.google.com
```

**host (Linux):**
```bash
host www.google.com
host -t MX gmail.com
```

### [[Middle SysAdmin]] уровень

#### Windows DNS Server (продвинутый)

**Reverse Lookup Zones:**
```powershell
# Создать Reverse Zone для 192.168.1.0/24
Add-DnsServerPrimaryZone -NetworkId "192.168.1.0/24" -ReplicationScope "Forest"

# Добавить PTR запись
Add-DnsServerResourceRecordPtr -Name "100" `
    -ZoneName "1.168.192.in-addr.arpa" `
    -PtrDomainName "www.contoso.local"
```

**Conditional Forwarders:**
```powershell
# Forward запросы для определенного домена
Add-DnsServerConditionalForwarderZone -Name "partner.com" `
    -MasterServers "10.0.0.1"
```

**Forwarders (глобальные):**
```powershell
# Установить forwarders (Google DNS)
Set-DnsServerForwarder -IPAddress "8.8.8.8","8.8.4.4"
```

**DNS Scavenging (автоочистка старых записей):**
```powershell
# Включить Scavenging на сервере
Set-DnsServerScavenging -ScavengingState $true `
    -RefreshInterval "7.00:00:00" `
    -NoRefreshInterval "7.00:00:00" `
    -ScavengingInterval "7.00:00:00"

# Включить на зоне
Set-DnsServerZoneAging -Name "contoso.local" -Aging $true
```

**Round Robin DNS (простая балансировка):**
```powershell
# Несколько A записей с одним именем
Add-DnsServerResourceRecordA -Name "www" -ZoneName "contoso.local" -IPv4Address "192.168.1.100"
Add-DnsServerResourceRecordA -Name "www" -ZoneName "contoso.local" -IPv4Address "192.168.1.101"
Add-DnsServerResourceRecordA -Name "www" -ZoneName "contoso.local" -IPv4Address "192.168.1.102"

# DNS будет отвечать в циклическом порядке
```

**DNS Policies (Windows Server 2016+):**
```powershell
# Разные ответы в зависимости от subnet
Add-DnsServerQueryResolutionPolicy -Name "Moscow-Policy" `
    -Action ALLOW `
    -ClientSubnet "eq,192.168.1.0/24" `
    -ZoneName "contoso.local" `
    -ZoneScope "Moscow"

# Split-brain DNS (разные ответы internal vs external)
```

#### BIND DNS (Linux)

**Установка (Ubuntu):**
```bash
apt install bind9 bind9utils bind9-doc
```

**Конфигурация (/etc/bind/named.conf.local):**
```
zone "contoso.local" {
    type master;
    file "/etc/bind/zones/db.contoso.local";
};

zone "1.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/zones/db.192.168.1";
};
```

**Zone file (/etc/bind/zones/db.contoso.local):**
```
$TTL    604800
@       IN      SOA     ns1.contoso.local. admin.contoso.local. (
                        2024011201 ; Serial
                        604800     ; Refresh
                        86400      ; Retry
                        2419200    ; Expire
                        604800 )   ; Negative Cache TTL
;
@       IN      NS      ns1.contoso.local.
@       IN      A       192.168.1.10
ns1     IN      A       192.168.1.10
www     IN      A       192.168.1.100
mail    IN      A       192.168.1.50
@       IN      MX  10  mail.contoso.local.
```

**Проверка конфигурации:**
```bash
named-checkconf
named-checkzone contoso.local /etc/bind/zones/db.contoso.local

systemctl restart bind9
systemctl status bind9
```

### [[Senior SysAdmin]] уровень

#### DNS Security (DNSSEC)

**DNSSEC** - цифровая подпись DNS ответов:
```powershell
# Windows DNS - включить DNSSEC на зоне
Add-DnsServerSigningKey -ZoneName "contoso.com" `
    -Type KeySigningKey `
    -CryptoAlgorithm RsaSha256

Add-DnsServerSigningKey -ZoneName "contoso.com" `
    -Type ZoneSigningKey `
    -CryptoAlgorithm RsaSha256

# Подписать зону
Invoke-DnsServerZoneSigning -ZoneName "contoso.com" -Sign
```

**Проверка DNSSEC:**
```bash
dig +dnssec www.cloudflare.com
# Должен быть RRSIG record
```

#### DNS Load Balancing

**GeoDNS** - разные ответы по географии:
- Используй: Azure Traffic Manager, Route 53 (AWS), Cloudflare Load Balancing

**Anycast DNS** - один IP, множество серверов:
- BGP объявление одного IP из разных локаций
- Запрос идет к ближайшему серверу

#### Split-Horizon DNS (Split-Brain)

**Внутренние vs внешние клиенты видят разное:**
```
Internal: www.contoso.com → 192.168.1.100 (internal IP)
External: www.contoso.com → 203.0.113.100 (public IP)
```

**Реализация:**
- Разные DNS зоны (internal/external)
- DNS Views в BIND
- DNS Policies в Windows Server 2016+

#### Monitoring и Performance

**Мониторинг:**
```powershell
# Статистика Windows DNS
Get-DnsServerStatistics

# Queries per second
(Get-Counter '\DNS\Total Query Received/sec').CounterSamples

# Cache hit rate
Get-DnsServerStatistics | Select-Object -ExpandProperty CacheStatistics
```

**Performance tuning:**
- Cache size optimization
- Forwarders vs root hints
- TTL tuning
- Multiple DNS servers (primary/secondary)

#### DNS в Active Directory

**AD-Integrated Zones:**
- Хранятся в AD (не в файлах)
- Автоматическая репликация
- Secure dynamic updates
- Multi-master (любой DC может обновлять)

**SRV Records для AD:**
```
_ldap._tcp.dc._msdcs.contoso.local → DC01.contoso.local:389
_kerberos._tcp.contoso.local → DC01.contoso.local:88
_gc._tcp.contoso.local → DC01.contoso.local:3268
```

**Troubleshooting AD DNS:**
```powershell
# Проверить DNS регистрации AD
nltest /dsgetdc:contoso.local

# Перерегистрировать DNS записи
ipconfig /registerdns

# На DC:
net stop netlogon
net start netlogon
```

## 🔗 Связанные технологии

- [[Active Directory]] - полностью зависит от DNS
- [[DHCP]] - часто интегрирован с DNS для dynamic updates
- [[Основы сетей]] - понимание UDP/TCP портов (53)
- [[Windows Server]] или [[Linux]] - платформы для DNS

## 📚 Ресурсы

- RFC 1034, 1035 - DNS спецификация
- "DNS and BIND" by Cricket Liu - классика
- PowerDNS documentation
- Microsoft DNS documentation

## 💡 Best Practices

1. **Минимум 2 DNS сервера** - redundancy
2. **Используй AD-Integrated зоны** в Windows окружении
3. **Настрой Reverse DNS** - многие сервисы проверяют
4. **TTL по назначению** - короткий для часто меняющихся, длинный для стабильных
5. **Мониторь DNS** - critical service
6. **Scavenging** - для dynamic DNS
7. **DNSSEC** - для публичных зон (защита от подделки)
8. **Backup зон** - регулярно

## 🚨 Типичные ошибки

1. **Один DNS сервер** - SPOF
2. **DNS сервер не может резолвить сам себя** - циклическая зависимость
3. **Неправильные Forwarders** - могут замедлить всё
4. **Игнорирование TTL** - изменения применяются не сразу
5. **Нет Reverse DNS** - проблемы с почтой и некоторыми сервисами
6. **DC не является DNS** в AD окружении - плохая практика
7. **Использование ISP DNS на DC** - должен использовать себя или другой DC

---

**Связанные уровни:**
- 🎓 [[Junior SysAdmin]] - базовое понимание и troubleshooting
- 📈 [[Middle SysAdmin]] - администрирование DNS серверов
- 🚀 [[Senior SysAdmin]] - проектирование DNS инфраструктуры

#технология #dns #сети #критический-навык
