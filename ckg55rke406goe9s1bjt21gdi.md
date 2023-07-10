---
title: "Understanding topological sort by cooking some Biryani"
datePublished: Sun Jun 21 2020 10:56:18 GMT+0000 (Coordinated Universal Time)
cuid: ckg55rke406goe9s1bjt21gdi
slug: understanding-topological-sort-by-cooking-some-biryani
canonical: https://dev.to/ankitutekar/understanding-topological-sort-by-cooking-some-biryani-43c3
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1602423812515/oqJEOmEUJ.jpeg
tags: algorithms, computer-science, data-structures, beginners

---


<p>Topological sort is an algorithm I wish I had programmed in my brain. So many tasks to do every day, with so much dependencies among them. It would've definitely improved my productivity without being dependent on any external tool.</p>

#### Disclaimer:
<p>The example of chicken Biryani is chosen just for making it sound fun. I have tried to generalize the steps and removed details. Steps were chosen from the first link Google gave me. There are multiple variations of this recipe. If you don’t know what Biryani is, visit [here](https://en.wikipedia.org/wiki/Biryani). If you ever decide to cook some Biryani, this is the post you shouldn't be referring to. I guess you know what happens when we start development with vague requirements, right?
<p>Alright then, let's start cooking some chicken Biryani.</p>

### Topological sort:
<p>
<ul>
<li>Topological sort is an algorithm used for the ordering of vertices in a graph. It outputs linear ordering of vertices based on their dependencies. We represent dependencies as edges of the graph.</li>
<li>It works only on Directed Acyclic Graphs(DAGs) - Graphs that have edges indicating direction. The vertices have one-way relationship among them. Also, there shouldn’t be any cyclic dependency among the vertices.</li>
<li>Some common applications of this algorithm in the world of computer science include- 
<ul>
<li>Scheduling jobs</li>
<li>Instruction scheduling</li>
<li>Deciding compilation sequence through make files.</li>
</u>
</li>
</ul>
</p>

### The problem:
<p>Suppose we were making Biryani and we wanted someone to tell us the linear ordering of steps in its recipe i.e. a sequence of steps that can be followed by taking one step at a time. Steps in making Biryani have interdependence among them, just like any other recipe. The generalized process of cooking chicken Biryani is outlined below-
<ul>
<li>Buy all the required ingredients.</li>
<li>Soak some rice.</li>
<li>Marinate chicken.</li>
<li>Prepare layering, i.e. fried Onions, mint leaves, etc.</li>
<li>Cook the rice.</li>
<li>Cook chicken.</li>
<li>Add layering on chicken.</li>
<li>Put the rice on top of it.</li>
<li>Serve!</li>
</ul>
</p> 

This problem sounds so trivial, but real-world applications of this algorithm can include hundreds of steps(vertices) and much more complex dependencies. For the purpose of this post, we will keep it short and easy to understand.

### The algorithm:
There are multiple topological sorting [algorithms](https://en.wikipedia.org/wiki/Topological_sorting#Algorithms) to approach this problem. One of these algorithms is [Kahn's algorithm](https://en.wikipedia.org/wiki/Topological_sorting#Kahn's_algorithm), which is what I have chosen to explain in this post. 
<p>The algorithm works in the below steps:
<ol>
<li>Find out a vertex which doesn’t have any dependency, i.e. the first task that can be performed.</li>
<li>Remove the vertex from the graph, removing the dependencies(edges) that other vertices had on this task. Add this vertex to result list.</li>
<li>Repeat above two steps for rest of the graph.</li>
</ol></p>

#### Pseudocode can be written as:
```
RS <- List to hold final ordered vertices
PQ <- A queue to hold vertices chosen for processing after 
they are removed from the graph

while PQ is not empty:
    -Current <- Dequeue a vertex from PQ
    -Add Current to RS
    for every vertex n dependent on Current:
        -Remove edge Current -> n
        -if n has no other incoming edge, enqueue it to PQ

if graph still has edges then
    return error (graph is not a DAG)
else
    return RS
```

This uses [Breadth First Search](https://en.wikipedia.org/wiki/Breadth-first_search) approach. There is also [Depth First Search](https://en.wikipedia.org/wiki/Topological_sorting#Depth-first_search) variation of this algorithm. 
Basically, making sure that step 'y' that is dependent on step 'x' is performed after step 'x', is the main goal of this algorithm.

Alright then, our Biryani making problem can be solved by algorithm above. Let's put the steps in graph form:

![Alt Text](https://cdn.hashnode.com/res/hashnode/image/upload/v1602423809092/im_XlxvgA.png)

<ul>
<li>We will start with a vertex that is not dependent on anything else. In our case, it is 'Buy Ingredients(BI)'. We will add this to processing queue and start the loop.</li>
<li>We will remove first vertex from the processing queue. Currently only 'BI' is present in our queue. We will add it to the result list.</li>
<li>We will remove the edges BI->SRC, BI->PL and BI->MCH from the graph. Here, what 'removing the edge' actually means depends on your implementation. In my implementation(linked below), I have maintained an in-degree counter at each vertex 'x' to track the number of vertices 'x' is dependent on. Also, I have maintained an adjacency list at 'x' to indicate the next steps to be taken from it. Each time a vertex 'v' is added to the result list, in-degrees of vertices that were dependent on 'v' will be decremented by one.</li>
<li>For all three vertices(SRC, PL, MCH), BI was the only dependency. Now that BI is added to result, these three vertices will be added to processing queue.</li>
<li>Above steps will be performed until processing queue is empty.</li>
</ul>
I have depicted the processing for above graph in below table. If it's hard to read, same has been uploaded in the repository linked below.

![Alt Text](https://cdn.hashnode.com/res/hashnode/image/upload/v1602423811058/lncZe_fUz.png)

<p>The implementation of this problem can be found on my repository [here](https://github.com/ankitutekar/TopologicalSortOfBiryaniRecipe).
</p>

<p>The core of the topological sort algorithm is in the gist file [here](https://gist.github.com/ankitutekar/fd0334729f9255b98115b49ba67fe9c4).</p>

<h3>Some facts about topological sort:</h3>

<ul>
<li>It only works on directed acyclic graph. If a graph has cycles, it can't have valid topological sort. </li>
<li>A graph can have more than one valid topological sort. In above example, if we were to process vertex MCH(Marinate Chicken) before vertex PL(Prepare layering), ordering would've been a bit different(and still valid). </li>
<li>Every DAG has at-least one valid topological sort. </li>
<li>Time complexity of this algorithm is O(|V| + |E|). |V| being number of vertices in graph and |E| being number of edges. We are visiting every vertex once and additional |E| time for calculating in-degrees.</li>
</ul>

Feel free to ping me if you have any doubts. Also, do try Biryani at least once!