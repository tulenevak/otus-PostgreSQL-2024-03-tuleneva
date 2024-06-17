**Отчет ДЗ №12 Работа с join'ами, статистикой**  
   
	Требуется:   
	- знать и уметь применять различные виды join'ов
	- строить и анализировать план выполенения запроса
	- оптимизировать запрос
	- уметь собирать и анализировать статистику для таблицы     

1. Реализовала прямое соединение 2х таблиц   
  
	Создала 2 таблицы: "Марки машин", "Владельцы":  

	*CREATE TABLE car_brands (  
		id SERIAL PRIMARY KEY,  
		brand_name VARCHAR(255) NOT NULL  
		); *   
  
	*INSERT INTO car_brands (brand_name) VALUES  
	('Toyota'),  
	('Honda'),  
	('Ford'),  
	('Chevrolet'),  
	('BMW'),  
	('Mercedes-Benz'),  
	('Volkswagen'),  
	('Audi'),  
	('Nissan'),  
	('Hyundai'); *     
    
	*CREATE TABLE car_owners (  
		id SERIAL PRIMARY KEY,  
		owner_name VARCHAR(255) NOT NULL,  
		brand_id INT REFERENCES car_brands(id)  
		);*   
   
	*INSERT INTO car_owners (owner_name, brand_id) VALUES  
		('John Doe', 1),  
		('Jane Doe', 2),  
		('Peter Pan', 3),  
		('Alice Wonderland', 4),  
		('Bob Smith', 5),  
		('Mary Jones', 6),  
		('David Brown', 7),  
		('Susan Williams', 8),  
		('Michael Davis', 9),  
		('Jennifer Wilson', 10),  
		('Richard Garcia', 1),  
		('Linda Rodriguez', 2),  
		('Christopher Martinez', 3),  
		('Barbara Wilson', 4),  
		('Thomas Anderson', 5),  
		('Elizabeth Taylor', 6),  
		('Joseph Lewis', 7),  
		('Patricia Moore', 8),  
		('Daniel Jackson', 9),  
		('Margaret Clark', 10);*  
  
	Прямое соединение таблиц:  
	
	*select * from car_brands cb, car_owners co where cb.id = co.brand_id;*  
   
	![рис.1](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW12%20-%20join/image/im1.jpg) 
  
    
	План запроса:  
  
	*explain analyze  
	select * from car_brands cb, car_owners co where cb.id = co.brand_id;*  
    
	![рис.2](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW12%20-%20join/image/im2.jpg) 
  

1. Реализовала левостороннее соединение (LEFT JOIN)   
  
	Основные типы соединений:  
	• *INNER JOIN*: Используется для объединения строк из двух таблиц, только если они соответствуют условию соединения.  
	• *LEFT JOIN*: Используется для объединения всех строк из первой таблицы и соответствующих строк из второй таблицы.  
	• *RIGHT JOIN*: Используется для объединения всех строк из второй таблицы и соответствующих строк из первой таблицы.  
	• *FULL JOIN*: Используется для объединения всех строк из обеих таблиц.  
  
	Левостороннее соединение:
  
	*select * from car_brands cb left join car_owners co on cb.id = co.brand_id;*  
   
	![рис.3](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW12%20-%20join/image/im3.jpg) 
  
  
	План запроса:  
	*explain analyze select * from car_brands cb left join car_owners co on cb.id = co.brand_id;*  
   
	![рис.4](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW12%20-%20join/image/im4.jpg) 
  
  
