**Отчет ДЗ №10 Репликация**  
   
	Требуется:  
	- реализовать свой миникластер на 3 ВМ      
  
1. На 1 ВМ Изменила настройки конфигурации.   
  
	![рис.1](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW10%20-%20replica/image/im1.jpg) 
    
	Создала таблицы test для записи, test2 для запросов на чтение.   
	Раздала соответствующие права новому пользователю.  
  
	![рис.2](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW10%20-%20replica/image/im2.jpg) 
  
	Проверила права  
  
	![рис.3](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW10%20-%20replica/image/im3.jpg) 
  
1. Создала публикацию таблицы test.  
  
	![рис.4](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW10%20-%20replica/image/im4.jpg)  
  
1. На ВМ 2 также установила postgresql 15, применила параметры конфигурации.    
	
	*sudo nano /etc/postgresql/15/main/postgresql.conf*  
	*sudo nano /etc/postgresql/15/main/pg_hba.conf*  
	
	Создала пользователя и таблицы test2 для записи, test для запросов на чтение.  
  
	![рис.5](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW10%20-%20replica/image/im5.jpg)  
  
	Проверила права.  
  
	![рис.6](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW10%20-%20replica/image/im6.jpg)  
  
1. Создала публикацию таблицы test2 и подписалась на публикацию таблицы test1 с ВМ №1.  
  
	*CREATE PUBLICATION test2_pub FOR TABLE test2;*  
  
	![рис.7](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW10%20-%20replica/image/im7.jpg)  
  
	Во время подписи на публикацию возникла проблема подключения,   
	пришлось вернуться на ВМ 1 и скорректировать конфигурационный файл.  
  
	![рис.8](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW10%20-%20replica/image/im8.jpg)   
  
	Подписка:   
  
	![рис.9](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW10%20-%20replica/image/im9.jpg)   
  
1. На 3ВМ поставила postgresql 15, исправила конф.файлы.  
   
	![рис.10](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW10%20-%20replica/image/im10.jpg)   
  
	Создала пользователя, таблицы, права на чтение.  
  	
	![рис.11](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW10%20-%20replica/image/im11.jpg)   
  
	Проверила права.    
  	
	![рис.12](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW10%20-%20replica/image/im12.jpg)  
  
	Подписалась на таблицы из ВМ1 и ВМ2 и проверила, что все изменения есть.  
  
	*CREATE SUBSCRIPTION test2_3_pub CONNECTION 'host=10.7.0.215 port=5432 user=postgres password=postgres dbname=postgres' PUBLICATION test2_pub WITH (copy_data = true);*  
	*CREATE SUBSCRIPTION test_3_pub CONNECTION 'host=10.7.0.141 port=5432 user=postgres password=postgres dbname=postgres' PUBLICATION test_pub WITH (copy_data = true);*  
  	
	![рис.13](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW10%20-%20replica/image/im13.jpg)   
  	
	![рис.14](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW10%20-%20replica/image/im14.jpg)  
  
	![рис.15](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW10%20-%20replica/image/im15.jpg) 
  	
  
  