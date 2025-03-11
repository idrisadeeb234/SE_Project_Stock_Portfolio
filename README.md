We have a frontend and a backend in different folders in the spring project
Maven/Gradle are build tools
JDBC -> Database connectivity
Hibernate (won't be taught)

Inversion of control -> Control of object creation given to someone else; it is a philosophy; dependency injection is the
                        actual implementation of IOC.
                        IOC --> Principle
                        Dependency Injection --> Design Pattern (3 Techniques)
                            1) Constructor Injection
                            2) Setter Injection
                            3) Field Injection(Not recommended)
High Level Hierarchy
client --> layer1(Accept client request) --> layer2(Perform business logic) -->layer3(Connect to DB)-->database

Config File
Do you want spring to handle all classes?
We need objects of only few classes!
Hey,spring, don't create objects for everything, just create few
This "talking to the framework" is done in the configuration file

Spring Boot
Before you even run your web application we need an Apache Tomcat server along with all the configurations; not efficient
for small scale projects

Spring boot takes care of all "configuration problems"

Container
Springapplication.run(...) --> Basically creates a container
Your project has a lot of classes, and now you are saying "Hey spring it is your job to create objects";but where exactly
will spring do it?Spring will have a container in the JVM called the IOC container(Spring container) inside which all objects
will be created.
Thus, the first thing you need while running a spring application, is not the object but the container; the line responsible
for creating a container is written above

We assume Dev is a class, Laptop is another class and the Dev uses the Laptop
Dependency Injection
When you create an object of a class, you are actually creating this object in the JVM; outside the spring container; by
doing this it becomes your responsibility to manage the object lifecycle.

The object may be present in the container(IOC), which can be accessed by us.
In order for us to receive the object, we need to talk to the container; how do we do that?
Maybe we can get a reference of the container?
The type of the IOC container(even the container is an object which is present in the JVM) is ApplicationContext.
The run method in Springapplication.run(...) returns the object of ConfigurableApplicationContext which extends the
interface ApplicationContext which means that run is returning the object of ApplicationContext
Thus we can write something like this:
    Application context = SpringApplication.run(....)
After this we can simply use "context"
    Dev obj = context.getbean(Dev.class) #I am assuming that the object is already present in the IOC container, I just
                                           have to use it
What if the object is not present in the container?
    We know that spring does not create the object by default, it expects you to tell it which objects you want to create
    On the top of the class of which we want an object of, we can add a @component annotation; spring will understand that
    it has to manage that bean(object)

Autowiring
Hey spring framework i know you have created an object for dev, now i want you to create an object for Laptop as well, it
is simple, we add the @component annotation.Now we have two objects in the spring container(Dev and Laptop), how will we
connect these two objects?
@Autowired annotation --> Now your spring boot says, i want to create an object of Dev but Dev is dependent on Laptop; so
let's connect these two
Constructor injection --> Default autowiring
Field Injection and Setter Injection --> @Autowired annotation required
Question : How does the spring framework know that when you say @Autowired, it will connect to the Laptop class and not
           some other class?
Answer :   When you say Autowiring, it goes for "By Type" of the class; not "by Name"
           Adding an interface on top of computer will not cause an error(in case of autowiring the computer rather than
           laptop) as laptop is of type "computer" and autowiring is "by Type"
Another question Arises, what if we have another class Desktop which implements Computer and has its own specific imple-
mentation of the compile method? Which implementation of computer will autowire connect?
1. We can use a @Primary annotation of either Laptop or Desktop
2. We can use another annotation(this time below the @autowired) which would be @qualifier("laptop") or @qualifier("desktop")

Creating a Spring project without spring boot
1. Need to add the spring dependency in the pom.xml without which we wont have the IOC container and components
2. ApplicationContext context = new ClassPathXmlApplication(configlocation = "...xml"); we cant create an object of
   ApplicationContext as ApplicationContext is an interface; instead we have to go for classes that implement the Appli-
   cationContext interface which would be ClassPathXmlApplication.
It will still throw an error "bean factory not initialized or closed"; we need to create a configuration file in which we
will define our configurations
3. Spring XML Config
   In /main/resources we need to define a .xml file in which we need to define configurations
   Assume we have created an xml file spring.xml, here we define our bean in the <bean> tag
   It will throw an error initially as the spring project does not know what a bean tag is, thus we need to specify what a bean
   is by importing from "bean configuration" on the web(there will be a template)
   Objects will only be created in the spring container for those that are defined in the xml config file
   What is we define an object more than once, no problem it will make those many beans!

Constructor and setter injection
When you are creating an object in the config (xml) file; in the <bean> tag; you can assign properties.
    1. Property tag(in built) {setter injection}
    2. Constructor-arg {constructor injection}

Spring Web
Spring web cannot work on A JVM, we need a web container, in Java we have servlet which runs inside the web container; it will run on TomCat
We have an embedded TomCat in spring boot
We need some layer that will accept the request --> Controller Layer(special class)
In early days the server had to send both the data and the layout(frontend); but nowadays we are using two different applications for the fronend and backend.We have the layout ready which will go to the client, the server only sends data in the form of JSON or XML.

