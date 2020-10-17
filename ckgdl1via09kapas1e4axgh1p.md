## Making sense of FaaS by learning about Azure Functions – Part I

<p>You must've seen the term <i>FaaS</i> or <i>Function as a Service</i> popping up on the internet every now and then. It has gained quite popularity recently. When I came across the term for the first time, I was puzzled. Function as a service? Do they write functions for us and we call them from our code? Is it like a paid library of utility functions having some secret code that only cloud providers can write? Well, after reading a few articles I came to know that I couldn't have been more wrong! I have been exploring this technology lately and been fascinated by how powerful it is.</p>
<p>In this article, we will explore FaaS by learning about some of its characteristics that are different from our traditional ways of doing things in software development. I will be using Azure Functions as a point of reference for explaining a few things. In part II, we will dive into Azure Functions.</p>
<p>First, let's start by defining FaaS - <i>It is a service provided by cloud providers that allows developers to run their code without worrying about building and managing required infrastructure.</i></p>
<p>So, it is one of the service, from plethora of services provided by cloud providers. Most of the major cloud providers provide this service, some examples include [AWS lambda](https://aws.amazon.com/lambda/), Azure Functions, [GCP Functions](https://developers.google.com/learn/topics/functions).</p>
Characteristics explained below will give you more insights into above definition -

<h3>No building and management of server side things</h3>
<ul>
<li>Cloud computing has different models based on how much responsibility we outsource - [IaaS, PaaS, SaaS](https://www.bmc.com/blogs/saas-vs-paas-vs-iaas-whats-the-difference-and-how-to-choose/) and FaaS. FaaS is a serverless technology. Our apps are deployed on someone else's server, i.e. in data centers of cloud providers. Serverless doesn't mean just provisioning of servers by cloud providers,  there is more to it.</li>
<li>In FaaS, we are responsible for only the code our functions contain and possibly a couple of configuration files. All the infrastructure building and management tasks like container/VM management, supplying required dependencies for the app, OS level processes management, host networking etc. are performed by cloud providers and systems setup by them. We just have to <b>specify</b> our resource requirements like memory size and CPUs, max number of VM instances, etc. Azure will provision those resources for us and will take care of managing those resources too.</li>
<li>Azure provides us different plans for hosting our function apps. Default one is consumption plan - provides auto-scaling, requires minimal configuration and is more suitable for irregular traffic patterns . You can switch to the premium plan if you have higher resource requirements than what is provided in the consumption plan. It provides auto-scaling, gives more features than the consumption plan like pre-warmed instances, VNet connectivity and is more suitable for apps that are always/mostly running.
<li>You can also run your functions in dedicated App Service Plan or Kubernetes cluster. Detailed comparison of different hosting plans for functions can be found [here](https://docs.microsoft.com/en-us/azure/azure-functions/functions-scale#hosting-plans-comparison) on MS docs.</li>
</ul>
<h3>Auto-scale based on load</h3>
<ul>
<li>As the number of requests increase, our function app is automatically scaled to support increased load. If we have to provision server instances(physical or virtual) by ourselves, there will be need for advance provisioning. In some cases, under-provisioning can also happen. These scenarios will be challenging to handle from cost and management perspective.</li>
<li>Due to the elastic scaling nature of FaaS, functions can also be scaled down to zero instances if there isn't enough activity to keep our functions alive. This is beneficial from cost perspective as we do not pay for the time when functions are not running.</li>
<li>In case of Azure Functions, the consumption plan and the premium plan both add necessary compute resources as load increases. If you are deploying functions in dedicated App Service Plan or on a Kubernetes clusters,  you will need to configure scaling manually and it is not as fast as elastic scaling of consumption and premium plans.</li>
<li>Consumption plan can scale upto 200 instances max and premium plan can scale upto 100 instances. Single instance can serve multiple requests. We can also throttle the scaling through configuration files.</li>
</ul>
<h3>Smaller execution duration</h3>
<ul>
<li>The design of FaaS demands functions to be ephemeral. This makes FaaS more suitable for event-driven use cases, something that is triggered as a result of some event(e.g. a record added in the database), it does its job and shut-downs.</li>
<li>If your function has been idle for a few minutes, it will be scaled down to zero instances. Serving new requests needs restarting the function app, which might happen on another instance and can take some time. This is known as <i>cold start</i> issue. This time could be in milliseconds but if it's happening frequently, can cause performance issues. The cold start time is affected by number of factors like how many dependencies your function needs to load, was the instance already running, etc.</li>
<li>Azure [recommends](https://docs.microsoft.com/en-us/azure/azure-functions/functions-best-practices) that we should be avoiding long-running functions. The timeout duration depends on the plan you are using and is configurable. For the consumption plan, default is 5 minutes and maximum configurable is 10 minutes. Switching to the premium plan gives you default of 30 minutes but you can configure it to never time out.</li>
<li>You can mitigate the cold start issue in the premium plan which provides option of pre-warmed instances and always ready instances. You can also run your function app on a dedicated App Service Plan. These options are more suitable for scenarios where your apps are always running/ mostly running, and can entail costs even if there are no requests, e.g. the premium plan requires at-least one instance to be pre-warmed so there is minimum cost associated with it.</li>
<li>Azure provides another option of [durable functions](https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-overview?tabs=csharp), which are extension to Azure Functions and are designed to be stateful, suitable for potentially long running workflows and provides abstractions for workflow orchestrations.</li>
</ul>
<h3>Minimal/ no local state</h3>
<ul>
<li>As stated above, the lifetime of functions is supposed to be smaller, this brings constraints on amount of data that can be held into main memory. The process will get terminated if there aren't enough requests to keep the instance alive. Your function app might be moved around on server/container instances every now and then.</li>
<li>Due to this, some external persistence layer like Redis cache on external storage needs to be implemented.</li>
<li>Azure recommends that our functions be stateless. You can switch to durable functions which provide functionality of preserving state across requests.</li>
</ul>
<h3>Pay as per consumption model</h3>
<ul>
<li>
Under pay as per consumption model of FaaS, we only pay for the number of requests that are actually executed unlike PaaS or IaaS offerings and on-prem setups where we have to keep our servers running even when there are no requests. In FaaS offering, we are not paying for idle CPU cycles. This brings huge cost benefits.
</li>
<li>Say you have launched a product for users in your country and it gets traffic only in the evenings and mornings. Using FaaS services wherever applicable will be beneficial for you from cost perspective. Such irregular traffic patterns and trying out things for newly launched products are ideal scenarios for FaaS offerings.</li>
<li>This is more evident in the consumption plan of Azure Functions. You are paying on the basis of resource consumption per request, and number of executions. Average memory consumption per request in gigabytes is multiplied by number of milliseconds your request is executed for, to get a metric - gigabytes seconds(by the way, first million executions are free each month!!!). In the premium plan, you pay based on number of core seconds and memory used across instances.</li>
</ul>
<br>
<p>So, <i>does FaaS make sense for your application?</i> As explained above, it can really help you with cost-cuttings when applied for correct scenarios. Cloud providers want to support needs of all the customers, their offerings are flexible. The line between PaaS offerings and FaaS offerings is getting blurry. Many of the above characteristics are applicable to PaaS upto certain degree.
Also, some of the characteristics explained above are not applicable to certain plans of FaaS offerings. We shouldn't get too much into terminologies, instead properly examine our use case and choose what fits our needs.</p>
<p>Have you created something with FaaS? How has your experience been? Do share your thoughts below.</p>
<br />
<span>Cover image credits <a href="https://unsplash.com/@tvick?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Taylor Vick</a> on <a href="https://unsplash.com/?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Unsplash</a></span>
<br />
#### References:
<ol>
<li>[Serverless Architectures by Mike roberts on MartinFowler.com](https://martinfowler.com/articles/serverless.html)</li>
<li>[MS Docs - Azure Functions scale and hosting](https://docs.microsoft.com/en-us/azure/azure-functions/functions-scale)</li>
<li>[Azure Functions pricing](https://azure.microsoft.com/en-us/pricing/details/functions/)</li>
</ol>