# 🖥️  Launch EC2 Instance
- launch an Ubuntu instance.
- name it as ``EKS-main``
# ☸️  Create EKS Cluster
- Create EKS cluster
- Wait 15–20 minutes for the cluster to become active.
- Ensure the worker nodes are created.
![Screenshot 2025-07-04 110411](https://github.com/user-attachments/assets/075589c8-f40c-4060-a316-747db4b375f4)

# 🛢️  Create RDS (MariaDB) Database
- go to EC2, you will see two  eks worker node (instances)
- name one of them as ``ekscluster-database-node``

### Create RDS Database
- Database Creation Method: Standard create
- Engine: MariaDB
- Templates: Free tier
- DB instance identifier: database-1 (default)
- Master username: admin (default)
- Password: Passwd123$
- Connectivity: Connect to an EC2 compute resource
- EC2 instance: choose the database instance
- VPC security group (firewall): choose existing 
- Additional VPC security group: eks-cluster-sg-k8s-ekscluster
- create Database.
# 🔌 Connect to Worker Node and Setup DB
- edit security group of instance ekscluster-database-node
- Remove the previous rulees and add new rule -> allow all traffic
- SSH into one of the EKS worker nodes.(database instance)
![Screenshot 2025-07-04 113037](https://github.com/user-attachments/assets/c9619859-5bb1-491a-b2c0-cefed4bc7dad)


- cheack the OS
```
cat /etc/os-release
```
- install mariaDB (for Amazon linux OS)
```
yum update
yum install mariadb105-server -y
```
- install mariaDB ( for Ubuntu OS)
```
sudo apt update -y
sudo  apt install mariadb-server -y
```
```
systemctl start mariadb
```
```
systemctl enable mariadb
```
- log in to Database
```
mysql -h <database_endpoint> -u admin -pPasswd123$
```
- replasce the database endpoint, username and password with your actual values.
- run following commands to create database
```
show databases;
```
```
create database  studentapp;
```
```
use studentapp;
```
- run this query to create table
```
 CREATE TABLE if not exists students(student_id INT NOT NULL AUTO_INCREMENT,  
	student_name VARCHAR(100) NOT NULL,  
	student_addr VARCHAR(100) NOT NULL,   
	student_age VARCHAR(3) NOT NULL,      
	student_qual VARCHAR(20) NOT NULL,     
	student_percent VARCHAR(10) NOT NULL,   
	student_year_passed VARCHAR(10) NOT NULL,  
	PRIMARY KEY (student_id)  
);
```
```
show tables;
```
- logout from the database
```
exit
```
# 🔐  Update Security Group for EKS Cluster
- Go to EKS cluster's Security Group.
- Remove existing inbound rules.
- Add new inbound rule:
   - Type: All Traffic
   - 
   - Source: 0.0.0.0/0
# 📥 Clone the Project on EC2
- back to main server(EKS-main)
- install git
```
  install git -y
```
- clone the repository
```
git clone https://github.com/abhipraydhoble/Project-StudentApp.git
```
```
 cd Project-StudentApp
```
```
cd Studentapp-project/BE
```
- edit context.xml file
```
sudo vim context.xml
```
- update it with your ``Database endpoint``, ``password`` and ``uasename``
```
    <!-- Default set of monitored resources. If one of these changes, the    -->
    <!-- web application will be reloaded.                                   -->
    <WatchedResource>WEB-INF/web.xml</WatchedResource>
    <WatchedResource>${catalina.base}/conf/web.xml</WatchedResource>
    <Resource name="jdbc/TestDB" auth="Container" type="javax.sql.DataSource"
               maxTotal="100" maxIdle="30" maxWaitMillis="10000"
               username="admin" password="Passwd123$" driverClassName="com.mysql.jdbc.Driver"
               url="jdbc:mysql://database-1.cja8geg0in7p.ap-southeast-1.rds.amazonaws.com:3306/studentapp"/>
    <!-- Uncomment this to disable session persistence across Tomcat restarts -->
    <!--
    <Manager pathname="" />
    -->
```
![Screenshot 2025-07-02 122357](https://github.com/user-attachments/assets/7b99748b-b066-4d90-af00-9fd3557f4afe)

# 🐳  Build and Push Backend Docker Image
- install docker (for Amazon linux OS)
```
sudo apt update -y
sudo apt install docker.io -y
sudo systemctl start docker
sudo usermod -aG docker ubuntu
newgrp docker
sudo chmod 777 /var/run/docker.sock
```
- install Docker (for Ubuntu OS)
```
sudo apt update -y
sudo apt install docker.io -y
sudo systemctl start docker
sudo usermod -aG docker ubuntu
newgrp docker
sudo chmod 777 /var/run/docker.sock
```
- build dcoker image
```
docker build -t <dockerhub_username>/ be .
```
- check images
```
docker images
```
- log in to dockerhub
```
docker login -u <dockerhub_username>
```
- push the image to dockerhub
```
docker push <dockerhub_username>/be
```
- now edit ``deployment.yaml``
```
 sudo vim deployment.yaml
```
- replace the image name with your docker image 
```
spec:
  8   template:
  9     metadata:
 10       labels:
 11         app: studentapp
 12     spec:
 13       containers:
 14         - name: studentapp-container
 15           image: tejaswinism/be
 16           ports:
 17             - containerPort: 8080
 18           resources:
 19           
 ```
![Screenshot 2025-07-02 125312](https://github.com/user-attachments/assets/b467f94e-4ed5-4df1-a1cf-bf4d2ea10468)

- deploy the EKS
```
kubectl apply -f .
```
- get sevices
```
kubectl get svc
```
- cpoy the load balnacer link
# 🌐  Access Backend
- Copy the EXTERNAL-IP of backend LoadBalancer.
- Access: http://<svc-lb-link>:8080/student





