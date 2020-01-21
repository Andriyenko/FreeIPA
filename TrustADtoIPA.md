## Настройка доверительных отношений между доменами AD и IPA
* Домен: ad.domain.com IdM - example.com
  
Устанавливаем доп. пакеты:
```
yum install -y samba samba-client samba-winbind-clients sssd-libwbclient ipa-server-trust-ad ipa-admintools
```
Получаем билет Kerberos
```
[vagrant@ipa ~]$ ipa service-add cifs/ipa.example.com
--------------------------------------------
Added service "cifs/ipa.example.com@EXAMPLE.COM"
--------------------------------------------
  Principal name: cifs/ipa.example.com@EXAMPLE.COM
  Principal alias: cifs/ipa.example.com@EXAMPLE.COM
  Managed by: ipa.example.com
```
Формируем keytab для samba:
```
[vagrant@ipa ~]$ ipa-getkeytab -p cifs/ipa.example.com -k /etc/samba/samba.keytab -s ipa.example.com
Keytab successfully retrieved and stored in: /etc/samba/samba.keytab
```
Добавляем зону пересылки на сервере IPA (на сервере AD так же требуется добавить, [мануал](http://winintro.ru/dnsmgr.ru/html/e324865f-1cbe-42ec-bf18-a220c0e26fe6.htm#bkmk_winui))
```
[vagrant@ipa ~]$ ipa dnsforwardzone-add moroz.local --forward-policy=only --forwarder=192.0.2.2 --skip-overlap-check
Server will check DNS forwarder(s).
This may take some time, please wait ...
  Zone name: domain.com.
  Active zone: TRUE
  Zone forwarders: 192.0.2.2
  Forward policy: only
```
Меняем переменную LDAPTLS_REQCERT (ipaservice - пользователь AD для синхронизации)
```
[vagrant@ipa ~]$ LDAPTLS_REQCERT=never ldapsearch -x -Z -D 'domain\ipaservice' -h dc.domain.com -b "dc=domain,dc=com" -s sub "(objectClass=user)"
```
Настроим необходимые конфигурации на IPA сервере
```
[vagrant@ipa ~]$ ipa-adtrust-install --add-sids -U --netbios-name="EXAMPLE" --enable-compat -a "PASSWORD"

The log file for this installation can be found in /var/log/ipaserver-install.log
==============================================================================
This program will setup components needed to establish trust to AD domains for
the IPA Server.
This includes:
  * Configure Samba
  * Add trust related objects to IPA LDAP server
To accept the default shown in brackets, press the Enter key.
Configuring CIFS
  [1/25]: validate server hostname
  [2/25]: stopping smbd
  [3/25]: creating samba domain object
  [4/25]: creating samba config registry
  [5/25]: writing samba config file
  [6/25]: adding cifs Kerberos principal
  [7/25]: adding cifs and host Kerberos principals to the adtrust agents group
  [8/25]: check for cifs services defined on other replicas
  [9/25]: adding cifs principal to S4U2Proxy targets
  [10/25]: adding admin(group) SIDs
  [11/25]: adding RID bases
  [12/25]: updating Kerberos config
'dns_lookup_kdc' already set to 'true', nothing to do.
  [13/25]: activating CLDAP plugin
  [14/25]: activating sidgen task
  [15/25]: map BUILTIN\Guests to nobody group
  [16/25]: configuring smbd to start on boot
  [17/25]: adding special DNS service records
  [18/25]: enabling trusted domains support for older clients via Schema Compatibility plugin
  [19/25]: restarting Directory Server to take MS PAC and LDAP plugins changes into account
  [20/25]: adding fallback group
  [21/25]: adding Default Trust View
  [22/25]: setting SELinux booleans
  [23/25]: starting CIFS services
  [24/25]: adding SIDs to existing users and groups
This step may take considerable amount of time, please wait..
  [25/25]: restarting smbd
Done configuring CIFS.

=============================================================================
Setup complete

You must make sure these network ports are open:
        TCP Ports:
          * 135: epmap
          * 138: netbios-dgm
          * 139: netbios-ssn
          * 445: microsoft-ds
          * 1024..1300: epmap listener range
          * 3268: msft-gc
        UDP Ports:
          * 138: netbios-dgm
          * 139: netbios-ssn
          * 389: (C)LDAP
          * 445: microsoft-ds

See the ipa-adtrust-install(1) man page for more details

=============================================================================
```
НА сервере AD открываем оснаску Active Directory Домены и Доверия
* Открываем свойства зоны domain.com, далее вкладка Отношения доверия, далее кнопка Создать отношения доверия..., далее Вводим имя домена: EXAMPLE.COM, далее Доверие леса, далее Двухсторонее, далее Только для данного домена, далее Проверка подлинности в лесу, далее Задаем пароль для отношения доверия, далее В следующих окнах читаем информацию и нажимаем далее, пока не дойдем до подтверждения исходящего доверия. Отвечаем "Нет", далее Далее отвечаем "Нет" на запрос подтверждения входящего доверия, далее Готово
  
На сервере IPA устанавливаем двухсторонне доверие
```
[vagrant@ipa ~]$ ipa trust-add --type=ad domain.com --admin ipaservice --password --two-way=true
Active Directory domain administrator's password:
----------------------------------------------------
Added Active Directory trust for realm "domain.com"
----------------------------------------------------
  Realm name: domain.com
  Domain NetBIOS name: DOMAIN
  Domain Security Identifier: S-1-5-21-22533453494-2391153606-1419618105
  Trust direction: Two-way trust
  Trust type: Active Directory domain
  Trust status: Established and verified
```
Проверяем статус:
```
[vagrant@ipa ~]$ ipa trust-show domain.com
  Realm name: domain.com
  Domain NetBIOS name: DOMAIN
  Domain Security Identifier: S-1-5-21-2253914694-2391153606-1419618105
  Trust direction: Two-way trust
  Trust type: Active Directory domain
```
