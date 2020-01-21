## Update IdM
Yum для обновления пакетов Identity Management и системы:
```
yum update ipa-*
```
* После обновления пакетов Identity Management требуется подождать хотя бы минут 10, пока все остальные серверы в топологии получают обновленную схему, даже если вы не обновляете их пакеты. Это гарантирует, что любые новые записи, которые используют новую схему, могут быть реплицированы среди других серверов.
* Downgrading пакетов Identity Management не поддерживается.
* Red Hat рекомендует обновиться только до следующей минорной версии. Например, обновить Identity Management до Red Hat Enterprise Linux 7.3,после обновить уже до Red Hat Enterprise Linux 7.4. Обновление с более ранних версий может вызвать проблемы.