1. Реализовала кросс-соединение (CROSS JOIN)      
  	
	Соединение *CROSS JOIN* - это тип соединения, который создает декартово произведение двух таблиц.   
	Декартово произведение - это таблица, которая содержит все возможные комбинации строк из двух исходных таблиц.   
  
	Как работает *CROSS JOIN*:  
	• Для каждой строки в первой таблице, *CROSS JOIN* создает новую строку в результирующей таблице для каждой строки во второй таблице.  
	• Количество строк в результирующей таблице равно произведению количества строк в первой и второй таблицах.  
	
	Применение *CROSS JOIN*:  
	• Редко используется самостоятельно: *CROSS JOIN* обычно не используется самостоятельно, так как он генерирует много строк, которые могут быть не нужны.  
	• Создание комбинаций: *CROSS JOIN* может быть полезен для создания всех возможных комбинаций элементов из двух таблиц, например, для генерации тестовых данных.  
	• Сочетание с WHERE: *CROSS JOIN* можно использовать в сочетании с предложением WHERE для фильтрации результирующего набора данных и получения только необходимых комбинаций.  
  
	Важно отметить:  
	• *CROSS JOIN* не использует никакие условия соединения, поэтому он генерирует все возможные комбинации строк из двух таблиц.  
	• *CROSS JOIN* не требует использования ключевых слов JOIN или ON.  
  
  
	Кросс-соединение:  
  
	*select * from car_brands cb cross join car_owners co;*  
   
	![рис.5](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW12%20-%20join/image/im5.jpg) 
  
  
	План запроса: 
  
	*explain analyze select * from car_brands cb cross join car_owners co;*  
   
	![рис.6](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW12%20-%20join/image/im6.jpg) 
  

1. Реализовала полное соединение (FULL JOIN)   
  
	Полное соединение таблиц *FULL JOIN* - это тип соединения, который объединяет все строки из обеих таблиц, независимо от того, соответствуют ли они условию соединения или нет.  
  
	Как работает полное соединение:  
	• Соединяет все строки из первой таблицы с соответствующими строками из второй таблицы, основываясь на условии соединения.  
	• Для строк, не имеющих соответствия в другой таблице, оно создает новые строки с NULL значениями для столбцов, которые не были соединены.  
  
	Применение полного соединения:  
	• Объединение данных из разных таблиц: Когда необходимо объединить все данные из обеих таблиц, независимо от наличия соответствия.  
	• Анализ данных: Полное соединение может быть полезно для анализа данных, когда требуется просмотреть все записи из обеих таблиц, включая те, которые не имеют соответствия.  
  
	Важно отметить:  
	• Полное соединение создает больше строк в результирующей таблице, чем другие типы соединений, так как оно включает все строки из обеих таблиц.  
	• Используйте *FULL JOIN* с осторожностью, так как он может создать большие наборы данных, которые могут замедлить обработку запросов.  
  
  
	Полное соединение:  
  
	Добавила новую марку машину в первую таблицу. Во вторую таблицу владельца не добавляю.  
  
	*INSERT INTO car_brands (brand_name) VALUES  
		('Opel');*  

	Результат запроса:  
  
	*select * from car_brands cb full outer join car_owners co on cb.id = co.brand_id;*  
   
	![рис.7](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW12%20-%20join/image/im7.jpg) 
  
  
	План запроса: 
	
	*explain analyze select * from car_brands cb full outer join car_owners co on cb.id = co.brand_id;*  
  
	![рис.8](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW12%20-%20join/image/im8.jpg) 
  
  
