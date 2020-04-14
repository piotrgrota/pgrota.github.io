---
layout: post
title: Document code with Swagger
---

![_config.yml]({{ site.baseurl }}/images/post1/swagger.png)


I was always curious how you can do good documentation to the code itself without writing a lot of text in other platforms like Confluence.
Problem that I could see was that lot of them was always outdated and hard to follow.

How can you do it differently - let's see one approach with help of Spring Boot and Swagger.

We will need fresh Spring Boot project:  
[Spring Boot](https://start.spring.io/) with following two dependencies.

<script src="https://gist.github.com/piotrgrota/897a9005f6e55e8a97a78dbcda8e4e97.js"></script>

Please replace ${swagger.version} with specific version I used : <swagger.version>2.9.2</swagger.version>.

You may ask What it gives us?


Swagger is an open-source software framework backed by a large ecosystem of tools that helps developers design, build, document, and consume RESTful web services. While most users identify Swagger by the Swagger UI tool, the Swagger toolset includes support for automated documentation, code generation, and test-case generation.

[Swagger](https://en.wikipedia.org/wiki/Swagger_(software))

These two libraries will allow us to document our API:

1.  springfox-swagger2 - will produce json format that contains information about our api
2.  springfox-swagger-ui - will produce nice UI to our consumers

Now we need to add few lines of the code :


<script src="https://gist.github.com/piotrgrota/e6e01307a9b263b8fb9373a60e03f629.js"></script>



* @EnableSwagger2 - Enables Springfox swagger 2
* Docket - Springfox’s, primary api configuration mechanism is initialized for swagger specification 2.0
* select() - returns an instance of ApiSelectorBuilder to 
give fine grained control over the endpoints exposed via swagger.
* paths() - allows selection of Path's using a predicate. 
The example here uses an any predicate (default). Out of the box we provide predicates for regex, ant, any, none.
* apis()  - allows selection of RequestHandler's using a predicate. 
The example here uses an any predicate (default). 
Out of the box predicates provided are any, none, withClassAnnotation, withMethodAnnotation and basePackage.
* build() - The selector needs to be built after configuring the api and path selectors.


I created following simple Rest API to demenstrate how it works.


<script src="https://gist.github.com/piotrgrota/1ca0579fda35a9121b370491afbfdd37.js"></script>



We have sample POST request for creating simple transaction in the context of user and making it Confirmed or rejected.
You can see we added annotation and description here:
@ApiOperation("create transaction for specific user.") - annotation contains short description what this method can do

How can we see documentation of it?

Navigate to below url : in my case I launched application with default port

[Local Swagger UI](http://localhost:8080/app-demo/swagger-ui.htm)

I used app-demo as root of my application 
server.servlet.context-path=/app-demo 

![_config.yml]({{ site.baseurl }}/images/Apr-10-2020-17-54-35-swagger.gif)


You can now see descirption we added tpo the code also what is nice you can execute the api and see response.
Also there is available curl command if needed.



```BASH
curl -X POST "http://localhost:8080/app-demo/transactions/create" \
-H "accept: application/json" -H "Content-Type: application/json" \
-d "{ \"amount\": 34.5, \"name\": \"NameSample\"}"
```


Was it hard ?

Don't think so ....

You can now how fast you can add documentation to the code without leaving IDE.






