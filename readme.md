# Product Management tool
Project Overview

The project involves building a microservices-based architecture consisting of offer-service, product-service, service-discovery, and an API gateway. These services collectively aim to streamline the management of products and offers, ensuring scalability, maintainability, and enhanced user experiences.

Team Member:

Bishal Pudel : 2023SL93025

Harsh Tomar  : 2023SL93050

Microservices in this project:
- microservice-ui (frontend)
- service-registry
- api-gateway
- product-service
- offer-service

Business Use Case Documentation: https://docs.google.com/document/d/1zE-AAtCcXp3k_mMkBKh0Fd4YzmUGAbjdt49Tb9AAs3U/edit?usp=sharing

Swagger Documentation:

Product Microservice:

https://app.swaggerhub.com/apis/watchserviceapi/watch-business/1.0.0#/ForecastTemperature

Offer-Microservice API Documentation:

https://app.swaggerhub.com/apis/2023SL93025/offerservice/1.0.0#/Offer/get_offer

# What is microservice?
Microservice is a modern as well as a popular architecture of designing software application over the last few years. 
There are lots of content on the internet to describe what microservice really is and those are very informative. 
But here I wanna describe it simply, concisely in production style.  

A microservice application is consist of different services where every service is an application which is
  1. **Independently deployable**   
  2. **Independently scalable**  

above two are the key requirements of a microservice application.

In this microservice application here are two service **product-service** and **offer-service** both independently 
deployable and scalable. **They are using two different database but this is not an issue about microservice architecture. 
They can use the same database.**

To expose these two service as microservice architecture we used two other service those are **service-registry** for 
service discovery and **api-gateway** for dynamic service routing as well as load balancing.

# Have a look the workflow
![workflow](readme-images/flow-diagram.png)

 Overall Application Diagram
