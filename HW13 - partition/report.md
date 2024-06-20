**Отчет ДЗ №13 Секционирование таблицы**  
   
	Требуется:   
	- научиться секционировать таблицы     
	
  
	Теория  
  
	Секционирование таблицы в SQL - это техника, которая позволяет разделить одну большую таблицу на несколько более мелких таблиц, называемых секциями.   
	Эти секции связаны между собой по определенному критерию, например, по значению столбца даты или по диапазону значений столбца ID.  
  
	Преимущества секционирования:  
	Улучшенная производительность запросов:    
		- Запросы, которые обращаются только к определенной части данных, могут быть выполнены быстрее, так как они обращаются только к соответствующей секции.  
		- Снижается фрагментация данных, что улучшает производительность сканирования таблиц.  
	Управление пространством:  
		- Секции могут быть удалены или добавлены по мере необходимости, что позволяет легко управлять размером таблицы и использовать дисковое пространство более эффективно.  
	Упрощение резервного копирования и восстановления:  
		- Секционирование позволяет создавать резервные копии и восстанавливать только необходимые секции, что сокращает время и ресурсы, необходимые для этих операций.  
	Улучшенная конфиденциальность:  
		- Секции могут быть разделены на разные физические хранилища, что обеспечивает более высокий уровень конфиденциальности данных.  
  
	Типы секционирования:  
	- Секционирование по диапазону: Секции определяются диапазонами значений одного или нескольких столбцов.  
	- Секционирование по списку: Секции определяются списком дискретных значений одного столбца.  
	- Секционирование по хешу: Секции определяются хеш-функцией, примененной к одному или нескольким столбцам.  
  
	Как работает секционирование:   
	- SQL-сервер создает секции с помощью специальных операторов, таких как PARTITION BY.  
	- Запросы, которые обращаются к секционированной таблице, автоматически распределяются по соответствующим секциям.  
	- Управление секциями (добавление, удаление, изменение) осуществляется с помощью специальных операторов.  

	Важные моменты:  
	- Выбирайте правильное критерий секционирования:  Оно должно быть основано на том, как вы чаще всего будете обращаться к данным в таблице.  
	- Учитывайте количество секций:  Слишком большое количество секций может усложнить управление таблицей и снизить производительность.  
	- Используйте секционирование с осторожностью:  Это мощная техника, но ее неправильное применение может привести к проблемам.  
  
 
 
   
1. Развернула базу flights
  
	*cd /tmp/  
	/tmp$ wget https://edu.postgrespro.ru/demo-big.zip  
	/tmp$ chmod 777 /tmp/demo-big.zip  
	/tmp$ unzip demo-big.zip  
	/tmp$ sudo -u postgres psql -f demo-big-20170815.sql*  
	![рис.1](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW13%20-%20partition/image/im1.jpg)   
  
  	
	Проверила диапазон данных  
  
	*select min(scheduled_departure),max(scheduled_departure) from flights;*  
  
	![рис.2](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW13%20-%20partition/image/im2.jpg)   
  
  	
	Секционировала таблицу по месяцам (Чтобы работал режим выборки данных из партиций, необходимо включить параметр *enable_partition_pruning*)    
  
	*create table flights_1 (like flights) partition by range (scheduled_departure);*  
  
	*create table flights_1_2016_08 partition of flights_1 for values from ('2016-08-01') to ('2016-09-01');  
	create table flights_1_2016_09 partition of flights_1 for values from ('2016-09-01') to ('2016-10-01');  
	create table flights_1_2016_10 partition of flights_1 for values from ('2016-10-01') to ('2016-11-01');  
	create table flights_1_2016_11 partition of flights_1 for values from ('2016-11-01') to ('2016-12-01');  
	create table flights_1_2016_12 partition of flights_1 for values from ('2016-12-01') to ('2017-01-01');  
	create table flights_1_2017_01 partition of flights_1 for values from ('2017-01-01') to ('2017-02-01');  
	create table flights_1_2017_02 partition of flights_1 for values from ('2017-02-01') to ('2017-03-01');  
	create table flights_1_2017_03 partition of flights_1 for values from ('2017-03-01') to ('2017-04-01');  
	create table flights_1_2017_04 partition of flights_1 for values from ('2017-04-01') to ('2017-05-01');  
	create table flights_1_2017_05 partition of flights_1 for values from ('2017-05-01') to ('2017-06-01');  
	create table flights_1_2017_06 partition of flights_1 for values from ('2017-06-01') to ('2017-07-01');  
	create table flights_1_2017_07 partition of flights_1 for values from ('2017-07-01') to ('2017-08-01');  
	create table flights_1_2017_08 partition of flights_1 for values from ('2017-08-01') to ('2017-09-01');  
	create table flights_1_2017_09 partition of flights_1 for values from ('2017-09-01') to ('2017-10-01');*  
	
  	
	Скопировала данные в новую таблицу  
  
 *insert into flights_1 (select * from flights);*  
  
	![рис.3](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW13%20-%20partition/image/im3.jpg)   
   
  
	*select 'select pg_size_pretty(pg_table_size('''||tablename||'''));' from pg_tables where tablename like 'flights_1%';*  
  
	![рис.4](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW13%20-%20partition/image/im4.jpg)   
   
  	
	Проверила содержание одной из партиций  
  
	*select pg_size_pretty(pg_table_size('flights_1_2017_06'));*  
  
	![рис.5](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW13%20-%20partition/image/im5.jpg)    
  
	*select * from flights_1_2017_06;*  
  
	![рис.6](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW13%20-%20partition/image/im6.jpg)   
  
  	
	Проверила размер основной таблицы  
  
	*select pg_size_pretty(pg_table_size('flights_1'));*  
  
	![рис.7(https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW13%20-%20partition/image/im7.jpg)   
    
  	
	Основная таблица данных не содержит, но благодаря партициям select вытаскивает данные   
  
	*select * from flights_1;*  
  
	![рис.8](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW13%20-%20partition/image/im8.jpg)     
  	
	
	
	
	
	
	
	
	
	
	
	
	
	
	

  
    