A simple spring project obtained from spring web by only adding dev tools and spring web dependencies starts a server at localhost:8080 without typing a single line of code; of course the output is some error message but it is not "page not found".Suppose we want to display "hello world" on the server; we need someone on the server who can accept our request and handle it; it has to be created! This class will act as a controller(first layer), which will accept a request and respond appropriately.

package com.aar.SimpleWebApp;

@Controller{Will this do our job?} --> No as on the website we have multiple requests; everything is a request with different URLs; so for every request you can create a different method that will respond; we use a @RequestMapping annotation on top of that specific method

@RestController --> Added afterwords
public class HomeController {

    @RequestMapping("/") --> *added later
    @ResponseBody --> Can be added if we dont want to use RestController
    public String greet(){
        return "hello world";
    }
}
A simple code like this can be added, but unfortunately the error will still persist; what do we have to change or add?
Even after adding the @Controller and @RequestMapping annotations, the error will persist, although this time it will say "No static resource "hello world""; what is this?
The controller behind the scene says; ok i am going to call the method, but still it isnt working?Why? What the spring web does is that when you are saying "hello world" it will look for a file called "hello world"!; which we do not have in our project; the fix here would be replacing the @Controller annotation with the @RestController annotation;REST api is a concept where you return the data from server to client(not layout, only data) --> an alternative but depreciated approach would be to use a @ResponseBody annotation above the method keeping the @Controller annotation the same.

There is no compulsion that all requests have to be put in one controller; we can have multiple controllers
	LoginController -->  Handles login requests("/login")
The question that arises here is that "how will the spring framework know for which request which controller to go to?"
Spring web or spring MVC has something called a "Front Controller", whenever you send a request, the request goes to a hidden layer(Front Controller), this front controller sees all the request mappings and knows for which request it has to send the request to which controller.

Usually in real world applications, for example an ecommerce website, we do not return plain text, we return objects(like a product), whose data is in either JSON or XML format which is readable by the frontend.Let us create a productcontroller to demonstrate the same

Product class which defines the product
package com.aar.SimpleWebApp;

@Data --> annotation for Lombok support
public class Product {
    private int prodId;
    private String prodName;
    private int price;
}
As the attributes are private, we need getters and setters for access, an easier automation for this process would be to use the "Lombok library" which will take care of the getter setter and constructor creation --> Lombok can be added by creating a dependency in the pom.xml

If a controller wants the data it wont ask data from the database, but will ask data from the service layer(layer 2); business logic should not be written inside the controller;thus a service layer needs to be created to write business logic.
We can revisit the concept of MVC here which stands for Model View Controller
			Model --> Represents Data
			View --> Returns a page
			Controller --> Request handling	

Two layer data retreival code:

1. Product in the model package, responsible for data representation

package com.aar.SimpleWebApp.model;

@Data --> Lombok support annotation
@AllArgsConstructor --> Lombok support annotation
public class Product {
    private int prodId;
    private String prodName;
    private int price;
}

2. Product service(layer two) in service package; here the business logic is written

package com.aar.SimpleWebApp.service;

import java.util.Arrays;
import java.util.List;

@Service --> Acts like a @Component annotation, which tells the spring framework to manage this bean in the IOC
public class ProductService {

    List<Product> products = Arrays.asList(new Product(101,"Iphone",50000),new Product(102,"Samsung",34000));
    public List<Product> getproducts(){
        return products;
    }
}

3. Product controller(layer 1)

package com.aar.SimpleWebApp.controller;

@RestController
public class ProductController {
    @Autowired
    ProductService service;
    @RequestMapping("/products")
    public List<Product> getproducts(){
        return service.getproducts();
    }
}

Output at localhost:8080/products

[
  {
    "prodId": 101,
    "prodName": "Iphone",
    "price": 50000
  },
  {
    "prodId": 102,
    "prodName": "Samsung",
    "price": 34000
  }
]

Now the next step would be CRUD operations on this data?
REST api runs on http protocol which has certain methods like GET --> Read,POST --> Create,PUT --> Update,DELETE --> Delete
By default when you pass a url in the browser it sends the get request, what about the other ones?This is not something you can do with your browser; we need to use seperate tools like postman.What if i want to return only the product with ID 101

http://localhost:8080/products/101 --> wont work with our current implementation
To make this work we need to create one more method that will accept this url.

In ProductController --> Addition
@RequestMapping("/products/{prodId}")
    public Product getproductbyid(@PathVariable int prodId){
        return service.getproductbyid(prodId);
    }
In ProductService --> Addition
public Product getproductbyid(int prodId) {
        return products.stream().filter(p -> p.getProdId() == prodId)
                .findFirst().get();
    }
#Jackson library is responsible for converting java object to JSON and vice verse

Now what if we want to add a product on the server?
The @RequestMapping is set to GET by default, in order to "put" or "add" some data onto the server we need to use @PostMapping(Can be on the same URL as long as there arent two annotations of the same type)

In ProductController --> Addition
 @PostMapping("/products")
    public void addproduct(@RequestBody Product prod){
        System.out.println("Received Product: " + prod);
        service.addproduct(prod);
    }
In ProductService --> Addition
 public void addproduct(Product prod){
        products.add(prod);
    }

Update and Delete
#Telusko Video time stamp 3 hour
