# FreeIPA
## Немного теории:
С IdM: ИТ-администратор может:
* Храните идентификационные данные в одном центральном месте: сервер IdM
* Одновременное применение политик к нескольким машинам
* Установите различные уровни доступа для пользователей, используя управление доступом на основе хоста, делегирование и другие правила
* Централизованное управление правилами привилегий
* Монтирование домашних каталогов
  
Большинство сервисов не должны быть установлены на сервере IdM. Например, такие службы, как центр сертификации (CA), сервер DNS или сервер протокола сетевого времени (NTP), могут быть установлены на другом сервере за пределами домена IdM.
Клиент IdM не требует, чтобы выделенное клиентское программное обеспечение взаимодействовало как часть домена. Требуется только правильная конфигурация системы определенных служб и библиотек, таких как Kerberos или DNS. Эта конфигурация указывает клиентскому компьютеру использовать службы IdM.
Системный демон служб безопасности (SSSD) - это клиентское приложение для кэширования учетных данных. Рекомендуется использовать SSSD на клиентских компьютерах, поскольку это упрощает требуемую настройку клиента. SSSD также предоставляет дополнительные функции, например:
* Проверка подлинности клиента в автономном режиме обеспечивается кэшированием учетных данных из централизованных хранилищ удостоверений и аутентификации находящиеся локально на клиенте
* Улучшена согласованность процесса аутентификации, поскольку нет необходимости поддерживать как центральную учетную запись, так и локальную учетную запись пользователя для автономной аутентификации
* Интеграция с другими сервисами, такими как sudo
* Авторизация на основе хост-контроля доступа (HBAC)
  
Из соображений производительности и стабильности Red Hat рекомендует не устанавливать другие приложения или службы на серверы IdM. Например, серверы IdM могут быть исчерпывающими для системы, особенно если число объектов LDAP велико. Кроме того, IdM интегрирован в систему, и, если сторонние приложения изменяют конфигурационные файлы, от которых зависит IdM, IdM может сломаться.
  
При установке сервера IdM системные файлы перезаписываются для настройки домена IdM. IdM создает резервные копии исходных системных файлов в /var/lib/ipa/sysrestore/.
### Требования
Оперативная память - это самая важная аппаратная функция для правильного размера Чтобы определить, сколько оперативной памяти вам требуется, рассмотрите следующие рекомендации:
* Для 10 000 пользователей и 100 групп: не менее 3 ГБ ОЗУ и 1 ГБ подкачки
* Для 100 000 пользователей и 50 000 групп: не менее 16 ГБ ОЗУ и 4 ГБ пространства подкачки
## Подготовка
* Red Hat рекомендует отключить NSCD на компьютерах Identity Management. В качестве альтернативы, если отключение NSCD невозможно, включайте NSCD только для карт, которые SSSD не кэширует:
```Change the /etc/nscd.conf file:
enable-cache hosts yes
enable-cache passwd no
enable-cache group no
enable-cache netgroup no
enable-cache services no 
```
* IPv6 Должен быть включен в системе
* Полное доменное имя должно быть действительным DNS-именем, что означает, что допускаются только цифры, буквенные символы и дефисы (-). Другие символы, такие как подчеркивание, в имени хоста вызывают сбои DNS. Кроме того, имя хоста должно быть в нижнем регистре; заглавные буквы не допускаются. Полное доменное имя не должно преобразовываться в адрес обратной связи. Он должен соответствовать общедоступному IP-адресу машины, а не 127.0.0.1. К примеру /etc/hosts:
```
127.0.0.1	localhost.localdomain	localhost
::1		localhost6.localdomain6	localhost6
192.0.2.1	server.example.com	server
2001:DB8::1111	server.example.com	server
```
* Список Портов для окрытия в firewalld:
Service	    Ports	                Protocol
HTTP/HTTPS	80, 443	              TCP
LDAP/LDAPS	389, 636	            TCP
Kerberos	  88, 464	              TCP and UDP
DNS	        53	                  TCP and UDP
NTP	        123	                  UDP
* Список сервисов для окрытия в firewalld
Table 2.2. firewalld Services
Service name	For details, see:
freeipa-ldap	/usr/lib/firewalld/services/freeipa-ldap.xml
freeipa-ldaps	/usr/lib/firewalld/services/freeipa-ldaps.xml
dns	          /usr/lib/firewalld/services/dns.xml
  
