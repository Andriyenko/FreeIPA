## Logs
Рекомендуется использовать централизованные логи на основе докер образа: 
* [rsyslog, elasticsearch and kibana](https://github.com/pschiffe/rsyslog-elasticsearch-kibana)
  
Скрипт для клиентов IdM:
* [ipa-log-config](https://github.com/pschiffe/ipa-log-config)
  
FreeIPA сервер:
* /var/log/httpd/error_log: FreeIPA API логи + Apache errors
* /var/log/krb5kdc.log: FreeIPA KDC
* /var/log/dirsrv/slapd-$REALM/access: Directory Server инициализация
* /var/log/dirsrv/slapd-$REALM/errors: Directory Server errors (включая ошибки репликации)
* /var/log/pki/pki-tomcat/ca/transactions: FreeIPA PKI transactions/logs
* /var/log/pki-ca/debug
* /var/log/pki-ca-install.log #Журнал установки Центра Сертификации DogTag CA
* /var/log/messages
  
FreeIPA client logs:
* /var/log/sssd/*.log: SSSD logs
* /var/log/audit/audit.log: user login attempts
* /var/log/secure: reasons why user login failed
