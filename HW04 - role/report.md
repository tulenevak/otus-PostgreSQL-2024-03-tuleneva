**Отчет ДЗ №4 Работа с базами данных, пользователями и правами**  
  
	Требуется:  
	- создание новой базы данных, схемы и таблицы,  
	- создание роли для чтения данных из созданной схемы созданной базы данных,  
	- создание роли для чтения и записи из созданной схемы созданной базы данных.  

1. Создала БД *testdb*, зашла в эту БД под пользователем *postgres*, создала схему *testnm*    
  	
	![рис.1](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW04%20-%20role/image/im1.jpg)  
  
1. Создала таблицу *t1* с одной колонкой *c1* типа *integer*, добавила строку со значением *c1=1*.  
	Создала новую роль *readonly*.  
  
	![рис.2](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW04%20-%20role/image/im2.jpg)  
  	
1. Дала новой роли право на подключение к базе данных *testdb*.  
	Дала новой роли право на использование схемы  *testnm*.  
	Дала новой роли право на select для всех таблиц схемы *testnm*.  
	Создала пользователя testread с паролем *test123*.  
	Дала роль *readonly* пользователю *testread*.  
  	
	Команды:  
	*GRANT connect on DATABASE testdb TO readonly;*  
	*GRANT usage on SCHEMA testnm to readonly;*  
	*GRANT SELECT on all TABLEs in SCHEMA testnm TO readonly;*  
	*CREATE USER testread with password 'test123';*  
	*GRANT readonly TO testread;*  
  
	![рис.3](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW04%20-%20role/image/im3.jpg)  
  	
1. Попыталась зайти под пользователем *testread* в базу данных *testdb*  
  
	![рис.4](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW04%20-%20role/image/im4.jpg)  
  	
	Получила ошибку (потому что необходимо указывать, что работаем не через *Peer authentication* пользователя линукс, а через пароль).  
	Прописала четкое подключение.    
	*psql -h 10.7.0.141 -U testread -d testdb –W*   
  
	![рис.5](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW04%20-%20role/image/im5.jpg)  
  
1. Пытаюсь посмотреть содержимое созданной таблицы  
	*select * from t1;*  
	Не сработало, т.к. таблица создана в схеме *public* а не testnm и прав на *public* для роли *readonly* не давали.  
	Проверяю *search_path*.  
  
	![рис.6](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW04%20-%20role/image/im6.jpg)  
  
	В *search_path* вижу *"$user", public*. При том что схемы *$USER* нет, то таблица по умолчанию создалась в *public*.
  
1. Возвращаюсь в БД *testdb* под пользователем *postgres*.  
	Удаляю таблицу *t1*.  
	Создаю ее заново, но уже с явным указанием имени схемы *testnm*.  
  	
	![рис.7](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW04%20-%20role/image/im7.jpg)    
  	
1. Добавляю строку со значением *c1=1*.  
	Захожу под пользователем *testread* в базу данных *testdb*.  
	Пытаюсь посмотреть содержимое *select * from testnm.t1;*  
  
	![рис.8](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW04%20-%20role/image/im8.jpg)  
  
	Получаю ошибку  
	(потому что *grant SELECT on all TABLEs in SCHEMA testnm TO readonly*  
	дал доступ только для существующих на тот момент времени таблиц, а *t1* пересоздавалась).  
	Меняю грант  
	*ALTER default privileges in SCHEMA testnm grant SELECT on TABLES to readonly;*  
  
	![рис.9](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW04%20-%20role/image/im9.jpg)  
  
1. После этого запрос к таблице опять не дал результата. 
	Возвращаюсь и снова даю грант на все таблицы.  
	Теперь запрос выдает нужный результат.  
  
	![рис.10](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW04%20-%20role/image/im10.jpg)  
     
1. Создаю таблицу *t2*. Получается создать, хотя прав на создание и вставку данных не давали.  
 
	![рис.11](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW04%20-%20role/image/im11.jpg)  
  
	Это произошло потому, что *search_path* указывает в первую очередь на схему *public*.  
	А схема *public* создается в каждой БД по умолчанию.  
	И *grant* на все действия в этой схеме дается роли *public*.   
	А роль *public* добавляется всем новым пользователям.   
	Соответсвенно каждый пользователь может по умолчанию создавать объекты в схеме *public* любой БД,   
	если у него есть право на подключение к этой БД.   
  	
	Чтобы забыть про роль *public*,выполняю *REVOKE*.   
  
	![рис.12](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW04%20-%20role/image/im12.jpg)  	
  
1. Далее проверяю, что создать таблицу невозможно:  
	*\c testdb testread;*  
	*create table t2(c1 integer);*  
  
	![рис.13](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW04%20-%20role/image/im13.jpg) 
  
	Со всем справиться самостоятельно не получилось.   
	Пришлось подглядеть в подсказки, тема сложная.   
	Но в итоге всё понятно, спасибо!  
  