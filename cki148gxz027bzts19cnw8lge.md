## Making sense of FaaS by learning about Azure Functions – Part II

<p>I have been getting into Azure lately and out of myriad of available services, I chose Azure Functions to explore first. In [Part I](https://www.learningstuffwithankit.dev/making-sense-of-faas-by-learning-about-azure-functions-part-i), we learned about characteristics of <strong>Function As A Service</strong>. We also saw some features of Azure's <strong>Azure Functions</strong> offering as examples of those characteristics. In this post, we will dive into Azure Functions by learning about its key concepts and see some examples.</p>
<p>I will be referencing code snippets from a small web-app that I have worked on while learning Azure Functions. It uses multiple serverless Azure solutions and is implemented in dotnetcore(C# language) and React. If you are not familiar with C#, no worries, I will try to explain C# specific syntactical parts in the examples below.</p>
<p>The app gives real time share market updates to clients. Data is stored in Cosmos DB - an Azure storage solution. There are some Azure Functions written to manipulate and fetch the data. Azure SignalR provides real time updates to connected clients who are accessing the React based app on their browser. If you want to read more about it or see the code, have a look at my github repository:</p>

%[https://github.com/ankitutekar/share-market-live-updates-using-Azure-services]

<p>Function is a primary concept in Azure Functions. These are similar to functions/methods in our traditional programming but with some superpowers! Azure functions can be written in a variety of languages including C#, Java, JavaScript, Python and more. Moreover, you can also add third-party dependencies such as npm /NuGet packages to extend its functionality.</p>
<p>The building blocks that make Azure Functions so powerful are its triggers and bindings. Trigger specifies when a function is to be executed. We can have a timer trigger, HTTP triggers, and also triggers can be set up for handling events happening in other Azure Services. Bindings allow you to connect to other services seamlessly. Azure needs a function.json file for executing functions, which has trigger and bindings information embedded in it. Let's talk about these concepts one by one:</p>
### Triggers
<ul>
<li>A trigger is what causes our function to run. Every function must have exactly one trigger. Suppose you want to add a message to queue when record is inserted in DB or schedule some code to run every 10 minutes, triggers can do that and much more. Azure supports timer trigger, HTTP trigger, Event Hub trigger, Blob trigger, Queue trigger and web-hooks too.</li>
<li>One of the most important benefits of these triggers is that you don't have to write code to connect to other Azure services, this is done <i>declaratively</i>. You just specify the source trigger information, the data from the trigger event is received in the function parameter.</li>
<li>To make sense of this, let's look at an example from the share market updates web-app - In below example, we are running the function whenever an update occurs in Cosmos DB. This updated document is reported to the connected SignalR client.
![Azure Function example](https://cdn.hashnode.com/res/hashnode/image/upload/v1606447953524/kzHva-rKR.png)
</li>
<li>If you are not familiar with C#, you will find the parameter declaration a bit weird. These are C# attributes used with function parameters, which allow us to decorate parameters with additional metadata. Attributes allow us to <i>declaratively</i> specify metadata about entities(class, methods, parameters, etc.) in the format [AttributeName(AttributeParameter1 = Value, AttributeParameter2 = Value, ...)].</li>
<li>In case of Azure Functions using C#, attributes are used to specify trigger and bindings information in function parameters. <i>FunctionName</i> attribute specifies entry point method in the class. In above example, we have passed <i>databaseName, collectionName</i> as parameters to attribute class constructor and further <i>ConnectionStringSetting, LeaseCollectionName, CreateLeaseCollectionIfNotExists</i> as attribute parameters. This attribute provides metadata about <i>updatedDocs</i> function parameter specifying collection to monitor for changes. If any records in the collection are updated, this function will be triggered and updated records will be available in <i>updatedDocs</i> function parameter.</li>
<li>
We have specified Cosmos DB connection information with its trigger and SignalR hub information with output binding, inside function parameters. As you can see, there is no code written to open connection to Cosmos DB, fetch the data or close the connection!!! Similarly, there is no code written to open connection to SignalR hub, we have just specified connection information <i>declaratively</i>! These operations are performed behind the scenes by Azure Functions runtime and code written in appropriate SDKs based on triggers/bindings.
</li>
</ul>
<h3>Bindings</h3>
<ul>
<li>Similar to triggers, bindings allow us to declaratively connect our function to other Azure Services. Unlike triggers, bindings are optional and you can have multiple bindings. Azure supports connecting to variety of services through bindings, e.g. storage services(blob, queue, table, Cosmos), event grids, event hubs, SignalR, twilio, etc. Moreover, you can also add your custom bindings.</li>
<li>Bindings have direction - <i>in</i>, <i>out</i> and <i>inout</i>. These directions specify if we are loading data into the function from other Azure service or outputting data from function to service. In above example, we have used SignalR output binding, specified with <i>[SignalR(HubName="notifications")]</i>. Note that the direction for bindings is optional and can be inferred by the way we have specified parameters, e.g. using <i>IReadOnlyList</i>, <i>IAsyncCollector</i> in above example. The SignalR output binding will send updated share data to all connected clients. It also allows us to specify user Ids and groups.</li>
<li>There are many trigger/binding specific attribute parameters supported using which we can do much more than what I have done in the above example.</li>
</ul>
<h3>function.json</h3>
<ul>
<li><i>function.json</i> is another important part of Azure Functions, along with the code that we write. It holds binding and trigger information in it, which is used by runtime for monitoring events, determining how to pass data to and from a function and for taking scaling decisions.</li>
<li>This file is auto-generated if you are using a compiled language but for scripting languages, it will be your responsibility to provide one. For compiled languages, it is generated based on annotations(attributes in above example) that you have written. It can be edited directly through Azure Portal as well, but you shouldn't be doing so unless you are just trying things out.</li>
<li>In case of Azure Functions using C#, you will find function.json generated in bin folder, one file per function in your function app. Function app is a collection of functions, a unit of Azure Functions deployment. Below file was generated for example above -

![functionJson example](https://cdn.hashnode.com/res/hashnode/image/upload/v1606447988307/JSRPSOoZ_.png)
</li>
<li>Along with this, there are two function app level config files:
 <ol><li><i>host.json</i> - holds runtime configuration</li>
 <li><i>local.settings.json</i> - holds secrets such as connection strings and is not meant to be published to Azure.</li></ol></li>
</ul>
<br />
<p>Azure provides extensions for Visual Studio and VS code for <a href="https://docs.microsoft.com/en-us/azure/azure-functions/functions-develop-local" target="_blank">local development of Azure Functions</a>. It also supports Eclipse and IntelliJ. Functions can be deployed in <a href="https://docs.microsoft.com/en-us/azure/azure-functions/functions-deployment-technologies" target="_blank">multiple ways</a> - GitHub actions, Azure DevOps, Azure App Service based deployments, etc.</p>
<p>This was my first time trying out serverless things and I must tell you, I have been fascinated by how much productive you can get in so little time. I hope you enjoyed these articles and I added something to your understanding of FaaS with Azure. Do let me know if any queries, will try my best to answer those.</p>
<h3>References and further reading</h3>
<ul>
<li><a href="https://docs.microsoft.com/en-us/azure/azure-functions/functions-reference" target="_blank">Azure Functions developer guide</a></li>
<li><a href="https://docs.microsoft.com/en-us/azure/azure-functions/supported-languages" target="_blank">Supported languages</a></li>
<li><a href="https://docs.microsoft.com/en-us/azure/azure-functions/functions-triggers-bindings?tabs=csharp#bindings-code-examples" target="_blank">Supported bindings</a></li>
<li><a href="https://docs.microsoft.com/en-us/azure/azure-signalr/signalr-concept-serverless-development-config" target="_blank">Azure Functions development and configuration with Azure SignalR Service</a></li>
<li><a href="https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-cosmosdb-v2-trigger?tabs=csharp" target="_blank">Azure Cosmos DB trigger for Azure Functions</a></li>
<li><a href="http://json.schemastore.org/function" target="_blank">Full JSON schema for function.json</a></li>
<li><a href="https://github.com/Azure/azure-functions-host/wiki/function.json" target="_blank">Some additional examples of function.json</a></li>
</ul>
