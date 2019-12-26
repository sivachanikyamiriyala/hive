Hive practice 

1.creation of database 
2.creation of tables 
3.describe database and tables 
4.alter database and table 
5.delete database and delete table 
6.



create database if not exists chanikya
comment 'database name is chanikya'
location '/hive/chanikya' 
with dbproperties('key1'='value1') ;

use chanikya ;

create table if not exists student1(name string,id int,course string,year int) 
comment 'table name is student1' 
row format delimited 
fields terminated by '\t' 
lines terminated by '\n' 
stored as textfile 
;

create table if not exists student2(name string,id int,course string,year int) 
comment 'table name is student2' 
row format delimited 
fields terminated by '\t' 
lines terminated by '\n' 
stored as textfile 
;

create external table if not exists student3(name string,id int,course string,year int) 
comment 'table name is student3' 
row format delimited 
fields terminated by '\t' 
lines terminated by '\n' 
stored as textfile 
;

create temporary table if not exists student4(name string,id int,course string,year int) 
comment 'table name is student4' 
row format delimited 
fields terminated by '\t' 
lines terminated by '\n' 
stored as textfile 
;



load data local inpath '/home/siva/work/hive_inputs/student.txt' overwrite into table student1 ; 

load data local inpath '/home/siva/work/hive_inputs/student.txt' overwrite into table student2 ; 


load data local inpath '/home/siva/work/hive_inputs/student.txt' overwrite into table student2 ; 


dfs -mkdir -p /home/siva/work/hive_inputs/ ;

dfs -put /home/siva/work/hive_inputs/student.txt /home/siva/work/hive_inputs/ ; 

load data inpath /home/siva/work/hive_inputs/student.txt  into table student2 ;


dfs -put /home/siva/work/hive_inputs/student.txt /home/siva/work/hive_inputs/ ; 


--external tables 

create table if not exists studenttext1(name string,id int,course string,year int) 
comment 'table name is studenttext1' 
row format delimited 
fields terminated by '\t' 
lines terminated by '\n' 
stored as textfile 
;


create table if not exists studenttext2(name string,id int,course string,year int) 
comment 'table name is studenttext2' 
row format delimited 
fields terminated by '\t' 
lines terminated by '\n' 
stored as textfile 
location '/hive/externaldata' 
tblproperties('key1'='value') 

;

create table if not exists student
(name string comment 'student name',
id int comment 'student id',
course string comment 'student course', 
year int comment 'student year'
)
comment 'student is the name of the table' 
row format delimited 
fields terminated by '\t' 
lines terminated by '\n' 
stored as textfile 
location '/hive/chanikya/student' ;

load data local inpath '${env:HOME}/work/hive_inputs/student.txt' overwrite into table student ;



create table if not exists studentdata
(name string comment 'student name',
subjects array<string> comment 'student subjects', 
marks map<string,int> comment 'student marks',
address struct<loc:string comment 'student location',pincode:int comment 'student pincode',city:string comment 'student city'> comment 'add',
details uniontype<double,int,array<string>,struct<a:int,b:string>,boolean> comment 'student details'
)
comment 'studentdata is the table name' 
row format delimited 
fields terminated by '#' 
collection items terminated by ','
map keys terminated by ':'
lines terminated by '\n' 
stored as textfile 
location '/hive/chanikya/studentdata' 
tblproperties('key1'='value1') 
;

load data local inpath '${env:HOME}/work/hive_inputs/studentdata.txt' overwrite into table studentdata ;

select * from studentdata ;
select name,subjects[0],marks['math'],address.pincode,details from studentdata ;


insert overwrite local directory '/home/siva/chanikya1' select * from studentdata ;

insert overwrite local directory '/home/siva/chanikya2' select name,subjects[0],marks['math'],address.pincode,details from studentdata ;

insert overwrite local directory '/home/siva/chanikya3' row format delimited fields terminated by '#' collection items terminated by ',' map keys terminated by ':' lines terminated by '\n' select * from studentdata ;




















































