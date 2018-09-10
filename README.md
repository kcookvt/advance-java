# Advance-Java
A lightweight REST framework for Java, built on top of com.sun.net.httpserver
## Getting Started
Read the following to set up Advance-Java on your machine
### Prerequisites
* [Java Development Kit](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)
### Installation
##### Install Advance-Java from source code
Your Java project should look like this:
```java
src->
    main->
        java->
            project_name
```
Simply download the Github code and copy the `advance` folder into your `java` folder.
To use the framework, import it like so:
```java
import advance.*;
//OR
import advance.Server;
import advance.Controller;
```
## Build Your First App
### Setting up the server
First, you need to build a server that directs HTTP traffic to Controllers, so create a file named `App.java` and put it into your project directory.
```java
package project_name;
import advance.Server;
public class App {
    public static void main(String[] args){
        //Initialize a server with a specified port
        Server app = new Server(3000);
        //Listen on port 3000
        app.listen();
    }
}
```
### Adding controllers
Right now, your server will do nothing. To route URLs to actions, create a controller in the file `/controllers/MainController.java`.
```java
package project_name.controllers;
import advance.Controller;
import com.sun.net.httpserver.HttpExchange;
public class MainController extends Controller {
    //Create a receiver for the GET method
    public void get(HttpExchange he){
        //Set a byte[] response to be sent
        super.response = "Hello world!".getBytes();
    }
}
```
Now, edit your `app.java` file to look like this:
```java
package project_name;
import advance.Server;
import project_name.controllers;
public class App {
    public static void main(String[] args){
        Server app = new Server(3000);
        //Route any "/" request to MainController
        app.addController("/", MainController());
        app.listen();    
    }
}
```
You now have an app that will send `hello world` to the browser at `localhost:3000` ! Continue reading for further information.
## Routing
### REST Methods
Advance supports all of the HTTP methods (GET, POST, PUT, PATCH, DELETE). You can handle one of them by creating a controller method with a corresponding lowercase name. If not handler is found, the server will respond witha 405 status code. Here is how an app would handle a couple of methods:
`app.java`
```java
package project_name;
import advance.Server;
import project_name.controllers;
public class App {
    public static void main(String[] args){
        Server app = new Server(3000);
        app.addController("/showMethod", MethodsController());
        app.listen();    
    }
}
```
`MethodsController.java`
```java
package project_name.controllers;
import advance.Controller;
import com.sun.net.httpserver.HttpExchange;
public class MethodsController extends Controller {
    public void get(HttpExchange he){
        super.response = "GET request to /".getBytes();
    }
    public void post(HttpExchange he){
        super.response = "POST request to /".getBytes();
    }
    public void patch(HttpExchange he){
        super.response = "PATCH request to /".getBytes();
    }
    public void put(HttpExchange he){
        super.response = "PUT request to /".getBytes();
    }
    public void delete(HttpExchange he){
        super.response = "DELETE request to /".getBytes();
    }
}
```
### Parameters and queries
Advance `Controller`s will automatically parse URL parameters (if specified) and any query strings that are detected. You can set url parameters by passing a url like `/your/url/:param` into the `Server` `addController` method. Queries and parameters are found in the `super.query` and `super.params` hashmaps, both of which are accessible to controller methods. Below is an example.
`app.java`
```java
package project_name;
import advance.Server;
import project_name.controllers;
public class App {
    public static void main(String[] args){
        Server app = new Server(3000);
        app.addController("/search/:category", ParamController());
        app.listen();    
    }
}
```

`ParamController.java`
```java
package project_name.controllers;
import advance.Controller;
import com.sun.net.httpserver.HttpExchange;
public class ParamController extends Controller {
    public void get(HttpExchange he){
        //If request url is /search/laptops?price=500
        String resp = "Search: " + params["category"] + ", $" + query["price"];
        super.response = resp.getBytes(); //Browser will show "Search: Laptop, $500";
    }
}
```
## Sessions
Sessions are easy to use and automatically setup in Advance as hashmaps. Access them in your app as follows:
`SessionController.java` - can be any controller
```java
package project_name.controllers;
import advance.Controller;
import project_name.db.*;
public class SessionController extends Controller {
    public void get(HttpExchange he){
        session["username"] = "johnsmith";
        session["password"] = "password123";
        super.response = "Session set".getBytes();
    }
    public void post(HttpExchange he){
        User newUser = new User(session["SID"], session["username"], session["password"]);
        newUser.save();
        super.responseCode = 302;
        super.headerEdits.put("Location", "/user/home");
    }
}
```
## Other HTTP Nuts & Bolts
### Setting an HTTP response
This was glanced over previously, but this is the code to add in a controller:
```java
super.response = "Your response here".getBytes();
```
### Changing the response code
The response code is set to 200 by default, but, to change it, you can do the following in a controller method:
```java
super.responseCode = 200;
```
### Changing response headers
Changing response headers is simple; in a controller add this code:
```java
super.headerEdits.put("key", "value");
```
### Redirects
Redirection is a two step proccess in which the response code and headers are changed. 
```java
super.responseCode = 302;
super.headerEdits.put("Location", "url");
```
## Running a project
Advance projects can be run like any other Java program. Access the server using `http://localhost:port-you-set` in the browser.