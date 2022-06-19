## Домашнее задание к занятию "08.02 Работа с Playbook"

### Подготовка к выполнению
1. Создайте свой собственный (или используйте старый) публичный репозиторий на github с произвольным именем.  
создал

2.Скачайте playbook из репозитория с домашним заданием и перенесите его в свой репозиторий.  
выполнено

4. Подготовьте хосты в соответствии с группами из предподготовленного playbook.  
будет использоваться вм и на ней запускаться playbook (localhost)

### Основная часть  
1. Приготовьте свой собственный inventory файл prod.yml.  

```yml
clickhouse:
  hosts:
    localhost:
       ansible_connection: local

```
2. Допишите playbook: нужно сделать ещё один play, который устанавливает и настраивает vector.  
выполнено
3. При создании tasks рекомендую использовать модули: get_url, template, unarchive, file.  
скачал deb пакет
url: "https://packages.timber.io/vector/{{ vector_version }}/vector_{{ vector_version }}-1_amd64.deb"
4. Tasks должны: скачать нужной версии дистрибутив, выполнить распаковку в выбранную директорию, установить vector.  
установил с помощью
```
 ansible.builtin.apt:
            deb: ./vector_{{ vector_version }}-1_amd64.deb
```

5. Запустите ansible-lint site.yml и исправьте ошибки, если они есть.  
выполнено
6. Попробуйте запустить playbook на этом окружении с флагом --check.  
запустил
7. Запустите playbook на prod.yml окружении с флагом --diff. Убедитесь, что изменения на системе произведены.  

8. Повторно запустите playbook с флагом --diff и убедитесь, что playbook идемпотентен.

```
root@anton-v-m:~/netology/ansible-playbook/playbook# ansible-playbook site.yml -i inventory/prod.yml --diff

PLAY [Install Clickhouse] ***************************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************************************
ok: [localhost]

TASK [Get clickhouse distrib] ***********************************************************************************************************************************
ok: [localhost] => (item=clickhouse-client)
ok: [localhost] => (item=clickhouse-server)
failed: [localhost] (item=clickhouse-common-static) => {"ansible_loop_var": "item", "changed": false, "dest": "./clickhouse-common-static_22.3.3.44_all.deb", "elapsed": 0, "item": "clickhouse-common-static", "msg": "Request failed", "response": "HTTP Error 404: Not Found", "status_code": 404, "url": "https://packages.clickhouse.com/deb/pool/stable/clickhouse-common-static_22.3.3.44_all.deb"}

TASK [Get clickhouse distrib] ***********************************************************************************************************************************
ok: [localhost]

TASK [Install clickhouse packages] ******************************************************************************************************************************
ok: [localhost] => (item=./clickhouse-common-static_22.3.3.44_amd64.deb)
ok: [localhost] => (item=./clickhouse-client_22.3.3.44_all.deb)
ok: [localhost] => (item=./clickhouse-server_22.3.3.44_all.deb)

TASK [Create database] ******************************************************************************************************************************************
ok: [localhost]

PLAY [install Vector] *******************************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************************************
ok: [localhost]

TASK [get vector distr] *****************************************************************************************************************************************
ok: [localhost]

TASK [Install vector package] ***********************************************************************************************************************************
ok: [localhost]

PLAY RECAP ******************************************************************************************************************************************************
localhost                  : ok=7    changed=0    unreachable=0    failed=0    skipped=0    rescued=1    ignored=0   

```

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

11. Готовый playbook выложите в свой репозиторий, поставьте тег 08-ansible-02-playbook на фиксирующий коммит, в ответ предоставьте ссылку на него.  