1. Реализовала запрос, в котором использованы разные типы соединений   
  
	Создала еще одну таблицу «Срок владения автомобилем»:  
  
	*CREATE TABLE car_ownership_duration (  
		id SERIAL PRIMARY KEY,  
		owner_id INT REFERENCES car_owners(id),  
		duration_years INT NOT NULL  
		);*  
  
	*INSERT INTO car_ownership_duration (owner_id, duration_years) VALUES  
		(1, 5),  
		(2, 3),  
		(3, 7),  
		(4, 2),  
		(5, 10),  
		(6, 8),  
		(7, 1),  
		(8, 4),  
		(9, 9),  
		(10, 6),  
		(11, 2),  
		(12, 4),   
		(13, 6), 
		(14, 1),  
		(15, 3),  
		(16, 7),  
		(17, 5),  
		(18, 9),  
		(19, 10),  
		(20, 8);*  
  
	Результат запроса:
  
	*select co.owner_name,  cb.brand_name, cd.duration_years from car_owners co   
	left join car_ownership_duration cd on co.id = cd.owner_id right join car_brands cb on cb.id = co.brand_id;*   
  
	![рис.9](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW12%20-%20join/image/im9.jpg) 
   
	План запроса:  
  
	*explain analyze select co.owner_name,  cb.brand_name, cd.duration_years from car_owners co   
	left join car_ownership_duration cd on co.id = cd.owner_id right join car_brands cb on cb.id = co.brand_id;*  
   
	![рис.10](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW12%20-%20join/image/im10.jpg) 
  
   
=======================================================================
  
	Не очень поняла, что нужно было сделать в задании со *.  
	Но т.к. в целях ДЗ было также изучение статистики, посмотрела, что выводят таблицы *pg_stat*.  
  
	В PostgreSQL *pg_stat* - это набор системных таблиц, которые хранят статистику о различных аспектах базы данных.  
	Они используются для отслеживания производительности, анализа использования ресурсов и диагностики проблем.  
  
	Основные таблицы *pg_stat*:    
  
	• pg_stat_all_tables: Содержит статистику по всем таблицам в базе данных, включая:  
		• reltuples: Оцениваемое количество строк в таблице.  
		• relpages: Количество страниц, используемых таблицей.  
		• last_mod_time: Время последнего изменения таблицы.   
		• last_analyze_time: Время последнего анализа таблицы.  
	• pg_stat_all_indexes: Содержит статистику по всем индексам в базе данных, включая:  
		• idx_scan: Количество сканирований индекса.  
		• idx_tup_read: Количество прочитанных строк с помощью индекса.  
		• last_mod_time: Время последнего изменения индекса.  
	• pg_stat_all_sequences: Содержит статистику по всем последовательностям в базе данных, включая:  
		• last_value: Последнее значение, возвращенное последовательностью.  
		• call_count: Количество вызовов функции nextval() для последовательности.  
	• pg_stat_bgwriter: Содержит статистику о фоновом писателе (background writer), который отвечает за запись изменений на диск.  
	• pg_stat_database: Содержит статистику по каждой базе данных, включая:  
		• numbackends: Количество подключений к базе данных.  
		• xact_commit: Количество завершенных транзакций.  
		• xact_rollback: Количество отмененных транзакций.  
	• pg_stat_user_tables: Содержит статистику по таблицам, доступным для определенного пользователя.  
	• pg_statio_user_tables: Содержит статистику ввода-вывода для таблиц, доступных для определенного пользователя.  
	• pg_stat_io_tables: Содержит статистику ввода-вывода для всех таблиц в базе данных.  
	• pg_statio_all_tables: Содержит расширенную статистику ввода-вывода для всех таблиц в базе данных.  
	• pg_statio_all_indexes: Содержит расширенную статистику ввода-вывода для всех индексов в базе данных. 

	Как использовать *pg_stat*:  
	• Анализ производительности:  Просмотрите статистику ввода-вывода, количество сканирований таблиц и индексов, чтобы выявить узкие места в производительности.  
	• Диагностика проблем:  Изучите статистику об отмененных транзакциях, ошибках и времени последнего анализа таблицы, чтобы диагностировать проблемы с базой данных.  
	• Мониторинг использования ресурсов:  Проанализируйте количество подключений к базе данных, активных пользователей и использования ресурсов для оптимизации конфигурации и планирования ресурсов.  

	Важно отметить:  
	• Статистика pg_stat обновляется в фоновом режиме, поэтому данные могут быть немного устаревшими.   
	• pg_stat таблицы не предназначены для прямого редактирования, так как это может привести к непредсказуемым результатам.    