!!! Red Hat настоятельно рекомендует использовать DNS, интегрированный с IdM, для базового использования в развертывании IdM: когда сервер IdM управляет DNS, существует тесная интеграция между DNS и собственными инструментами IdM, что позволяет автоматизировать некоторые функции управления записями DNS. При использовании встроенного DNS-сервера большая часть ведения DNS-записи автоматизируется. Необходимо только: настроить правильное делегирование из родительского домена на серверы IdM. При использовании внешнего DNS-сервера необходимо: вручную создать новый домен на DNS-сервере, вручную заполнить новый домен записями из файла зоны, обновлять записи вручную после установки или удаления реплики, а также после любых изменений в конфигурации службы IdM, например после настройки доверия Active Directory
### Установка Freeipa
* Открываем Порты:
```
firewall-cmd --permanent --add-port={53,80,88,135,138,139,389,443,445,464,636,1024-1300,3268}/tcp
firewall-cmd --permanent --add-port={53,88,123,138,139,389,445,464}/udp
firewall-cmd --reload
```
* Приводим /etc/resolv.conf к виду:
```
search example.com
nameserver 192.0.2.1
```
* Устанавливаем нужные пакеты 
```
yum -y install bind bind-utils bind-dyndb-ldap ipa-server ipa-server-dns
```
* Перед началом установки, требуется попроваить конфигурационный файл DNS сервера, /etc/named.conf:
```
listen-on port 53 { any; }
//listen-on-v6 port 53 { ::1; };
allow-query { any; }
dnssec-validation no;
```
* Запускаем DNS сервер
```
systemctl start named
```
* Настройка IdM сервера 
sudo ipa-server-install --setup-dns --ssh-trust-dns --mkhomedir --allow-zone-overlap
  
Отвечаем на вопросы. как в примере:
```
...
Server host name [idm-server.example.com]: <Enter>
...
Please confirm the domain name [example.com]: <Enter>
...
Please provide a realm name [EXAMPLE.COM]: <Enter>
... Directory Manager password: < "password" >
Password (confirm): < "password" >
...
IPA admin password: < "password" >
Password (confirm): < "password" >
...
Do you want to configure DNS forwarders? [yes]: Yes
...
Enter IP address for a DNS forwarder: <Enter>
...
Do you want to configure the reverse zone? [yes]: Yes
...
Continue to configure the system with these values? [no]: Yes
...
Domain name: example.com
```
  
В конце должно выйти сообщение:
```
The IPA Master Server will be configured with:
Hostname: ipa.example.com
IP address(es): 192.0.2.1
Domain name: example.com
Realm name: EXAMPLE.COM
BIND DNS server will be configured to serve IPA domain with:
Forwarders: No forwarders
Forward policy: only
Reverse zone(s): 0.25.172.in-addr.arpa.
Continue to configure the system with these values? [no]: Yes
...
...
The ipa-client-install command was successful
==============================================================================
Setup complete
Next steps:
 1. You must make sure these network ports are open:
 TCP Ports:
 * 80, 443: HTTP/HTTPS
 * 389, 636: LDAP/LDAPS
 * 88, 464: kerberos
 * 53: bind
 UDP Ports:
 * 88, 464: kerberos
 * 53: bind
 * 123: ntp
 2. You can now obtain a kerberos ticket using the command: 'kinit admin'
 This ticket will allow you to use the IPA tools (e.g., ipa user-add)
 and the web user interface.
Be sure to back up the CA certificates stored in /root/cacert.p12
These files are required to create replicas. The password for these
files is the Directory Manager password
```
* Приводим /etc/resolv.conf к виду:
```
search example.com
nameserver 127.0.0.1
```
  
Далее проверяем работу служб:
```
[ipa ~]$ sudo ipactl status
Directory Service: RUNNING
krb5kdc Service: RUNNING
kadmin Service: RUNNING
named Service: RUNNING
httpd Service: RUNNING
ipa-custodia Service: RUNNING
ntpd Service: RUNNING
pki-tomcatd Service: RUNNING
ipa-otpd Service: RUNNING
ipa-dnskeysyncd Service: RUNNING
ipa: INFO: The ipactl command was successful
```
  
