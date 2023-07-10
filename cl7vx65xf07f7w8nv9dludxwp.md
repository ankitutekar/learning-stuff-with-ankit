---
title: "Economies of scale in the cloud"
datePublished: Sat Sep 10 2022 13:04:48 GMT+0000 (Coordinated Universal Time)
cuid: cl7vx65xf07f7w8nv9dludxwp
slug: economies-of-scale-in-the-cloud
cover: https://cdn.hashnode.com/res/hashnode/image/unsplash/l5Tzv1alcps/upload/v1662609160541/MgzaKcBVd.jpeg
tags: cloud, aws, azure, cloud-computing

---

Owing to the increasing adoption of cloud computing, familiarity with cloud computing services is becoming one of the most crucial skills for software development professionals. The cost efficiency of the cloud plays a key role in this continuously increasing adoption. In the quest to understand why public cloud providers are able to offer such cost-efficient solutions,  I learned about economies of scale experienced by cloud providers. In this article, we will be discussing different factors contributing to this effect.
    
## Economies of scale basics
    
Let's go through the basics of economies of scale before applying it in the context of the cloud. Economies of scale come in the picture when a company starts benefiting because of the size of its operation. It is depicted in the image below. Even though output(i.e. production) is continuously increasing, the production cost per unit has decreased from Cx to Cy.

