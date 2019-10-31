Table of Contents
=================
 * [Eshopool: Android Application Development](#eshopool-android-application-development)
    * [1 Introduction](#1-introduction)
       * [1.1 System Architecture](#11-system-architecture)
       * [1.2 Use Case](#12-use-case)
    * [2 Task Description](#2-task-description)
       * [2.1 Android Application - eShopool](#21-android-application---eshopool)
       * [2.2 Web Service](#22-web-service)
          * [2.2.1 SOAP Service](#221-soap-service)
          * [2.2.1 Tomcat Service](#221-tomcat-service)
       * [2.3 Master-Slave Architecture &amp; Middleware](#23-master-slave-architecture--middleware)
       * [2.4 Database](#24-database)
          * [2.4.1 JDBC Tool Class](#241-jdbc-tool-class)
          * [2.4.2 MySQL Database](#242-mysql-database)
    * [3 Reference](#3-reference)

      
# Eshopool: Android Application Development
[![Badge](https://img.shields.io/badge/Website-eShopool-orange.svg)](https://eshopool.com)
[![course](https://img.shields.io/badge/CNSCC-311-inactive)](http://www.lusi.lancaster.ac.uk/CoursesHandbook/ModuleDetails/ModuleDetail?yearId=000114&courseId=016887)
[![LICENSE](https://img.shields.io/hexpm/l/plug)](https://github.com/eShopool/Android-Application/blob/master/LICENSE)

## 1 Introduction
### 1.1 System Architecture

Our project establishes a second-hand trading platform, called ‘eShopool’, which realizes properties of the distributed system. The front-end provides a user-friendly UI and can be installed on Android mobile devices. There are eight servers in our system, including four database servers, two soap servers and two tomcat servers. Database servers consist of all information of users, commodities and trade records, and the Tomcat stores all images related to the system such as users’ profile photos and commodities’ pictures. The multi-Mater-Slave Architecture guarantees Read/Write Splitting as well as load-balance features, and a middleware is designed to support that. SOAP servers implement necessary operations for database servers, so they provide functions’ interfaces for frontend. In addition, the asymmetric encryption algorithm was applied to solve the secure problem of data transfer. As a distributed system, several databases servers have the replication of all records and synchronize the data.   

<p align=center>
<img src="https://github.com/TechSang/Project-Overview/blob/master/image/Main%20Architecture.jpg" width="550" />
</p>  

### 1.2 Use Case  

<p align=center>
<img src="https://github.com/TechSang/Project-Overview/blob/master/image/Usecase.jpg" width="600" />
</p>  

## 2 Task Description
### 2.1 Android Application - eShopool
We build an Android application as a client to execute all function in our project. It’s named as ‘eshopool’ which is a second-hand trading platform. User can register on App and upload his second-hand goods or browse commodities in the platform, and other operations like purchasing and selling.  

* In the registration process, users need to input phone number, used to receive the verification code, and password. The back-end will automatically generate public key and private key that will be used to check user info in future operations.  
* In the creating goods procedure, users can upload commodity’s image, price, amount and description.  
* In the purchasing procedure, users can input commodity’s number and get a feedback of trade information after successful payment.  

### 2.2 Web Service
#### 2.2.1 SOAP Service
We adopt SOAP protocol in the back-end which receives and replies request from front-end and loads the database and other resources from a distributed storage system.  

The NetBeans operating environment is set up and all Web Servers were deployed. Then, the method in the front-end will be implemented to call Web Servers. According to different requirements, the SOAP request message is generated, sent to glassfish server. At the same time, the corresponding method is called, and the result is returned to the front-end. Secondly, operations in JDBC class are embedded into Web Servers, and are called to realize the communication between Web Servers and the database, then modify the remote database.  

Next, generate and store asymmetric key pair using the encryption method embedded in Web Service, as well as transmit public key to the client. All message transmission related to Web Service and clients is encrypted by RSA, an asymmetric cryptographic algorithm, and Base64 encryption, which is negotiated well in the development period in order to protect users’ privacy and ensure information security. Concretely, Web Service is expected to generate paired keys –private key and public key for users when they first create their accounts. From then on, user holds their public keys and every message transmission between clients and Web Service will be encrypted by first this key and then Base64 encoder respectively. As for Web Service side, it applies a Base64 decoder and private key to decrypt the message. After that, Web Service will carry out operations according to the message from clients, such as returning query results, updating information of database, creating a new record for database and so on. Deservedly, the messages from the Web Service to clients are also encrypted in the same way. To sum up, the whole encryption system we designed is highly available, safe and robust for users.  

In addition, we add lock to the buy item operation to make sure that the same item can be purchased by only one user at the same time. Every time the user calls the buy item method, we will first check whether the item is in the purchasing process of another user. If the item is free to buy, add lock to this item first and then conduct the purchase method. At last, we will release the lock on the item. For the system problems that may happen during the buy item process, we also write exception logs and store it into local computer system. The Web Service will read the exception log every time recover from the system crashes and release the lock of that item preparing for next time’s operation. What’s more, we also synchronize the exception logs into another web server and enable it to read the log and recover the database from incorrect content.  

<p align=center>
<img src="https://github.com/TechSang/Project-Overview/blob/master/image/Asymmetric%20Cryptosystem.jpg" width="600" />
</p>  

#### 2.2.1 Tomcat Service
We also use Apache Tomcat, which is an open source implementation of the Java Servlet to receive and reply requirements from front-end and do the connection between the user, database and web photo server.  

Due to the feature limitation of SOAP and Glassfish, we find that they are difficult to load the local file and send it to the user. However, in our program, it involves a lot of operations with photos, we have to find another solution. So, we decide to use Tomcat and Servlet which are similar to the Glassfish and SOAP solve this problem at last.  

The main operations which are involved user, database and photo server are updating portrait, updating commodity pictures and getting commodity photos for a specific class. For the first two operations, the user will send a requirement to the server with its name which is generated by its UUID and the photo byte stream. When the photo server receiving the requirement, the server will change it from the byte stream to the picture and save it in the specified path in the server.
For the operation of searching for all commodities from a specific class, the photo server will connect to the database by JDBC and search for relevant information. And then, the server will generate a string result list with all commodities which is stored into a txt file at first, and then, this txt file will send back to the user. The user will find photos from the server by the given commodity list eventually.  

Besides, we also achieve the operation of Fuzzy Research. The user can search for all commodities by just a little word. When the user sends this kind of requirement to the web photo server, the server will first search from the database first to find if there are eligible commodities. And then, the photo server will return all relevant results to the server which is similar to the previous operation.  

### 2.3 Master-Slave Architecture & Middleware
The server's ability to withstand pressure is quite limited. When load pressure exceeds its limitation, it becomes clear that a single server may not response properly, and more servers are needed. To ensure the timely synchronization of data between servers, master and slave technical solutions are needed. Master-master model is also called AB replication in MySQL. Simply put, two machines A and B are master and slave. If a client writes data on A, the other machine B will also write data along with it, and the data on both servers is synchronized in real time.  

In real case, there are more read operations than write operations. We hope to improve the database's read performance and eliminate the conflicts between read and write operations so as to improve the write performance. Thus, we applied the master-slave architecture to implement the ‘Read/Write Splitting’.  

We also adopt ‘load balance’ function to distribute users’ requests to different servers to improve the performance and reliability of applications and databases. We use the Weighted Round-Robin algorithm. First, all databases will be accessed, the real-time latency of each node will be measured, and a weight will be calculated according to it. For each access attempt, the node with the largest current weight will be chosen as the database to provide service. Any nodes that fail to serve more than 3 times will be treated as a lost database, deleted from the valid list.  

Therefore, our distributed architecture is composed of “2-master and 2-slave servers” to achieve the goal of high robustness and performance. A middleware is written to implement “load balance” and “Read/ Write Splitting” functions for the system.  

<p align=center>
<img src="https://github.com/TechSang/Project-Overview/blob/master/image/Multi%20Master-Slave%20Architecture.jpg" width="600" />
</p>  
  
  
### 2.4 Database
#### 2.4.1 JDBC Tool Class
JDBC bridge a gap between Java and the database and has an ability to execute SQL statements in Java IDE. We designed a JDBC tool class which can be used to call MySQL and fulfill remote communication. It can provide operations like adding, deleting and modifying any tables, by ‘executeUpdate’, in the database. And give a return value of what you want to query by using ‘getResult’ to invoke the methods you wrote. Thus, we can get the information in the database, for example, the price and amount of a commodity. Or decrease the number of items available for sale and log the trade info after the transaction.  

<p align=center>
<img src="https://github.com/TechSang/Project-Overview/blob/master/image/JDBC.jpg" width="600" />
</p>  
  
  
#### 2.4.2 MySQL Database
Our database consists of ‘user’, ‘commodity’ and ‘trade_info’ table. The ‘user’ table has attributes of the customer and merchant. It stores user ID, phone number, password, public/private key, credit and several items. The ‘commodity’ table describes the merchandise news that seller updated. It stores commodity ID, name, price, number, category, merchant ID and several items. The ‘trade_info’ table records the trade info of transaction process. It stores buyer & seller & commodity ID, amount, date, etc. The connection between them is the ID which is designed as primary key and increase automatically one by one.  

<p align=center>
<img src="https://github.com/TechSang/Project-Overview/blob/master/image/Database.jpg" width="600" />
</p>  
  
  
## 3 Reference
-	Matthews, M., Cole, J., & Gradecki, J. D. (2003). MySQL and Java developer's guide. John Wiley & Sons.  
-	Hu, W. C., Kaabouch, N., & Yang, H. J. (2019). An Empirical Study of Mobile/Handheld App Development Using Android Platforms. In Advanced Methodologies and Technologies in Network Architecture, Mobile Computing, and Data Analytics (pp. 848-862). IGI Global.  
-	Cheon, J. H., Kim, D., Lee, J., & Song, Y. (2018, September). Lizard: Cut off the tail! A practical post-quantum public-key encryption from LWE and LWR. In International Conference on Security and Cryptography for Networks (pp. 160-177). Springer, Cham.  
-	Krishnan, M., Dinker, D., & George, J. (2014). Slave consistency in a synchronous replication environment. U.S. Patent No. 8,694,733. Washington, DC: U.S. Patent and Trademark Office.  
-	Truica, C. O., Boicea, A., & Radulescu, F. (2013). Asynchronous replication in Microsoft SQL Server, PostgreSQL and MySQL. In International Conference on Cyber Science and Engineering (CyberSE’13) (pp. 50-55).  
-	Li, W. L., Zhu, R. X., Kang, J., Tao, L., & Cai, G. H. (2014). A design of improved base64 encoding algorithm based on fpga. Applied Mechanics and Materials, 513-517, 2220-2223.  
-	Nurdiyanto, H., Rahim, R., & Wulan, N. (2017). Symmetric stream cipher using triple transposition key method and base64 algorithm for security improvement. Journal of Physics: Conference Series, 930, 012005.  
-	Josefsson, S. (2003). The Base16, Base32, and Base64 Data Encodings. RFC Editor.  
-	Brittain, J., & Darwin, I. F. (2007). Tomcat: The Definitive Guide: The Definitive Guide. " O'Reilly Media, Inc.".  
-	Kurniawan, B. (2015). Servlet & JSP: A Tutorial. Brainy Software Inc.
