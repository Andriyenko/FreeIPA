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
