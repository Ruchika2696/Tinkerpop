# Knowing Tinkerpop!!
## It is pretty cool!!
## This involves simple gremlin queries for graph database

## Consider the following graph that lists courses (circles) and prereq relationships (black arrows). Eg. CS101 is a prereq for CS201, and CS334, for CS400 (and so on). The two orange arrows are 'coreqs' (courses that may be taken together, possibly upon instructor approval for example). The orange arrows obviously lead to double connections: between CS420 and CS220, and between CS526 and CS400. If you'd like to think of nodes in terms of nouns, and edges in terms of verbs, our two kinds of edges would be 'requires pre-req' and 'is a co-req of'.

### 1) Creating a graph as: ![Courses](/courses.png)
```
myGraph = TinkerGraph.open()
```
### 2) Add Vertices:  It will add vertices to myGraph with given ids and labels and assign variable(kind of alias) as v1, v2.. etc so that we can access the verices using them.
```
	v1 = myGraph.addVertex(id, "CS101", label, "CS101")
 	v2 = myGraph.addVertex(id, "CS201", label, "CS201")
 	v3 = myGraph.addVertex(id, "CS220", label, "CS220")
 	v4 = myGraph.addVertex(id, "CS334", label, "CS334")
 	v5 = myGraph.addVertex(id, "CS420", label, "CS420")
 	v6 = myGraph.addVertex(id, "CS681", label, "CS681")
 	v7 = myGraph.addVertex(id, "CS400", label, "CS400")
 	v8 = myGraph.addVertex(id, "CS526", label, "CS526")
```

### 3) Add Edges: It will add edges from the given vertex. Syntax used is: 
```
sourceVertex.addEdge("value of edge", destinationVertex, id, idValue)
```
```
 	v2.addEdge("requires pre-req", v1, id, 1)
 	v3.addEdge("requires pre-req", v2, id, 2)
 	v4.addEdge("requires pre-req", v2, id, 3)
 	v5.addEdge("requires pre-req", v3, id, 4)
 	v5.addEdge("is a co-req of", v3, id, 5)
	v6.addEdge("requires pre-req", v4, id, 6)
 	v7.addEdge("requires pre-req", v4, id, 7)
 	v8.addEdge("requires pre-req", v7, id, 8)
 	v8.addEdge("is a co-req of", v7, id, 9)
```

### 4) Traversal: It is for traversal of graph. After this, we can directly use g1 instead of using ```myGraph.traversal()``` every time.

 ```
 g1 = myGraph.traversal()
 ```

### 5) a query that will output JUST the doubly-connected nodes: like ![Double](/double.png)

 ```
 g1.V().as("a").out().as("b").groupCount().by(select("a","b")).unfold().filter(select(values).is(eq(2))).select(keys)
 ```

 First of all we give alias "a" and "b" to the vertex. Here it simply means edges from vertex "a" to vertex "b". groupCount() will count the values generated by out() for every vertex pair "a" and "b". This means number of edges between "a" and "b". This happens because I gave parameters in by(). It indicates what grouping we are doing. unfold() will give the value of count produced. Now we just want the "a", "b" pairs with a count of 2. So we use filter. Here values represent the one generated by unfold step. We want to display the vertices. So using keys with select() will just give the ids which I have set as the course names during vertex creation steps.

### 6) a query that will output all the ancestors (for us, these would be prereqs) of a given vertex.
```
g1.V("CS526").repeat(out()).emit().dedup()
``` 
Here I give the id "CS526" as a parameter to V(). So the traversal starts from that vertex. Now I start the loop using repeat(). In loop I give out() which gives the node pointed by outgoing edges and it is passed by emit(). I can compare this as a while() loop in normal programming language. It breaks when there are no outgoing edges from a node that we reach. Here there can be a case that there are more than one edges between two nodes. so I used dedup() to avoid duplicates. To check for other vertices replace the value inside V().

### 7) a query that will output the max depth starting from a given node (provides a count (including itself) of all the connected nodes till the deepest leaf). This would give us a total count of the longest sequence of courses that can be taken, after having completed a prereq course.
```
g1.V("CS420").emit().repeat(__.in("requires pre-req")).path().count(local).order().by(decr).limit(1)
```
Here we have edge directions from bottom nodes to top nodes. So to go from top most node to the bottom most ones, we need to loop with incoming edges. So I used in() within repeat(). I used local to limit the counting to the currrent path as I want to count for every path and get the maximum one. So for every such path I get the count of the vertex given by the loop. Also, one important thing to note is that I used emit() before the looping. This is because I want to count the node itself too. If I use it after the loop then for the leaf nodes i.e. CS526 the answer would be 0 as there are no incoming edges to it. Putting emit() before repeat() works as do while loop in normal programming languages. After that, I ordered all the counts in decreasing order. Now if I give limit(1), it will just give the top most value which is the max value as they are sorted in decreasing order. To check for other vertices replace the value inside V(). 