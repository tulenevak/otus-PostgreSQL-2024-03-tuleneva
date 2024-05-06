**Отчет ДЗ №6 Работа с журналами**  
   
	Требуется:  
	- уметь работать с журналами и контрольными точками  
	- уметь настраивать параметры журналов    
  
1. Настроила checkpoint_timeout = 30 s.  
  
	![рис.1](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW06%20-%20journals/image/im1.jpg) 
  
	Перезапустила *sudo systemctl restart postgresql*.   
  
1. Посмотрела текущее местоположение записи в журнале.  
   
	![рис.2](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW06%20-%20journals/image/im2.jpg) 
  
	10 минут подавала нагрузку.
   
	![рис.3](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW06%20-%20journals/image/im3.jpg) 
  	
	Посмотрела текущее местоположение записи в журнале.  
   
	![рис.4](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW06%20-%20journals/image/im4.jpg) 
  
1. Объем журнальных файлов составил 643 MB.  
    
	![рис.5](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW06%20-%20journals/image/im5.jpg) 
  
	Объем в среднем на 1 контрольную точку:  
	643(объем всего)/600(время 10 мин)*30(время выполнения контрольной точки) = 32,15 MB.  
  
1. Проверила статистику.  
    
	![рис.6](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW06%20-%20journals/image/im6.jpg) 
  
	- *checkpoints_timed*: Количество точек проверки, выполненных по таймеру.  
	- *checkpoints_req*: Количество точек проверки, выполненных по запросу.  
	- *checkpoint_write_time*: Общее время, затрачиваемое bgwriter на запись контрольных точек, в миллисекундах.  
	- *checkpoint_sync_time*: Общее время, затрачиваемое bgwriter на синхронизацию контрольных точек с диском, в миллисекундах.  
	- *buffers_checkpoint*: Количество буферов, очищенных во время контрольных точек.  
	- *buffers_clean*: Количество буферов, очищенных фоновым процессом без создания контрольной точки.  
	- *maxwritten_clean*: Максимальное количество байт, записанных в один файл журнала в одном сеансе очистки.  
	- *buffers_backend*: Количество буферов, очищенных фоновыми процессами.  
	- *buffers_alloc*: Количество буферов, выделенных фоновыми процессами.  
  
	Полагаю, что такое большое количество точек связано с тем,  
	что время чекпоинтов я настроила вчера, а тесты запустила сегодня...  
	По крайней мере можно сделать вывод, что если *checkpoints_req* небольшое,   
	значит контрольные точки проходили примерно по расписанию  
	(при повышенной нагрузке контрольная точка вызывается чаще - при достижении объема *max_wal_size*).  
  
1. Сравнила tps в синхронном/асинхронном режиме утилитой pgbench.  
  
	*ALTER SYSTEM SET synchronous_commit = on;   
	pg_reload_conf()*  
  
	![рис.7](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW06%20-%20journals/image/im7.jpg) 
  
	*ALTER SYSTEM SET synchronous_commit = off;   
	pg_reload_conf()*  
  
	![рис.8](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW06%20-%20journals/image/im8.jpg) 
  	
	- Разница tps между синхронным методом 1065 tps и асинхронным методом 3644 tps (увеличилось количество транзакций в секунду).   
		В асинхронном режиме мы используем wal_writer_delay. Это увеличивает время отлика записи файл журнала wal на диск.  
	- Уменьшилась latency average 7 к 2 ms (среднее время задержки).  
	- Уменьшилась latency stddev 5 к 1 ms (время задержки с дисковым устройством).  
  
	Таким образом производительность сервера увеличилась, но упала надежность и отказоустойчивость.    
	Нужно выбирать в зависимости от целей БД (конкретной задачи).  
  
1. Создала 2й кластер с включенной контрольной суммой страниц.  
  
	![рис.9](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW06%20-%20journals/image/im9.jpg) 
  
	Создала таблицу и внесла данные. Посмотрела ее месторасположение на диске.  
  
	![рис.10](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW06%20-%20journals/image/im10.jpg)  
  
	Выключила кластер *sudo systemctl stop postgresql@15-main_2*.  
  
	Изменила пару байт в таблице *sudo nano /var/lib/postgresql/15/main_2/base/5/16388*.  
  
	![рис.11](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW06%20-%20journals/image/im11.jpg)  
   
	Включила кластер, сделала выборку из таблицы.  
  
	![рис.12](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW06%20-%20journals/image/im12.jpg)  
   
	Ошибки нет, но и данных нет.
	Погуглила: если падает ошибка можно использовать *ALTER SYSTEM SET ignore_checksum_failure = on;*    
	Тогда считаются неповрежденные данные из таблицы (т.е. не будут учитываться контрольные суммы).  
	Но т.к. у меня ошибка не упала, то результат остался такой же. 0 выведенных записей,   
	т.к. файл с таблицей поврежден полностью.  
  
	![рис.13](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW06%20-%20journals/image/im13.jpg)  
   