![Economies of scale V2 (2).png](https://cdn.hashnode.com/res/hashnode/image/upload/v1662609408711/GBdoDN8Yf.png align="left")
  
Let’s try to understand this effect with an example - say you are running a restaurant that serves donuts. When you started, you were serving around 5 customers per day. It’s been almost 6 months now, your business has boosted and now you are serving around 250 customers per day. You will need to increase the production of donuts, but the production cost per donut will go down because of the following factors -
    
- Now you can buy ingredients in large quantities at a discounted price, which in turn results in fewer costs for buying ingredients required per donut
- Let’s say, you had employed people anticipating a demand of 225 customers per day. Now, your employees are more productive instead of sitting idle like in earlier days. This increased productivity isn’t causing costs(salaries!) to go up in the same proportion, but generating more output than in earlier days
- There are other fixed costs involved such as advertising, rent of the place, electricity, etc. that stay almost the same
    
You can further standardize processes(e.g. recipes) and have employees do specialized tasks. This will further increase the speed and overall efficiency of the operations, contributing to more cost-efficient production of donuts.
    
## Economies of scale in the cloud
    
A company having large data centers that provides computing resources as a service is more likely to benefit from economies of scale than companies having their own in-house servers. This is because of factors such as resource pooling, demand aggregation, demand variation, and certain fixed costs involved in setting up and maintaining the infrastructure. Let’s see how these supply-side savings and consumer demand aggregation contribute to economies of scale in the cloud -
    
### Supply-side savings
    
Similar to how increased size of operation causes cost reductions in our donut restaurant above, a cloud provider benefits in the following ways -
    
- **Buying in bulk**
        
Cloud providers buy thousands of units of hardware equipment at a time and can have long-term contracts with the suppliers. These factors result in them getting bigger discounts as compared to retail buyers. 
        
- **Cost saving in power expenses**
        
Power supply costs are an important component of expenses in an IT company. Power supply and related labor costs vary geographically. If you have your own private data center, it is most likely to be co-located with your office in a big city. You are very unlikely to move to a cheaper location because the benefits will outweigh the costs associated with setting up new infrastructure in a new location. But considering size of their operations, cloud providers can afford to make this move and take advantage of this geographical variation in costs. Also, large data centers are likely to have better power utilization compared to private data centers because of demand aggregation. Cloud providers can get better deals in infrastructure required for setting up the power supply and cooling systems.
        
- **Cost savings in labor expenses**
        
Large DCs have many systems of similar configuration. A cloud provider can invest in bringing in more expertise, incurring some fixed costs. But the solutions they develop can be utilized for a large number of systems. A cloud provider has less maintenance cost per machine as compared to in-house data centers - a single operator can handle more machines(1000s) and automation helps them do their jobs faster. 
        
- **Standardization and homogenization of hardware**
        
Standardizing and deciding on a fixed set of hardware models can also contribute to additional cost savings -
<br>&nbsp;- Buying equipment in large quantities gives larger discounts
<br>&nbsp;- Maintenance tasks and updates are easier to manage and can be automated
<br>&nbsp;- Costs of investment in security software, automation and other tools are spread over a large number of systems(1000s), resulting in less cost per system

### Consumer demand aggregation
    
Pooling of resources, making them available to thousands of customers, and serving their aggregated but varying demands results in better overall hardware utilization. It rarely happens that usage patterns are constant throughout the day. For an hour, there are millions of requests coming into your services, and the next hour, this number drops down to hundred!! There are different factors that decide the usage pattern of a particular software and in turn, affect the utilization of underlying hardware:
    
- **Time of day**

Some apps get very high traffic in the morning(e.g. news aggregators) and some in the evening(e.g. food delivery). Direct consumer targeted services such as social media see high traffic in the mornings and evenings whereas services that we use at our work(e.g. code hosting services, email and other communication services) experience traffic during the day but are idle during non-office hours.
Suppose, you own a software company that develops SaaS communication tools(i.e. chat, email, video conferencing) for other companies. Your hardware utilization might be good throughout the day but it might be sitting idle in non-office timings. You need to have sufficient demand and scale for it to be used efficiently around the clock. Maybe if your products are being used worldwide, it can generate constant demand to keep your hardware utilization at maximum. But only large-scale companies and a few products have that kind of demand to get the most out of their hardware 24/7.
- **Industry**

Logistics services may see very high traffic during holiday seasons but stay quiet the rest of the year. Financial services may experience high resource requirements during the end of the financial year but minimal usage rest of the year. Engineers working on these solutions need to consider these timings of peak usage while provisioning resources to run these services on. Festival season sales need to be considered for your E-commerce shop and consideration of traffic during tax filing season will be important for a tax filing app.
- **Resource profile**

A server hosting a Redis instance needs to have specialized storage whereas an image processing task needs GPU on the machine it is running on. A software doing scientific calculations will need a high amount of compute but won't need networking resources of the same scale. A machine comes with a certain amount of compute, storage, and networking capabilities. But software running on these machines won’t use all of these in equal proportion. Sure you can buy specialized hardware but can you guarantee continuous demand for these resources? These hardware configurations are expensive compared to commodity hardware.
- **Uncertainty in growth predictions**

When you are launching something new, you have to plan the capacity for running it and procure resources accordingly. Most of the time, these activities need to be done well in advance because there are approvals and other communications that need to happen. Planning capacity is a difficult activity because it is difficult to anticipate demands. You can't be sure if your TikTok rival app will be a hit or just make TikTok more popular. Will people even use the new stories feature (that I have copied from other apps) or will consider it unnecessary and I will eventually have to get rid of it?
        
    
All of these factors open up opportunities for achieving better utilization through demand aggregation and resource pooling. Large data centers are more likely to benefit from these than in-house data centers. Why? Because it is difficult for most IT companies to generate this kind of demand - continuous demand with complimentary access patterns. In-house solutions have to consider peak usage timings while planning capacity. To benefit from the above listed factors, cloud providers procure thousands of resources in a data center, implement intelligent resource sharing algorithms for their efficient utilization and provide computing services on an on-demand basis to users all over the world. 
    
So this is how supply-side savings and consumer demand aggregation play a critical role in cloud economics. Are public cloud providers the only ones benefiting from economies of scale? Definitely not! If an organization had sufficient demand from private data centers situated in multiple offices that is now being aggregated into a few data centers, we can say that they are in the process of bringing cost efficiency through the economics of scale. Has your organization benefited from this effect in any way? Do let us know in the comments!
