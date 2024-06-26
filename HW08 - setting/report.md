**Отчет ДЗ №8 Нагрузочное тестирование и тюнинг PostgreSQL**  
   
	Требуется:  
	- сделать нагрузочное тестирование PostgreSQL  
	- настроить параметры PostgreSQL для достижения максимальной производительности      
  
1. Пользуюсь своей ВМ: 4 ядра ЦП, 8 ГБ ОЗУ и диск HDD на 350 ГБ  
  
1. Установлена Postgresql 15  
	Запустила pgbrench на текущих настройках конфигурации  
	*pgbench -P 30 -T 60 -c 10 -U postgres testdb*  
  
	![рис.1](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW08%20-%20setting/image/im1.jpg) 
  
1. С помощью калькулятора параметров PGTune настроила некоторые параметры  
	(добавила данные параметры в конце файла conf)  
  
	![рис.2](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW08%20-%20setting/image/im2.jpg)  
  
	![рис.3](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW08%20-%20setting/image/im3.jpg) 
   
	**Основные важные параметры:**  
	• *shared_buffers*: Уменьшение размера общего буфера может привести к более частым обращениям к диску, но может улучшить производительность для систем с большим объемом памяти.  
	• *effective_cache_size*: Уменьшение эффективного размера кэша может привести к вытеснению большего количества данных из кэша, что может улучшить производительность для систем с ограниченным объемом памяти.  
	• *work_mem*: Уменьшение размера рабочей памяти может привести к более частым сбросам сортировок на диск, но может улучшить производительность для систем с небольшим объемом памяти.  
	• *maintenance_work_mem*: Уменьшение размера рабочей памяти обслуживания может привести к более частым сбросам вакуумных операций на диск, но может улучшить производительность для систем с небольшим объемом памяти.  
	• *wal_buffers*: Уменьшение размера буферов WAL может привести к более частым сбросам WAL на диск, но может улучшить производительность для систем с быстрыми дисками.  
	• *max_connections*: Увеличение максимального количества подключений может привести к исчерпанию ресурсов системы и снижению производительности.  
  
	Можно также изменить следующие параметры в ущерб надежности:
	
	*bgwriter_delay* = 10ms (также добавила в файл)  
  
	Задаёт задержку между раундами активности процесса фоновой записи.  
	Во время раунда этот процесс осуществляет запись некоторого количества загрязнённых буферов  
	(это настраивается следующими параметрами). Затем он засыпает на время bgwriter_delay  
	(задаваемое в миллисекундах), и всё повторяется снова. Однако если в пуле не остаётся загрязнённых буферов,  
	он может быть неактивен более длительное время.  
	По умолчанию этот параметр равен 200 миллисекундам (200ms).  
	Уменьшение задержки фонового процесса записи может привести к более частым сбросам буферов на диск,  
	что может улучшить производительность, но может увеличить нагрузку на диск.
  
	*ALTER SYSTEM SET synchronous_commit = off;* --Асинхронный режим  
  
	Параметр определяет, после завершения какого уровня обработки WAL сервер будет сообщать об успешном выполнении операции)  
   
	*ALTER SYSTEM SET full_page_writes=off;* -- Отключение записи всего содержимого каждой страницы в wal при checkpoint  
  
	Когда этот параметр включён, сервер Postgres Pro записывает в WAL всё содержимое каждой страницы при первом изменении  
	этой страницы после контрольной точки. Это необходимо, т.к. запись страницы,  
	прерванная при сбое операционной системы, может выполниться частично, и на диске окажется страница,  
	содержащая смесь старых данных с новыми. При этом информации об изменениях на уровне строк, которая обычно сохраняется  
	в WAL, будет недостаточно для получения согласованного содержимого такой страницы при восстановлении после сбоя.  
	Сохранение образа всей страницы гарантирует, что страницу можно восстановить корректно, ценой увеличения объёма данных,  
	которые будут записываться в WAL. (Так как воспроизведение WAL всегда начинается от контрольной точки,  
	достаточно сделать это при первом изменении каждой страницы после контрольной точки.  
	Таким образом, уменьшить затраты на запись полных страниц можно, увеличив интервалы контрольных точек.)  
  
	Отключение этого параметра ускоряет обычные операции, но может привести к неисправимому повреждению   
	или незаметной порче данных после сбоя системы.  
  
	*ALTER SYSTEM SET wal_level=minimal;* --Перевод режима журналирования в минимальный режим. (Реплик у нас нет, а минимального уровня хватит для восстановления).
	
	Параметр wal_level определяет, как много информации записывается в WAL.  
	Со значением replica (по умолчанию) в журнал записываются данные, необходимые для поддержки архивирования WAL  
	и репликации, включая запросы только на чтение на ведомом сервере. Вариант minimal оставляет только информацию,  
	необходимую для восстановления после сбоя или аварийного отключения.  
  
	*ALTER SYSTEM SET max_wal_senders=0;* --Данный параметр надо сделать равным 0 т.к. изменился wal_level. Если этого не сделать, то кластер PostgreSQL не запустится.  
  
	*ALTER SYSTEM SET fsync=off;* --Отключение режима синхронизации (подтверждения физической записи на СХД) СУБД с ОС по дисковому вводу/выводу.  
  	
	Выключенный параметр может привести к неисправимой порче данных в случае отключения питания или сбоя системы.  
  
  
1. Применила настройки, рестартовала кластер. Провела нагрузочное тестирование.
  
	![рис.4](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW08%20-%20setting/image/im4.jpg) 
   
 
1. Сравниваю показатели:  
  
	•	number of transactions actually processed было 71 257 стало 213 934. Количество транзакций увеличилось в 3 раза.  
	•	latency average было 8.416 ms стало 2.802 ms. Средняя задержка уменьшилась в 3 раза.  
	•	latency stddev было 6.327 ms стало 1.889 ms. Средняя задержка дисковых устройств уменьшилась в 3,37 раза.  
	•	tps было 1187.7… стало 3566.4…. Примерно в 3 раза увеличилось количество транзакций в секунду.   
  
	Получается, что в целом производительность улучшилась в 3 раза. Считаю, очень хороший результат.   
	Но опять же в ущерб надежности.  
