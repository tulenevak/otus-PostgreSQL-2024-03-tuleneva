**Отчет ДЗ №5 Настройка autovacuum с учетом особеностей производительности**  
   
	Требуется:  
	- запустить нагрузочный тест pgbench  
	- настроить параметры autovacuum  
	- проверить работу autovacuum    
  
1. Создала БД *testdb*, зашла в эту БД под пользователем *postgres*, создала схему *testnm*.  
    	
1. Запустила тест на производительность с дефолтными настройками постгре:  
   
	![рис.1](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW05%20-%20vacuum/image/im1.jpg) 
  
1.	Изменила настройки в конф.файле: *sudo nano /etc/postgresql/15/main/postgresql.conf* в соответсвиис приложенным файлом к ДЗ.
  
	*max_connections = 40  
	shared_buffers = 1GB  
	effective_cache_size = 3GB  
	maintenance_work_mem = 512MB  
	checkpoint_completion_target = 0.9  
	wal_buffers = 16MB  
	default_statistics_target = 500  
	random_page_cost = 4  
	effective_io_concurrency = 2  
	work_mem = 6553kB  
	min_wal_size = 4GB  
	max_wal_size = 16GB*  
   
1.	Перезапустила *postgresql*: *sudo systemctl restart postgresql*.  
  
	![рис.2](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW05%20-%20vacuum/image/im2.jpg) 
  
	Погуглила, что выводит тест:  
	В первых семи строках выводятся значения некоторых самых важных параметров.  
	В шестой строке выводится максимальное число повторов транзакций с ошибками сериализации  
	или взаимоблокировки .  
	В восьмой строке показывается количество выполненных и запланированных транзакций  
	(произведение числа клиентов и числа транзакций для одного клиента);  
	эти количества будут различаться, только если выполнение завершится досрочно  
	или какие-либо команды SQL завершатся ошибкой.  
	(В режиме -T выводится только число фактически выполненных транзакций.)  
	В следующей строке выводится количество транзакций, не выполненных из-за ошибок  
	сериализации или взаимоблокировок.  
	В последней строке показывается число транзакций в секунду.
  
	Сложно сделать какой-то вывод, не сильно различаются результаты, но как будто бы стало чуть хуже.
	
	
1.	Создала таблицу с 1 млн записей.  

	![рис.3](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW05%20-%20vacuum/image/im3.jpg) 
 
1.	Посмотрела размер таблицы (42MB)

	![рис.4](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW05%20-%20vacuum/image/im4.jpg) 
 
1.	5 раз обновила строки и добавила символ в каждую строку.  
  
1.	Посмотрела количество живых и мертвых строк, и последний автовакуум:  
  
	*SELECT relname, n_live_tup, n_dead_tup,   
	trunc(100*n_dead_tup/(n_live_tup+1))::float AS "ratio%",   
	last_autovacuum FROM pg_stat_user_tables WHERE relname = 'random_data';*  

	![рис.5](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW05%20-%20vacuum/image/im5.jpg) 

	Мертвых строк нет, автовакуум уже сработал.  

1. 5 раз обновила строки и добавила символ в каждую строку.  
  
1. Посмотрела количество живых и мертвых строк, и последний автовакуум:  
  
	*SELECT relname, n_live_tup, n_dead_tup,   
	trunc(100*n_dead_tup/(n_live_tup+1))::float AS "ratio%",   
	last_autovacuum FROM pg_stat_user_tables WHERE relname = 'random_data';*  
  
	![рис.6](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW05%20-%20vacuum/image/im6.jpg) 
  
	Мертвые строки есть. Подождала, проверила сработал ли автовакуум.  
  
	![рис.7](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW05%20-%20vacuum/image/im7.jpg) 
  	
	Теперь мертвых строк нет.  
  
1.	Размер таблицы стал 296 MB.  
  
	![рис.8](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW05%20-%20vacuum/image/im8.jpg)  
  
1.	Отключила автовакуум:  
	*alter table random_data set (autovacuum_enabled = false);*  
  
1.	10 раз обновила строки и добавила символ в каждую строку.  
  
1.	Размер таблицы стал 507 MB.  
  
	![рис.9](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW05%20-%20vacuum/image/im9.jpg) 
  
1. Проверила, что автовакуум не сработал, имеется >10 млн мертвых строк.  
  
1. Включила автовакуум:  
	*alter table random_data set (autovacuum_enabled = true);*
  
	через время проверила, что автовакуум сработал  
  
	![рис.10](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW05%20-%20vacuum/image/im10.jpg) 
    
	Проверила размер таблицы, он **не изменился**.   
	Т.е. вакуум пометил строки, как свободные, на которые можно записывать новые записи, а сам размер файла с таблицей не изменился.  
  
  
  
**Задача со * **
	*Написать анонимную процедуру, в которой в цикле 10 раз обновятся все строчки в искомой таблице.  
	Не забыть вывести номер шага цикла.*    

	*create or replace procedure random_data_update()  
	LANGUAGE plpgsql  
	AS $procedure$  
	DECLARE  
  i Int;    
	BEGIN  
		FOR i IN 1..10 loop  
			UPDATE random_data  
			SET x = random()::text;  -- генерация случайного числа, и перевод его в текст  
			RAISE NOTICE 'Шаг цикла: %', i; -- выводит номер шага цикла в журнал уведомлений  
		END LOOP;  
		end;  
	$procedure$  
	;  

	call random_data_update();     

	select * from random_data;*  

	![рис.11](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW05%20-%20vacuum/image/im11.jpg) 
	