Проверим, что PTR записи будут создаваться\обновляться в обратной зоне днс 2.0.192.in-addr.apra
```
[ipa ~]$ kinit admin
[ipa ~]$ ipa dnszone-mod example.com --allow-sync-ptr=true
 Zone name: example.com.
 Active zone: TRUE
 Authoritative nameserver: ipa.example.com.
 Administrator e-mail address: hostmaster.example.com.
 SOA serial: 1536572066
 SOA refresh: 3600
 SOA retry: 900
 SOA expire: 1209600
 SOA minimum: 3600
 Allow query: any;
 Allow transfer: none;
 Allow PTR sync: TRUE
```
  
Копия всех сообщений пишется в файл журнала **/var/log/ipaserver-install.log**
  
Проверить какие билеты есть на данный момент:
```
[ipa ~]$ kinit admin
Password for admin@EXAMPLE.COM:
[ipa ~]$ klist
Ticket cache: KEYRING:persistent:1000:1000
Default principal: admin@EXAMPLE.COM
Valid starting Expires Service principal
09/06/2018 04:46:51 09/07/2018 04:46:48 krbtgt/EXAMPLE.COM@EXAMPLE.COM
```
  
Проверка настроек по умолчанию:
```
[ipa ~]$ ipa config-show
 Maximum username length: 32
 Home directory base: /home
 Default shell: /bin/sh
 Default users group: ipausers
 Default e-mail domain: example.com
 Search time limit: 2
 Search size limit: 100
 User search fields: uid,givenname,sn,telephonenumber,ou,title
 Group search fields: cn,description
 Enable migration mode: FALSE
 Certificate Subject base: O=EXAMPLE.COM
 Password Expiration Notification (days): 4
 Password plugin features: AllowNThash, KDC:Disable Last Success
 SELinux user map order: guest_u:s0$xguest_u:s0$user_u:s0$staff_u:s0-
s0:c0.c1023$unconfined_u:s0-s0:c0.c1023
 Default SELinux user: unconfined_u:s0-s0:c0.c1023
 Default PAC types: MS-PAC, nfs:NONE
 IPA masters: ipa.example.com
 IPA CA servers: ipa.example.com
 IPA NTP servers: ipa.example.com
 IPA CA renewal master: ipa.example.com
 IPA master capable of PKINIT: ipa.example.com
 ```
   
Поменяем оболучку с sh на bash
```
[ipa ~]$ ipa config-mod --defaultshell=/bin/bash
...
 Default shell: /bin/bash
...
```
  
Проверка файла зон, днс записей:
```
[ipa ~]$ ipa dnszone-find
 Zone name: 2.0.192.in-addr.arpa.
 Active zone: TRUE
 Authoritative nameserver: ipa.example.com.
 Administrator e-mail address: hostmaster.example.com.
 SOA serial: 1536149776
 SOA refresh: 3600
 SOA retry: 900
 SOA expire: 1209600
 SOA minimum: 3600
 Allow query: any;
 Allow transfer: none;
 Zone name: example.com.
 Active zone: TRUE
 Authoritative nameserver: ipa.example.com.
 Administrator e-mail address: hostmaster.example.com.
 SOA serial: 1536149809
 SOA refresh: 3600
 SOA retry: 900
 SOA expire: 1209600
 SOA minimum: 3600
 Allow query: any;
 Allow transfer: none;
----------------------------
Number of entries returned 2
----------------------------
[vagrant@ipa ~]$ ipa dnsrecord-find example.com --name=ipa --all
 dn: idnsname=ipa,idnsname=example.com.,cn=dns,dc=example,dc=com
 Record name: ipa
 Time to live: 1200
 A record: 192.0.2.1
  SSHFP record: 3 1 B89202D696F5480CB33A41917FBC13D440395E57, 3 2 FDDFCB234DD2AC54444A710080D6BDA5BA1220D6A2EE748FA13C2DD4 598A04E3, 1 1 0DED058F79EDD76ABE1B77B107E15C7A005A6DA9, 1 2
                C2EADF4B1CF24D0C1AB38543A86D6EDC2BE5CE365C94A3AC8AA34576 D9F0CB9B, 4 1 AA5BA74EC06A56FA8D8CD186D4A5B9A9C3FAFCD3, 4 2 5881381C19220471B2C008CD39F64D6A392CA2A25806A55D8566832D 839201D9
  objectclass: top, idnsrecord
----------------------------
Number of entries returned 1
----------------------------
```
Файлы журналов:
```
/var/log/pki-ca/debug
/var/log/pki-ca-install.log #Журнал установки Центра Сертификации DogTag CA
/var/log/dirsrv/ #Каталог журналов, куда попадают сообщения службы каталога
/var/log/messages
```
