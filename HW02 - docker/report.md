**Отчет ДЗ №2 Установка PostgreSQL**  
  

1.	Создала аккаунт в YC  
1.	Установила Интерфейс командной строки Yandex Cloud (CLI) и проверила работу  

	![рис.1](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW02%20-%20docker/image/im1.jpg)
	
1.	Создала сетевую структуру для ВМ   

	Столкнулась с проблемой лимита, узнала, что по умолчанию положено только 2 сети для аккаунта. Одну сеть до этого я создала средствами интерфейса аккаунта (также создала ВМ и реестр, потом все удалила). Теперь пробую сделать с помощью команд CLI. Из-за лимита, созданную ранее сеть, также пришлось удалить).  
 
	![рис.2](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW02%20-%20docker/image/im2.jpg)

	Сеть/подсеть созданы  
 
	![рис.3](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW02%20-%20docker/image/im3.jpg)
	
1.	Создала ключ SHH  
	
	![рис.4](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW02%20-%20docker/image/im4.jpg)
	
1.	Создала ВМ  
	![рис.5](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW02%20-%20docker/image/im5.jpg)

1. Подключилась к ВМ (другой IP, потому что удаляла ВМ и создавала заново на другой день)  
	
	![рис.6](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW02%20-%20docker/image/im6.jpg)
	
1.	Развернула докер и т.д.    
	
	![рис.7](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW02%20-%20docker/image/im7.jpg)

1.	Создала docker-сеть, подключила созданную сеть к контейнеру сервера Postgres:  
	
	![рис.8](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW02%20-%20docker/image/im8.jpg)	

1.	Создала БД, табличку + данные  
	
	![рис.9](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW02%20-%20docker/image/im9.jpg)	

1.	Удалила контейнер (через Cygwin, т.к. через cmd не видно ИД контейнера)  

	![рис.10](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW02%20-%20docker/image/im10.jpg)	

1.	Снова создала контейнер  
	Проверила наличие данных  	

	![рис.11](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW02%20-%20docker/image/im11.jpg)	

1.	Подключилась со своего компьютера  БД otus чере DBeaver  
	Проверила наличие данных  
	
	![рис.12](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW02%20-%20docker/image/im12.jpg)
 
