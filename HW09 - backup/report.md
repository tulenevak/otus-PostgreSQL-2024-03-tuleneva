**Отчет ДЗ №9 Бэкапы**  
   
	Требуется:  
	- применить логический бэкап. Восстановиться из бэкапа      
  
1. Пользуюсь своей ВМ: 4 ядра ЦП, 8 ГБ ОЗУ и диск HDD на 350 ГБ  
1. Установлена Postgresql 15  
	БД testdb  
	Схема testnm  
  
1. Создала таблицу с автосгенерированными 100 записями   
	*create table for_backup as*  
	*select 'не потерять' as x*  
	*from generate_series(1, 100);*  
    
1. Под линукс пользователем Postgres создала каталог для бэкапов  
	*sudo su postgres*  
	*cd /var/lib/postgresql*  
	*mkdir backups*  
	*ls*  
   
	![рис.1](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW09%20-%20backup/image/im1.jpg) 
  
1. Сделала логический бэкап используя утилиту COPY   
	*psql -d testdb*  
	*\copy testnm.for_backup to '/var/lib/postgresql/backups/for_backup.sql';*  
   
	![рис.2](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW09%20-%20backup/image/im2.jpg) 
  
1. Восстановила данные во вторую таблицу  
	*create table from_backup (x text);*  
	*\copy testnm.from_backup from '/var/lib/postgresql/backups/for_backup.sql';*  
    
	![рис.3](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW09%20-%20backup/image/im3.jpg) 
  
1. Используя утилиту pg_dump создала бэкап с оглавлением в кастомном сжатом формате 2 таблиц  
  
	//----------------------------------------------------------------------  
	Немного теории:  
	Бэкап с оглавлением в PostgreSQL относится к механизму резервного копирования, который создает иерархический снимок состояния базы данных на определенную точку во времени.  
  
	Оглавление (TOC) представляет собой метаданные, описывающие структуру резервной копии. Оно содержит информацию о следующих элементах:  
	• Набор данных таблиц  
	• Индексы  
	• Ограничения  
	• Процедуры  
	• Функции  
	• Триггеры  
  
	Когда создается бэкап с оглавлением, PostgreSQL фиксирует состояние базы данных и сохраняет его вместе с метаданными TOC.   
	Это позволяет восстановить резервную копию целиком или выборочно восстановить отдельные объекты (такие как таблицы или индексы).    
  
	Преимущества использования бэкапов с оглавлением:  
	• Быстрое и гибкое восстановление: Позволяет восстанавливать отдельные объекты без необходимости восстановления всей базы данных.  
	• Упрощенное управление резервными копиями: TOC обеспечивает дополнительную организацию резервных копий, упрощая анализ содержимого и управление ими.  
	• Эффективность использования пространства: Бэкапы с оглавлением хранят только измененные данные, что уменьшает размер резервной копии.  
	• Поддержка точек восстановления (PITR): Бэкапы с оглавлением позволяют выполнять восстановление до определенной точки во времени, даже если с тех пор база данных претерпела изменения.  
	//----------------------------------------------------------------------  
  
	*cd /var/lib/postgresql/backups*  
	*pg_dump -d testdb --table testnm.for_backup --table testnm.from_backup --format custom --compress=2 > backup.dump*  
    
	![рис.4](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW09%20-%20backup/image/im4.jpg) 
  
	Проверила наличие дампа в каталоге  
    
	![рис.5](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW09%20-%20backup/image/im5.jpg) 
  
	![рис.6](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW09%20-%20backup/image/im56.jpg) 
   
1. Используя утилиту pg_restore восстановила в новую БД только вторую таблицу 
   
	*echo "CREATE DATABASE testdb1;" | psql -U postgres*  
	*echo "CREATE SCHEMA testnm;" | psql -U postgres -d testdb1*  
	*pg_restore -d testdb1 -t from_backup backup.dump*  
    
	![рис.7](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW09%20-%20backup/image/im7.jpg) 
  
	Проверила данные   
	*psql -d testdb1*  
	*select * from testnm.from_backup limit 10;*  
    
	![рис.8](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW09%20-%20backup/image/im8.jpg) 
  