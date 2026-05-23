# Домашнее задание к занятию «Репликация и масштабирование. Часть 1» - Викторов Михаил

### Задание 1

На лекции рассматривались режимы репликации master-slave, master-master, опишите их различия.

*Ответить в свободной форме.*

---

### Задание 2

Выполните конфигурацию master-slave репликации, примером можно пользоваться из лекции.

*Приложите скриншоты конфигурации, выполнения работы: состояния и режимы работы серверов.*

---

## Дополнительные задания (со звёздочкой*)
Эти задания дополнительные, то есть не обязательные к выполнению, и никак не повлияют на получение вами зачёта по этому домашнему заданию. Вы можете их выполнить, если хотите глубже шире разобраться в материале.

---

### Задание 3* 

Выполните конфигурацию master-master репликации. Произведите проверку.

*Приложите скриншоты конфигурации, выполнения работы: состояния и режимы работы серверов.*

## Ответы
## Задание 1

Режимы репликации Master-Slave и Master-Master — распространённые архитектуры настройки репликации баз данных Вот их основные различия:

### Master-Slave
Один мастер, на который выполняются все операции записи (INSERT, UPDATE, DELETE).
Один или несколько слейвов, которые получают копии данных с мастера и служат для чтения.
Репликация однонаправленная: изменения с мастера передаются на слейвы.
Если мастер падает, можно вручную или автоматически переключить один из слейвов на роль мастера, но это требует дополнительной настройки.


### Master-Master
Два или более сервера, каждый из которых является мастером и может принимать записи.
Репликация двунаправленная: изменения с одного мастера передаются на другой и наоборот.
Позволяет распределять нагрузку на запись между серверами.

### Пример различий в работе:

### В Master-Slave:
Сервер A — мастер (все записи), Сервер B — слейв (только чтение).
Если A упал — B не может принимать записи, пока не станет мастером.

### В Master-Master:
Сервер A и Сервер B — оба принимают записи.
Если A упал — B продолжает работать. Когда A восстановится — синхронизируется с B.

## Задание 2
Поднимаем в Докере две базы:

<img width="1379" height="175" alt="image" src="https://github.com/user-attachments/assets/ab9c9bab-a644-47b3-bdee-3634a1cb364b" />

Настраиваем пользователя репликации, подключаемся к мастер-базе:
```
docker exec -it mysql-master mysql -u root -p
```
Создаем пользователя для репликации и смотрим статус мастера:
```
  CREATE USER 'repl'@'%' IDENTIFIED BY 'replicapassword';
  GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%';
  FLUSH PRIVILEGES;

  SHOW BINARY LOG STATUS;
```
<img width="990" height="172" alt="image" src="https://github.com/user-attachments/assets/46694981-9677-4d41-bf13-4c859151e8fc" />

Настраиваем слейв:
```
docker exec -it mysql-slave mysql -u root -p
```
Подключаем к мастеру:

```
  CHANGE REPLICATION SOURCE TO
    SOURCE_HOST='mysql-master',
    SOURCE_USER='repl',
    SOURCE_PASSWORD='replicapassword',
    SOURCE_LOG_FILE='mysql-bin.000003',
    SOURCE_LOG_POS=897,
    GET_SOURCE_PUBLIC_KEY=1;
```
Вы полняем команды старта слейва и проверки состояния:

```
  START REPLICA;
  SHOW REPLICA STATUS\G
```
<img width="599" height="199" alt="image" src="https://github.com/user-attachments/assets/762a4bc4-d257-4a39-a1f9-47ddadc944bd" />

Детальный вывод:

```
*************************** 1. row ***************************
             Replica_IO_State: Waiting for source to send event
                  Source_Host: mysql-master
                  Source_User: repl
                  Source_Port: 3306
                Connect_Retry: 60
              Source_Log_File: mysql-bin.000003
          Read_Source_Log_Pos: 897
               Relay_Log_File: relay-log.000002
                Relay_Log_Pos: 328
        Relay_Source_Log_File: mysql-bin.000003
           Replica_IO_Running: Yes
          Replica_SQL_Running: Yes
              Replicate_Do_DB:
          Replicate_Ignore_DB:
           Replicate_Do_Table:
       Replicate_Ignore_Table:
      Replicate_Wild_Do_Table:
  Replicate_Wild_Ignore_Table:
                   Last_Errno: 0
                   Last_Error:
                 Skip_Counter: 0
          Exec_Source_Log_Pos: 897
              Relay_Log_Space: 533
              Until_Condition: None
               Until_Log_File:
                Until_Log_Pos: 0
           Source_SSL_Allowed: No
           Source_SSL_CA_File:
           Source_SSL_CA_Path:
              Source_SSL_Cert:
            Source_SSL_Cipher:
               Source_SSL_Key:
        Seconds_Behind_Source: 0
Source_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error:
               Last_SQL_Errno: 0
               Last_SQL_Error:
  Replicate_Ignore_Server_Ids:
             Source_Server_Id: 1
                  Source_UUID: 470616e1-56f2-11f1-872d-c2c8a3dc3418
             Source_Info_File: mysql.slave_master_info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
    Replica_SQL_Running_State: Replica has read all relay log; waiting for more updates
           Source_Retry_Count: 10
                  Source_Bind:
      Last_IO_Error_Timestamp:
     Last_SQL_Error_Timestamp:
               Source_SSL_Crl:
           Source_SSL_Crlpath:
           Retrieved_Gtid_Set:
            Executed_Gtid_Set: 47085e6c-56f2-11f1-9d46-02e5bdeb8317:1-5
                Auto_Position: 0
         Replicate_Rewrite_DB:
                 Channel_Name:
           Source_TLS_Version:
       Source_public_key_path:
        Get_Source_public_key: 1
            Network_Namespace:
1 row in set (0.01 sec)
```

Проверяем работоспособность, заходим на мастер и пишем бойлерплейт в базу viktorov:

```
  USE viktorov;
  CREATE TABLE users (id INT AUTO_INCREMENT PRIMARY KEY, name VARCHAR(50));
  INSERT INTO users (name) VALUES ('Alice'), ('Bob');
```
<img width="733" height="423" alt="image" src="https://github.com/user-attachments/assets/aca0d9fc-23e2-4d75-b8fc-4391e121a316" />

Заходим на слейв, смотрим что там:

<img width="682" height="290" alt="image" src="https://github.com/user-attachments/assets/f3b3c0bb-500c-42c2-99b1-700dc59fe2d7" />





