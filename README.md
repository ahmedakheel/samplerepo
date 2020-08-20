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
12. On the private instance, install postgresql (apt install postgres). Ensure it is running as described in: 