![image](https://github.com/user-attachments/assets/0135e9de-c63e-4da0-9c04-1ed44d73a020)

# Run the services 

## System configuration prerequisites
### 1. Clone this project
Open terminal and run
````
https://github.com/BITSAPIBP2024/TEAMA13.git
````
In your current directory ``complete-microservice-application`` directory will be created with five different project inside.

### 2. Install Java and Maven
Install java 8 or higher version and Apache Maven 3.6.0 on your system.
Java 11 is installed in my system. This is not an issue. It will work fine in java 8 to java 11.

### 3. Install RabbitMQ
RabbitMQ is Advanced Message Queuing Protocol (AMQP), It should be installed and running in your system where product 
service will be deployed. Though this example is for localhost you need to install in your local computer and by default 
run in ``http://localhost:15672/``  
Username: guest  
Password: guest
  

### 4. Lombok
Lombok plugin should be installed in you IDE otherwise IDE will catch code error. I used Intellij Idea and I had to install 
lombok plugin.

### 5. Install Node, Angular and Angular CLI
In my system I used
````   
Angular CLI: 8.3.25
Node: 12.13.1
Angular: 8.2.14 
````
You need to install Node 12 or higher version and Angular 8 or higher version to run microservice-ui application.

## Run microservice-ui application
It's an angular based user interface application for these microservice frontend. It's not any high functional user interface 
I just tried to consume the backend services from here. You can check all the api just from **Postman**.  

Open terminal run below command to launch microservice-ui application
````
cd microservice-ui/
npm install
ng s --port 3000 --open
````
It will open a new tab in your browser with **http://localhost:3000/store** url as frontend application.
When it opens it calls ``http://localhost:8000/product-service/products`` by default to fetch all product list. 
It will not respond any products because backend services are not started yet.   

![store home](readme-images/store-home.png)

UI application is ready, Now we need to run it's backend applications. 
All the backend applications are developed by spring-boot.

## Run service-registry application
service-registry is the application where all microservice instances will register with a service id. When a service wants 
to call another service, it will ask service-registry by service id to what instances are currently running for that service id. 
service-registry check the instances and pass the request to any active the instance dynamically. This is called service 
discovery. The advantage is no need to hard coded instance ip address, service registry always updates itself, if one instance 
goes down, It removes that instance from the registry. 
- **Eureka** is used to achieve this functionality

Open a new terminal and run below command to launch service-registry
````
cd service-registry/
mvn clean install
mvn spring-boot:run
````
service-registry will launch in http://localhost:8761/  

![service registry](readme-images/service-registry.png)

Right now only service-registry is registered with Eureka. In your system it will show your ip address.  
All the backend application will register here one by one after launching and service-registry will show like this. 

![all registered service](readme-images/all-registered-service.png)

## Run api-gateway application
api-gateway application is the service for facing other services. Just like the entry point to get any service. Receive 
all the request from user and delegates the request to the appropriate service. 
- **Zuul** is used to achieve this functionality

Open a new terminal and run below command to run api gateway
````
cd api-gateway/
mvn clean install
mvn spring-boot:run
````
The application will run in ``http://localhost:8000/``.

api-gateway is configured such a way that we can call product-service and offer-service api through api-gateway.   
Like - when we call with a specific url pattern api-gateway will forward the request to the corresponding service based 
on that url pattern.  

| API             | REST Method   | Api-gateway request                                       | Forwarded service   | Forwarded URL                      |
|-----------------|:--------------|:----------------------------------------------------------|:--------------------|:-----------------------------------|
|Get all products |GET            |``http://localhost:8000/product-service/products``         | product-service     | ``http://localhost:8081/products`` |    
|Add new product  |POST           |``http://localhost:8000/product-service/products``         | product-service     | ``http://localhost:8081/products`` |    
|Update price     |PUT            |``http://localhost:8000/product-service/products/addPrice``| product-service     | ``http://localhost:8081/products`` |    
|Add offer        |POST           |``http://localhost:8000/offer-service/offer``              | offer-service       | ``http://localhost:8082/offer``    |

Above table contains all the used api in this entire application.

If we have multiple instance for product-service like ``http://localhost:8081`` and ``http://localhost:8180``.
So when we call ``http://localhost:8000/product-service/products`` api gateway will forward it to one of the two instance 
of product-service as load balancing in Round-robin order since Zuul api-gateway use Ribbon load balancer.
Api gateway frequently keep updated all available instance list of a service from eureka server.  
  
**you can create as many as instance you need for product-service as well as offer-service api-gateway will handle it smartly.**

So we can say that api-gateway is the front door of our backend application by passing this we need to enter kitchen or 
washroom whatever. [Bad joke LOL]

## Run Product service
Open a new terminal and run below command
````
cd product-service/
mvn clean install
````
The ``mvn clean install`` command will create a ``product-service-0.0.1-SNAPSHOT.jar`` inside ``target`` directory. 
We will run two product service instance by two different port. Run below command in a separate terminal
````
cd target/
java -jar product-service-0.0.1-SNAPSHOT.jar --server.port=8081
````
Above command will run product-service in 8081 port.

Run another instance of product-service in 8180 port by running below command in another terminal
````
java -jar product-service-0.0.1-SNAPSHOT.jar --server.port=8180
````
After few seconds you can see there are 2 instance running for product-service which is registered with Eureka server in http://localhost:8761/

***Note:** Both instance are running as separate application but they are using same database.*

Access product-service data source console in browser by
`localhost:8081/h2`  
To connect product data source h2 console use below credentials   
JDBC URL  : `jdbc:h2:~/product`  
User Name : `root`  
Password  : `root`  

Check product table. Right now there is no products.  
Let's add a new product by calling `localhost:8000/product-service/products` **POST** endpoint with below body in postman.
````
{
	"productCode": "TW1",
	"productTitle": "Titan",
	"imageUrl": "https://staticimg.titan.co.in/Titan/Catalog/90014KC01J_1.jpg?pView=pdp",
	"price": 30
}
````
Or in microservice-ui press ``Add New Product`` button and fill the pop-up window with above value then press ``Add`` to 
add new product.   

![add new product](readme-images/add-new-product.png)

After adding new product the window will be refreshed and you will see like this   

![after adding one product](readme-images/added-first-product.png)
maf 
Let's add two more new products

````
{
	"productCode": "FW1",
	"productTitle": "Fastrack",
	"imageUrl": "https://staticimg.titan.co.in/Fastrack/Catalog/38051SP01_2.jpg",
	"price": 30
}
````

````
{
	"productCode": "RW1",
	"productTitle": "Rolex",
	"imageUrl": "https://www.avantijewellers.co.uk/images/rolex-watches-pre-owned-mens-rolex-oyster-precision-vintage-watch-p3003-7660_medium.jpg",
	"price": null
}
````
So far our browser window like this  

![all added products](readme-images/all-product-added.png)

If you check product table there is three product added with no **discount_offer**. We will add **discount_offer** by 
sending an event notification from other offer-service application.

One additional information, I have not added any price when adding *Rolex*. Price can be added or updated later after 
adding any product by ``Add Price`` button.

## Run Offer service
Open separate terminal and run
 ````
 cd offer-service/
 mvn clean install
 mvn spring-boot:run
 ````
Application will run on ``http://localhost:8082/``

Access it's data source console in browser by
`localhost:8082/h2`  
To connect offer data source h2 console use below credentials  
JDBC URL  : `jdbc:h2:~/offer`  
User Name : `root`  
Password  : `root`

Check offer table and there is no offer right now.  
Let's add a offer for *product_id = 1* by calling `http://localhost:8000/offer-service/offer` POST endpoint with below body in postman
````
{
	"productId": 1,
	"discountOffer": 10
}
````
Or in microservice-ui press ``Add Discount`` button and fill the pop-up window with above value then press ``Add`` to 
add discount for *product_id = 1*  

![add offer](readme-images/add-offer.png)

If you check offer table there is an offer recorded for *product_id = 1*  and browser will show  

![offer added](readme-images/offer-added.png)

Now you are seeing *Payable: $27* for *product_id = 1* after calculating discount.


# Copyright & License

MIT License, see the link: [LICENSE](https://github.com/hnjaman/complete-microservice-application/blob/master/LICENSE) file for details.
