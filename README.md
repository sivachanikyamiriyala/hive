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


----------------->>>>>>>PARTITIONS AND BUCKETIZATIONS <<<<<<<<<<----------------------------


create table if not exists student_partition(name string,id int) 
partitioned by (course string,year int) 
row format delimited 
fields terminated by '\t' 
lines terminated by '\n' 
stored as textfile 
;



---static approach 

load data local inpath '/home/siva/work/hive_inputs/student_cse_1.txt' overwrite into table student_partition partition(course='cse',year=1) ;

load data local inpath '/home/siva/work/hive_inputs/student_cse_2.txt' overwrite into table student_partition partition(course='cse',year=2) ;


--Dynamic approach 

insert overwrite into table student_partition partition (course,year) select * from student ;


insert into table student_partition partition (course='ece',year) select name,id,year from student where course='cse' ;

insert into table student_partition partition (course='mca',year=2) select name,id,year from student where course='cse' and year=1 ;




create table if not exists users(name string,id int) 
comment 'table name is users' 
row format delimited 
fields terminated by '\t' 
lines terminated by '\n' 
stored as textfile 
;

load data local inpath '/home/siva/work/hive_inputs/users.txt' overwrite into table users ;


create table if not exists users1(name string,id int) clustered by (id) into 4 buckets ;


create table if not exists users2(name string,id int) clustered by (id) sorted by (id desc) into 5 buckets ;


insert into table users1 select * from users ;

insert into table users2 select * from users ;


select * from users2;


select * from users1;

select * from users1 tablesample (bucket 2 out of 4 on id) ;



select * from users1 tablesample (bucket 2 out of 5 on id) ;

---------joins------

create table if not exists products (name string,id int,course string,year int) 
comment 'table name is products' 
row format delimited 
fields terminated by '\t' 
lines terminated by '\n' 
stored as textfile 
;

load data local inpath '/home/siva/work/hive_inputs/products.txt' overwrite into table products ;


create table if not exists sales (name string,id int,course string,year int) 
comment 'table name is sales' 
row format delimited 
fields terminated by '\t' 
lines terminated by '\n' 
stored as textfile 
;

load data local inpath '/home/siva/work/hive_inputs/sales.txt' overwrite into table sales ;

select products.*,sales.* from products join sales where products.name=sales.name ;


select products.*,sales.* from products left outer join sales where products.name=sales.name ;

select products.*,sales.* from products right outer join sales where products.name=sales.name ;

select products.*,sales.* from products full outer join sales where products.name=sales.name ;


select /* +mapjoin(sales) */ products.*,sales.* from products join sales where products.name=sales.name ;
select /* +mapjoin(products) */ products.*,sales.* from products join sales where products.name=sales.name ;
select /* +mapjoin(sales) */ products.*,sales.* from products left outer join sales where products.name=sales.name ;

select /* +mapjoin(products) */ products.*,sales.* from products right outer join sales where products.name=sales.name ;



----storage

create table if not exists student_textfile(name string,id int,course string,year int) 
comment 'table name is student-text' 
stored as textfile ;


create table if not exists student_sequencefile(name string,id int,course string,year int) 
comment 'table name is student-text' 
stored as sequencefile ;

create table if not exists student_avro(name string,id int,course string,year int) 
comment 'table name is student-text' 
stored as avro ;

create table if not exists student_rcfile(name string,id int,course string,year int) 
comment 'table name is student-text' 
stored as rcfile ;

create table if not exists student_orc(name string,id int,course string,year int) 
comment 'table name is student-text' 
stored as orc ;

create table if not exists student_parquet(name string,id int,course string,year int) 
comment 'table name is student-text' 
stored as parquet ;



insert overwrite table student_textfile select * from student ;


insert overwrite table student_sequencefile select * from student ;

insert overwrite table student_avro select * from student ;

insert overwrite table student_rcfile select * from student ;

insert overwrite table student_orc select * from student ;

insert overwrite table student_parquet select * from student ;



---functions

create table if not exists docs(line string) ;

load data local inpath '/home/siva/work/hive_inputs/demoinput' overwrite into table docs ;

create tabel if not exists siva select explode(split(line,' ')) as word from docs ;

select word,count(1) from w group by word order by word ;



if multi delimiters are ther then 
1.create table single colum table 
2. load the data into the table 
3.Bases on the table create a new table with specific columns 
4.using insert command insert the data into table in selete query use split function 



------ACID

create table if not exists student_acid(name string,id int,course string,year int) 
clustered by (name) into 4 buckets 
stored as orc 
location '/hive/chanikya/student_acid/' 
tblproperties('transactional'='true') ;

load data local inpath '/home/siva/work/hive_inputs/student.txt' overwrite into table student_acid ;

can do insert and update after select query to check 

---------serde 

create table if not exists oldregex(name string,id string,course string,year string) 
row format serde 'org.apache.hadoop.hive.contrib.serde2RegEx' 
with serdeproperties("input.regex"="([^\t]*)()()()") ;


create table if not exists newregex(name string,id int,course string,year int) 
row format serde 'org.apache.hadoop.hive.serde2RegEx' 
with serdeproperties("input.regex"="([^\t]*)()()()") ;

load data local inpath '/home/siva/work/hive_inputs/student.txt' overwrite into table oldregex ;

load data local inpath '/home/siva/work/hive_inputs/student.txt' overwrite into table newregex ;  

----alter

create table test1(name string,id int) ;

alter table test1 rename to mytest ;

alter table mytest add columns (course string) ;
alter table mytest replace columns (name string,course int,id int) ;
alter table mytest change course course string comment 'student course' ;
alter table tablename set tblproperties ('key1'='value1') ;

alter database databasename set dbproperties('key1'='value1') ;


alter database databasename set owner user hadoop ;


alter database databasename set owner role hadoop ;

