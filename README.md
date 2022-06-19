## Домашнее задание к занятию "08.02 Работа с Playbook"


9. Подготовьте README.md файл по своему playbook. В нём должно быть описано: что делает playbook, какие у него есть параметры и теги.   
playbook содержит 2 play: 1) устанавливает clickhouse 2) устанавливает vector
1 play выполняется на группе хостов clickhouse из файла из папки inventory, содержит таски :  
`Get clickhouse distrib`: скачивает deb пакеты clickhouse в цикле. `clickhouse_packages` и `clickhouse_version` берет из файла group_vars/clickhouse/vars.yaml  
При возникновении ошибки скачивания пакета `clickhouse-common-static` запускается rescue таска `Get clickhouse distrib`,  которая скачивает пакет по альтернативному пути 
`Install clickhouse packages`: В этой таске производится установка скачанных пакетов с помощью `ansible.builtin.apt` и цикла  
После установки пакетов вызывается handler на перезапуск сервиса `Start clickhouse service`  
Далее запускается команда на создание базы данных `logs` в `ansible.builtin.command`  
Результат команды записывается в переменную `register: create_db`  
Далее производится обработка ошибок
```
failed_when: create_db.rc != 0 and create_db.rc !=82
      changed_when: create_db.rc == 0
```
То есть ошибка будет считаться ошибкой, если выполнилось условие create_db.rc != 0 and create_db.rc !=82
считается, что изменения произведены, когда выполняется условие create_db.rc == 0


2 play выполняется на группе хостов clickhouse: 
таска `get vector distr` скачивает deb пакет vector  
таска `Install vector package` устанавливает пакет vector  
Далее вызывается handler `restart vector` на перезапуск сервиса vector
