**Отчет ДЗ №11 Работа с индексами**  
   
	Требуется:   
	- знать и уметь применять основные виды индексов PostgreSQL  
	- строить и анализировать план выполнения запроса  
	- уметь оптимизировать запросы для с использованием индексов     
  
1. Создала таблицу *tbl_index* на *testdb1/testnm*  
  
	*create table tbl_index as  
	select generate_series as pkey, generate_series::text || (random() * 10)::text as cnumber,   
    (array['Volvo', 'BMV', 'Audi', 'Skoda', 'Lada'])[floor(random() * 6)] as cname  
	from generate_series(1, 100000);*    
  
	![рис.1](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW11%20-%20index/image/im1.jpg) 

	Проверила план запроса  
   
	*select * from testnm.tblindex where cname = 'Volvo';*  
  
	![рис.2](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW11%20-%20index/image/im2.jpg) 
    
	Создала индекс  

	*create index tbl_index_pkey_idx on tbl_index(pkey);*  
    
	![рис.3](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW11%20-%20index/image/im3.jpg)     
  
1. Проверила план запроса  
  
	*explain analyse select * from testnm.tbl_index where pkey = 2;*  
    
	![рис.4](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW11%20-%20index/image/im4.jpg) 
  
1. Реализовала индекс для полнотекстового поиска  
  
	*alter table tbl_index add column cname_tsv tsvector;*  
	*update tbl_index set cname_tsv=to_tsvector(cname);*  
  
	![рис.5](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW11%20-%20index/image/im5.jpg) 
  
	Проверила план запроса  
	*explain analyse select * from testnm.tbl_index where cname_tsv @@ to_tsquery('Audi');*  
  
	![рис.6](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW11%20-%20index/image/im6.jpg)  
  
1. Реализовала индекс на поле с функцией  
  
	*create index tbl_index_idx_func on tbl_index(lower(cname));*  
  
	![рис.7](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW11%20-%20index/image/im7.jpg) 
  
	Проверила план запроса  
	*explain analyse select * from testnm.tbl_index where cname = 'Lada';*  
  
	![рис.8](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW11%20-%20index/image/im8.jpg) 
  
1. Реализовала индекс на несколько полей  
  
	*create index tbl_index_idx2 on tbl_index(pkey,cname);*  
   
	![рис.9](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW11%20-%20index/image/im9.jpg) 
  
	Проверила план запроса  
	*explain analyse select * from testnm.tbl_index where pkey=119 and cname = 'BMV';*  
  
	![рис.10](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW11%20-%20index/image/im10.jpg) 
    
1. Преимущества индексов  
  
	Уникальный индекс:  
	• Гарантирует, что значения в определенном столбце или наборе столбцов являются уникальными.  
	• Полезен для поддержания целостности данных и ускорения поиска по уникальному ключу.  
	• Может замедлить вставку и обновление, поскольку база данных должна проверять уникальность каждого значения.  
  	
	Индекс на часть таблицы:  
	• Индексирует только подмножество строк в таблице, которые удовлетворяют определенному условию.  
	• Экономит место и ускоряет запросы к этому подмножеству.  
	• Может привести к избыточности индексов, если условие индексации пересекается с другими индексами.  
  
	Индекс для полнотекстового поиска:  
	• Позволяет выполнять поиск по тексту в столбцах.  
	• Ускоряет полнотекстовые запросы, такие как поиск по ключевым словам или фразам.  
	• Может быть ресурсоемким и медленным для вставки и обновления, поскольку база данных должна токенизировать и проиндексировать текст.  
  
	Индекс на поле с функцией:  
	• Индексирует столбец, к которому применяется определенная функция.  
	• Полезен для ускорения запросов, которые используют эту функцию в качестве условия поиска.  
	• Может быть менее эффективным, чем индекс на сам столбец, особенно если функция является сложной или редко используется.    
  
	Индекс на несколько полей:  
	• Индексирует несколько столбцов одновременно.  
	• Ускоряет запросы, которые используют эти столбцы в качестве условия поиска.  
	• Может быть менее эффективным, чем отдельные индексы для каждого столбца, особенно если столбцы имеют низкую кардинальность.  
  
1. Особых проблем при выполнении ДЗ не было.  
