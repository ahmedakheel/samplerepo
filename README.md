Instructions to create the environment described in https://github.com/servian/TechChallengeApp

1. Login to Oracle Cloud Infrastructure (OCI) console. 
2. Create a Virtual Cloud Network (VCN) called vcn4servian with 10.0.0.0/16 CIDR block and enable it to use DNS label resulting in domain names vcn4servian.oraclevcn.com
3. Create a regional private subnet with 10.0.1.0/24 CIDR. This will host the instance running postgres
4. Create a regional public subnet with 10.0.2.0/24 CIDR. This will host the web/application server
5. create a public and a private route tables, and assign to appropriate subnets.
6. Create a public and a private security lists and assign to appropriate subnets
7. Create an internet gateway and a nat gateway. Internet gateway assigned to public subnet. Nat gateway assigned to private subnet.
8. Route table for private subnet has dest 0.0.0.0/0 targeting NAT gateway
9. Route table for public subnet has dest 0.0.0.0/0 targeting igw (internet gateway)
8. add security lists to both subnets to allow port 22/ssh traffic ingress (stateful) - from 0.0.0.0/0 for public subnet and from 10.0.2.0/24 for private subnet.
9. add security list rule stateful egress for postgres port 5432 to private subnet. add security list stateful ingress rule for port 5432 from public subnet, in private subnet.
9. Launch Ubuntu LTS 20.04 instances, one in each subnet, using rsa public keys. putty on laptop will use private key. copy the key to linux instance in pub subnet as id_rsa under .ssh (permissions 400), so you can ssh to instance in private subnet - ssh user is ubuntu
10.On the public instance, wget https://github.com/servian/TechChallengeApp/archive/master.zip and unzip (apt install unzip if it doesnt exist yet).
11.On the public instance, install golang (apt install golang)
12. On the private instance, install postgresql (apt install postgres). Ensure it is running as described in: https://github.com/ahmedakheel/samplerepo/blob/master/checkpgsqlstatus.txt
13. On the postgresql server, login as postgres user and create database "app" as

root@servianprivinst:~# su - postgres
postgres@servianprivinst:~$ createdb app


change the postgres user password as shown
postgres@servianprivinst:~$ psql postgres
psql (12.2 (Ubuntu 12.2-4))
Type "help" for help.

postgres=# \password postgres
Enter new password:

### Welcome123 is the password ####


14. update conf.toml as shown

ubuntu@servianpubinst:~/TechChallengeApp-master/dist$ cat conf.toml
"DbUser" = "postgres"
"DbPassword" = "Welcome123"
"DbName" = "app"
"DbPort" = "5432"
"DbHost" = "10.0.1.3"
"ListenHost" = "*"
"ListenPort" = "3000"


15. run build.shown

ubuntu@servianpubinst:~/TechChallengeApp-master$ ./build.sh
go: finding github.com/spf13/viper v1.0.2
go: finding github.com/fsnotify/fsnotify v1.4.7
go: finding golang.org/x/sys v0.0.0-20180724212812-e072cadbbdc8
go: finding github.com/hashicorp/hcl v0.0.0-20180404174102-ef8a98b0bbce
go: finding github.com/magiconair/properties v1.8.0
go: finding github.com/mitchellh/mapstructure v0.0.0-20180715050151-f15292f7a699
go: finding github.com/pelletier/go-toml v1.2.0
go: finding github.com/spf13/afero v1.1.1
go: finding golang.org/x/text v0.3.0
go: finding github.com/spf13/cast v1.2.0
go: finding github.com/spf13/jwalterweatherman v0.0.0-20180109140146-7c0cea34c8ec
go: finding github.com/spf13/pflag v1.0.1
go: finding gopkg.in/yaml.v2 v2.2.1
go: finding github.com/gorilla/mux v1.6.2
go: finding github.com/lib/pq v0.0.0-20180327071824-d34b9ff171c2
go: finding github.com/spf13/cobra v0.0.3


16. Try the updatedb command

ubuntu@servianpubinst:~/TechChallengeApp-master/dist$ ./TechChallengeApp updatedb
Dropping and recreating database: app
DROP DATABASE IF EXISTS app
dial tcp 10.0.1.3:5432: connect: no route to host


***** As seen above, it fails, not being able to connect. postgresql.conf and hba.conf files were updated and postgres was restarted - as shown:


root@servianprivinst:~# diff /etc/postgresql/12/main/postgresql.conf /etc/postgresql/12/main/postgresql.conf-ORIG
63d62
< listen_addresses = '*'
root@servianprivinst:~# diff /etc/postgresql/12/main/pg_hba.conf /etc/postgresql/12/main/pg_hba.conf-ORIG
93d92
< host    all             all              0.0.0.0/0              md5
root@servianprivinst:~#

On both servers,update the iptables v4 rules files to include the rules mentioned in the link at the top: https://github.com/ahmedakheel/samplerepo/blob/master/iptablerules.txt


then run systemctl start iptables.service on both servers

17. Run the TechChallengeApp

ubuntu@servianpubinst:~/TechChallengeApp-master/dist$ ./TechChallengeApp updatedb
Dropping and recreating database: app
DROP DATABASE IF EXISTS app
CREATE DATABASE app
WITH
OWNER = postgres
ENCODING = 'UTF8'
LC_COLLATE = 'en_US.utf8'
LC_CTYPE = 'en_US.utf8'
TABLESPACE = pg_default
CONNECTION LIMIT = -1
TEMPLATE template0;
Dropping and recreating table: tasks
DROP TABLE IF EXISTS tasks CASCADE
CREATE TABLE tasks ( id SERIAL PRIMARY KEY, completed boolean NOT NULL, priority integer NOT NULL, title text NOT NULL)
Seeding table with data


http://129.146.174.182:3000 gives the below page..

add and delete tasks

updated page below

