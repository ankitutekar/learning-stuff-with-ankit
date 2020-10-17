## Dependency Injection : The what and whys

<p>Dependency Injection is one of those terms that I was frightened of when I started my career in software development. People used to throw this term around along with some DI framework name attached to it which made it more frightening for me. When you are a junior working on industrial projects, most of the basic configuration is already setup for you to code in. Most of the times you will get your job done without understanding low level configurations. But you should make sure you spend some time on understanding these things every once in a while.</p>
<p>Anyways, in this post I aim to share my learnings about DI. I am assuming that you are familiar with basic OOPs concepts such as classes, interfaces and constructors. So, let's get started, shall we?</p>

# What is a dependency?
<p>Let's consider an example - Say you are working on an E-Commerce project which has functionality of creating orders of product(quite surprising right!). Say you have a class named OrdersApi.</p>

```
public class OrdersApi
{
 /*
 Has methods to create order, retrieve order by ID, Delete an order, etc.
*/
}
``` 


<p>Suppose, you have another service for calculating delivery date. Delivery date calculation has some data-science stuff hence its abstracted away in another service. You also have some DB connectivity. You have written separate classes in your orders service for handling these two functionalities (modularity!!!) - OrdersRepository(for making DB calls), DeliveryService(for making http requests to delivery date calculation service). These two classes are dependencies for our OrdersApi class. </p> <p>Let's say you decide to log every request to OrdersApi in some log file. You don't want to repeat logging configuration code at every API it is being used in, so you put it in a new file - Logger, making it another dependency of OrdersApi</p>
<p>Note that the example I am using for this post is based on OOP concepts, but DI is not limited to just OOPs languages. e.g. Components, helper files in your react project, are also examples of dependency.</p>

# Different ways of supplying dependencies:
## 1. new() it up everywhere:
The simplest way of supplying these dependencies to our OrdersApi is by instantiating their instances in OrdersApi - 

```
public class OrdersApi
{
  string dbConnectionString = "your://db-connection-string";
  string deliveryServiceAddress = "their://delivery-service-address";
  OrdersRepository ordersRepository = new OrdersRepository(dbConnectionString);
  DeliveryService deliveryService = new DeliveryService(deliveryServiceAddress);
  Logger logger = new Logger();
}
```

This just works! But there are some problems associated with it - <ul>
<li>
Our OrdersApi is now tightly coupled with its dependencies. Along with its own responsibilities (i.e. orders management) OrdersApi needs to know how to initialize these dependencies. This is violation of [Single Responsibility Principle](https://en.wikipedia.org/wiki/Single-responsibility_principle)</li>
<li> Tightly coupled classes make unit testing difficult. If I am talking about writing unit tests for OrdersApi, OrdersApi should be only module being targeted in test cases. I should be able to mock its dependencies(mocking is something that mimics behavior of dependencies without instantiating actual dependencies)</li>
<li> This reduces maintainability. Changes in lower level modules (i.e. Logger, OrdersRepository, DeliveryService) shouldn't cause any changes in higher level modules (i.e. our OrdersApi class). I should be able to change my logging service library in Logger class, without having to update other classes that are dependent on it. My OrdersApi shouldn't worry about type of DB being used. With this design, there is high chance of frequent changes in your application because:<ul> <li> There is no contract (interfaces!) being shared between dependencies</li> <li>Dependent modules are responsible for instantiating their dependencies. They will need to be aware about concrete implementation of their dependencies.</li></ul>
This is breaking [Inversion Of Control](https://stackoverflow.com/questions/3058/what-is-inversion-of-control) principle.
</li>
</ul>

## 2.Let the consumer supply it:
So we will ask consumer of OrdersApi to supply these dependencies by putting them as constructor parameters to OrdersApi-
```
public class OrdersApi
{
  public OrdersApi(OrdersRepository _ordersRepository, DeliveryService _deliveryService, Logger _logger)
   {
      this.ordersRepository = _ordersRepository;
      this.deliveryService = _deliveryService;
      this.logger = _logger;
   }
}
```
This solution doesn't resolve the issues mentioned above. The issues associated with OrdersApi design are now transferred to its consumer. In this way, we will keep creating a chain, making our code more complicated and hard to maintain.

## 3.Let some third party take over!:
What if there was someone to take care of all these dependencies? Someone to make sure that all the dependents are provided with their dependencies, managing proper ordering, letting dependents do their job without having to know concrete implementations of dependencies, letting us (the developers) manage our code more effectively? That someone is called Dependency Injection Framework!
We keep our design such that dependent modules are not directly using their dependencies, they are using abstractions(i.e. dependencies and dependents share some sort of contract they will adhere to - interfaces!). Then we tell DI framework who is dependent on what and who implements which contract. With this type of implementation Our OrdersApi will look like below - 
```
public class OrdersApi
{
  public OrdersApi(IOrdersRepository _ordersRepository, IDeliveryService _deliveryService, ILogger _logger)
   {
      this.ordersRepository = _ordersRepository;
      this.deliveryService = _deliveryService;
      this.logger = _logger;
   }
}
```
Here, IOrdersRepository, IDeliveryService, ILogger are contracts(interfaces). We will register this configurations in DI container at application startups. Syntactical part/which file to register this in depends on the language we are working with and the DI framework we are using. And also, it is not necessary that you should always be using a framework, people sometimes have their own variations of a DI framework. Some languages provide built in support for handling these kind of issues. Idea will be the same, issues they are addressing will be similar as described above.
Below example shows a snippet in Startup.cs file of aspnet core application - 

```
public void ConfigureServices(IServiceCollection services)
{
    services.AddSingleton<ILogger, Logger>();
    services.AddScoped<IDeliveryService, DeliveryService>();
    services.AddTransient<IOrdersRepository, OrdersRepository>();
}
```

Having a dedicated entity to handle dependencies makes our job easy in following ways - <ul>
<li>Higher level modules are not responsible for their lower level dependencies anymore. They will always be provided with required dependencies by DI container.</li>
<li>Coupling is reduced, testability of our code is increased! We can safely mock the behavior of dependencies, making it easy to test the modules in isolation.</li>
<li>As we are making sure that there is no direct dependency and everything is dependent on contracts and abstractions, it ensures that minimum changes will be required if you decide to update the implementation of lower level dependencies/ replace them with something else. This increases maintainability of our project.</li>
<li>Use of dedicated DI framework comes with additional benefits such as configuring lifetime of particular dependency. As you can see in above code snippet, AddSingleton, AddScope and AddTransient are different lifetime supported by the framework which controls how many instances will be created during particular request and how they will be provided to different parts of the application.
</ul>
<p>So that's all I have to say about this topic. Personally, learning about DI has really benefited me. It exposed me to different concepts such as - IoC, SRP, mocking in unit testing, testability, use of interfaces in real world project, etc.. It definitely made me a better programmer.</p>
<p>So how do you handle dependencies in your project? Do share it below. Also, feel free to mock me(lol) in the comments section.</p>