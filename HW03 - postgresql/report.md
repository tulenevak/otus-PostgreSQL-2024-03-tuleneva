**Отчет ДЗ №3 Установка PostgreSQL**  
  

1. На моей ВМ стоял PostgreSQL 14
	Обновила до 15
	*sudo apt update && sudo apt upgrade -y -q && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo apt -y install postgresql-15*
	
	![рис.1](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW03%20-%20postgresql/image/im1.jpg)

1. Подключилась к БД *sudo -u postgres psql -p 5432*
	Выполнила запрос к таблице, созданной на 1м занятии *select * from persons;*
	чтобы убедиться в подключении.
	Далее создала таблицу test *create table test(c1 text);*
	*insert into test values('1');*

	![рис.2](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW03%20-%20postgresql/image/im2.jpg)
 
1. Остановила кластер
	*sudo systemctl stop postgresql@15-main*
	
1. Содала, подключила диск на 10Gb
 
	![рис.3](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW03%20-%20postgresql/image/im3.jpg)
	![рис.4](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW03%20-%20postgresql/image/im4.jpg)

1. Инициализация диска по инструкции
	*sudo apt update*

	![рис.5](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW03%20-%20postgresql/image/im5.jpg)
 
	*sudo apt install parted*
	*sudo parted -l | grep Error*

	![рис.6](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW03%20-%20postgresql/image/im6.jpg)

	Lsblk
	sudo parted /dev/sdb mklabel gpt
	sudo parted /dev/sdb mklabel msdos
	sudo parted -a opt /dev/sdb mkpart primary ext4 0% 100%
	Lsblk
	sudo mkfs.ext4 -L datapartition /dev/sdb1

	![рис.7](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW03%20-%20postgresql/image/im7.jpg)

	sudo e2label /dev/sdb1 newlabel
	lsblk –fs

	![рис.8](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW03%20-%20postgresql/image/im8.jpg)

	Подмонтирование файловой системы и сохранение в файл информации: автоматическое подключение диска при загрузке системы, с определенными параметрами

	![рис.9](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW03%20-%20postgresql/image/im9.jpg)

1. Перезагрузила инстанс, убедилась что диск примонтирован.
	Меняю владельца каталога и переношу содержимое
	*sudo chown -R postgres:postgres /mnt/data/*
	*sudo mv /var/lib/postgresql/15 /mnt/data*

1. Проверила запустился ли кластер

	![рис.10](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW03%20-%20postgresql/image/im10.jpg)

	Ошибка
	
1. Посмотрела какие файлы есть sudo  ls /etc/postgresql/15/main
	Выбрала нужный 
	*sudo nano /etc/postgresql/15/main/postgresql.conf*
	Исправила путь в конфигурационном файле на новый, куда было перенесено содержимое из var/lib

	![рис.11](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW03%20-%20postgresql/image/im11.jpg)

1. Запустила кластер sudo -u postgres pg_ctlcluster 15 main start
	Сразу не сделала скрин (закрыла случайно), при повторном запуске говорит, что кластер запущен
 
	![рис.12](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW03%20-%20postgresql/image/im12.jpg)

1. Проверила, что подключение к БД есть, и данные выводятся
	
	![рис.13](https://github.com/tulenevak/otus-PostgreSQL-2024-03-tuleneva/tree/main/HW03%20-%20postgresql/image/im13.jpg)
 
