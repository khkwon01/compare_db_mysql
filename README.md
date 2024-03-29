# MySQL DB 성능 비교

MySQL 8.0.33과 Maria 10.8.8 버전 성능을 sysbench로 테스트 했을 경우 성능 비교     

## 1. 테스트 환경
1) HW 정보 : cpu 1, mem 8GB, network 1Gbps
2) OS 버전 : centos 7.9.2009 (core)    
참고 : mariadb는 oracle linux 8.0에 설치 오류로 인해 설치가 안됨.
3) my.cnf 구성    
3.1) MySQL 설정
```
[mysql]
socket=/mysql/mysql8/mysql.sock

[mysqld]
datadir=/mysql/mysql8/data
socket=/mysql/mysql8/mysql.sock
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0
# Settings user and group are ignored when systemd is used.
# If you need to run mysqld under a different user or group,
# customize your systemd unit file for mariadb according to the
# instructions in http://fedoraproject.org/wiki/Systemd

[mysqld_safe]
log-error=/mysql/mysql8/log/mysql.log
pid-file=/mysql/mysql8/mysql.pid

#
# include all files from the config directory
#
!includedir /etc/my.cnf.d
```
3.2) Mariadb 설정
```
[mysql]
socket=/mysql/maria/mysql.sock
[mysqld]
datadir=/mysql/maria/data
socket=/mysql/maria/mysql.sock
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0
# Settings user and group are ignored when systemd is used.
# If you need to run mysqld under a different user or group,
# customize your systemd unit file for mariadb according to the
# instructions in http://fedoraproject.org/wiki/Systemd

[mysqld_safe]
log-error=/mysql/maria/log/mariadb.log
pid-file=/mysql/maria/mariadb.pid

#
# include all files from the config directory
#
!includedir /etc/my.cnf.d
```
3) 성능 테스트툴 설치 (sysbench)   
yum install sysbench  (1.0.17)
4) MySQL 계정 구성     
4.1) MySQL 설정    
create user systest identified by 'Welcome1';    
```# MySQL 8.0 계정과 sysbench 호환을 위해 아래와 같이 8.0 이전 인증 방식으로 설정, cache_sha2를 사용할 경우 오류발생```    
alter user systest identified with mysql_native_password by 'Welcome1';      
create database sysbench;    
grant all privileges on sysbench.* to systest;    
4.2) Mariadb 설정    
create user systest identified by 'Welcome1';    
create database sysbench;    
grant all privileges on sysbench.* to systest;    
     
## 2. 테스트 수행
1) sysbench 명령어
- prepare
```
sysbench --db-driver=mysql --time=50 --threads=100 --report-interval=20 --mysql-host=127.0.0.1 --mysql-port=3306 --mysql-user=systest --mysql-password="Welcome1" --mysql-db=sysbench --tables=20 --table_size=1000000 oltp_read_write --db-ps-mode=disable prepare
```
- run
```
sysbench --db-driver=mysql --time=50 --threads=100 --report-interval=20 --mysql-host=127.0.0.1 --mysql-port=3306 --mysql-user=systest --mysql-password="Welcome1" --mysql-db=sysbench --tables=20 --table_size=1000000 oltp_read_write --db-ps-mode=disable run
```
- clean
```
sysbench --db-driver=mysql --time=50 --threads=100 --report-interval=20 --mysql-host=127.0.0.1 --mysql-port=3306 --mysql-user=systest --mysql-password="Welcome1" --mysql-db=sysbench --tables=20 --table_size=1000000 oltp_read_write --db-ps-mode=disable cleanup
```
2) sysbench test 결과    
트랜잭션이 read/write 환경에서 여러가지 조건(thread 50, 100등)으로 여러번 테스트를 진행해도 MySQL 1.5 ~ 2배까지 성능이 더 좋게 측정됨.    
(트랜잭션이 read만일 경우에는 MySQL이 약간 좋게 표시됨 tps기준 MySQL : 691.15 per sec, MariaDB : 616.84 per sec)    
2) sysbench test 결과    
- MySQL
![image](https://github.com/khkwon01/comp_db/assets/8789421/92687705-5a29-441c-a8f9-4181e0a01fad)
![image](https://github.com/khkwon01/comp_db/assets/8789421/52d952e4-6671-46f5-96cc-106935c9cb7b)

- MariaDB
![image](https://github.com/khkwon01/comp_db/assets/8789421/4a54bea3-c13d-484d-b40e-dca6ba13a899)
![image](https://github.com/khkwon01/comp_db/assets/8789421/e939054c-df55-4644-b969-5ef479c5db46)

