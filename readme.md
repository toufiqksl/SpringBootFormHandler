Handling Form Submission

URL : https://spring.io/guides/gs/handling-form-submission/

This guide walks you through the process of using Spring to create and submit a web form.

What you’ll build

In this guide, you will build a web form which will be accessible at the following URL:

http://localhost:8080/greeting
Viewing this page in a browser will display the form. You can submit a greeting by populating the id and content form fields. A results page will be displayed when the form is submitted.

What you’ll need

About 15 minutes
A favorite text editor or IDE
JDK 1.8 or later
Gradle 2.3+ or Maven 3.0+
You can also import the code from this guide as well as view the web page directly into Spring Tool Suite (STS) and work your way through it from there.
How to complete this guide

Like most Spring Getting Started guides, you can start from scratch and complete each step, or you can bypass basic setup steps that are already familiar to you. Either way, you end up with working code.

To start from scratch, move on to Build with Gradle.

Create a web controller

In Spring’s approach to building web sites, HTTP requests are handled by a controller. These components are easily identified by the @Controller annotation. The GreetingController below handles GET requests for /greeting by returning the name of a View, in this case, "greeting". A View is responsible for rendering the HTML content:

src/main/java/hello/GreetingController.java

package hello;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.ModelAttribute;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;

@Controller
public class GreetingController {

    @RequestMapping(value="/greeting", method=RequestMethod.GET)
    public String greetingForm(Model model) {
        model.addAttribute("greeting", new Greeting());
        return "greeting";
    }

    @RequestMapping(value="/greeting", method=RequestMethod.POST)
    public String greetingSubmit(@ModelAttribute Greeting greeting, Model model) {
        model.addAttribute("greeting", greeting);
        return "result";
    }

}
This controller is concise and simple, but a lot is going on. Let’s analyze it step by step.

The @RequestMapping annotation allows you to map HTTP requests to specific controller methods. The two methods in this controller are both mapped to /greeting. By default @RequestMapping maps all HTTP operations, such as GET, POST, and so forth. But in this case the greetingForm() method is specifically mapped to GET using @RequestMapping(method=GET), while greetingSubmit() is mapped to POST with @RequestMapping(method=POST). This mapping allows the controller to differentiate the requests to the /greeting endpoint.

The greetingForm() method uses a Model object to expose a new Greeting to the view template. The Greeting object in the following code contains fields such as id and content that correspond to the form fields in the greeting view, and will be used to capture the information from the form.

src/main/java/hello/Greeting.java

package hello;

public class Greeting {

    private long id;
    private String content;

    public long getId() {
        return id;
    }

    public void setId(long id) {
        this.id = id;
    }

    public String getContent() {
        return content;
    }

    public void setContent(String content) {
        this.content = content;
    }

}
The implementation of the method body relies on a view technology, in this case Thymeleaf, to perform server-side rendering of the HTML. Thymeleaf parses the greeting.html template below and evaluates the various template expressions to render the form.

src/main/resources/templates/greeting.html

<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Getting Started: Handling Form Submission</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
</head>
<body>
	<h1>Form</h1>
    <form action="#" th:action="@{/greeting}" th:object="${greeting}" method="post">
    	<p>Id: <input type="text" th:field="*{id}" /></p>
        <p>Message: <input type="text" th:field="*{content}" /></p>
        <p><input type="submit" value="Submit" /> <input type="reset" value="Reset" /></p>
    </form>
</body>
</html>
The th:action="@{/greeting}" expression directs the form to POST to the /greeting endpoint, while the th:object="${greeting}" expression declares the model object to use for collecting the form data. The two form fields, expressed with th:field="{id}" and th:field="{content}", correspond to the fields in the Greeting object above.

That covers the controller, model, and view for presenting the form. Now let’s review the process of submitting the form. As noted above, the form submits to the /greeting endpoint using a POST. The greetingSubmit() method receives the Greeting object that was populated by the form. It then adds that populated object to the model so the submitted data can be rendered in the result view, seen below. The id is rendered in the <p th:text="'id: ' + ${greeting.id}" /> expression. Likewise the content is rendered in the <p th:text="'content: ' + ${greeting.content}" /> expression.

src/main/resources/templates/result.html

<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Getting Started: Handling Form Submission</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
</head>
<body>
	<h1>Result</h1>
    <p th:text="'id: ' + ${greeting.id}" />
    <p th:text="'content: ' + ${greeting.content}" />
    <a href="/greeting">Submit another message</a>
</body>
</html>
For clarity, this example utilizes two separate view templates for rendering the form and displaying the submitted data; however, you can also use a single view for both purposes.

Make the application executable

Although it is possible to package this service as a traditional WAR file for deployment to an external application server, the simpler approach demonstrated below creates a standalone application. You package everything in a single, executable JAR file, driven by a good old Java main() method. Along the way, you use Spring’s support for embedding the Tomcat servlet container as the HTTP runtime, instead of deploying to an external instance.

src/main/java/hello/Application.java

package hello;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
@SpringBootApplication is a convenience annotation that adds all of the following:

@Configuration tags the class as a source of bean definitions for the application context.
@EnableAutoConfiguration tells Spring Boot to start adding beans based on classpath settings, other beans, and various property settings.
Normally you would add @EnableWebMvc for a Spring MVC app, but Spring Boot adds it automatically when it sees spring-webmvc on the classpath. This flags the application as a web application and activates key behaviors such as setting up a DispatcherServlet.
@ComponentScan tells Spring to look for other components, configurations, and services in the the hello package, allowing it to find the HelloController.
The main() method uses Spring Boot’s SpringApplication.run() method to launch an application. Did you notice that there wasn’t a single line of XML? No web.xml file either. This web application is 100% pure Java and you didn’t have to deal with configuring any plumbing or infrastructure.

Build an executable JAR

If you are using Gradle, you can run the application using ./gradlew bootRun.

You can build a single executable JAR file that contains all the necessary dependencies, classes, and resources. This makes it easy to ship, version, and deploy the service as an application throughout the development lifecycle, across different environments, and so forth.

./gradlew build
Then you can run the JAR file:

java -jar build/libs/gs-handling-form-submission-0.1.0.jar
If you are using Maven, you can run the application using mvn spring-boot:run. Or you can build the JAR file with mvn clean package and run the JAR by typing:

java -jar target/gs-handling-form-submission-0.1.0.jar
The procedure above will create a runnable JAR. You can also opt to build a classic WAR file instead.
Logging output is displayed. The service should be up and running within a few seconds.

Test the service

Now that the web site is running, visit http://localhost:8080/greeting, where you see the following form:

Form
Submit an id and message to see the results:

Result
Summary

Congratulations! You have just used Spring to create and submit a form.