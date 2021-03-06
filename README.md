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


