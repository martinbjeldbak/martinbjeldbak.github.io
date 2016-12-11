---
title: "Representing mathematical graphs in Ruby"
date: 2012-10-15 17:08
comments: true
<!-- categories: Datastructures -->
---

In our Algorithms and Datastructures course, we've been working with algorithms that traverse directed and undirected graph structures. And in my quest to better understand these algorithms (depth-first search, breadth-first search, and so on…) I decided I would implement a graph data structure as a fun little weekend project.

Graphs have always piqued my interest, as they can visually be represented via images, and the fact that some of the algorithms have been quite entertaining to understand and see in action.

## Implementing graphs

First I thought about implementing vertices and edges as separate classes, but quickly found out that that wasn't the best decision, as I remembered that the algorithms in our course book defines graphs as an array of vertices containing linked lists with pointers (references in ruby) to vertices they have edges to. Ruby can take advantage of this, as Ruby always keeps references to objects, and doesn't duplicate them without specifically being told to. So creating the class Vertex as a decomposition of a Graph would be sufficient to give the required result. As you will see below, edge weight and labels are optionally represented by a hash in each vertex, distributing labels each adjacent vertex. Why have I implemented this? I don't know, really. The algorithms in our [book](http://www.amazon.co.uk/dp/0262533057) don't seem to be interested in edge weights… yet!

My first step is to implement a graph consisting of vertices and to create a function that outputs the graph using [DOT-language](http://en.wikipedia.org/wiki/DOT_language) notation before I start working on implementing any algorithms - they will show up in the next post or two. After an hour or so of jumping back and forth between using and not using the edge class gave me the following results:

{% highlight ruby %}
class Vertex
  attr_accessor :value, :d, :p, :color

  # :d = distance to root
  # :p = parent node

  def initialize(value)
    @value = value
  end

  def id
    self.object_id
  end

  def to_s
    "Node: #{@value}"
  end
end
{% endhighlight %}

And the graph class…

{% highlight ruby %}
## Graph.rb
# You are able to create a graph using an adjacency list
# from the provided class, or by manually adding edges
#

require './vertex.rb'
require './adjacencyList.rb'

class Graph
  attr_accessor :adjList

  def initialize(adjacencyList = AdjacencyList.new)
    # Allows you to populate the graph with an adjacency list
    # or it will make one for you
    @adjList = adjacencyList
  end

  def addEdge(srcVertex, dstVertex, weight = nil)
    @adjList.addEdge(srcVertex, dstVertex, weight)
  end

  def delEdge(srcVertex, dstVertex)
    @adjList.delEdge(srcVertex, dstVertex)
  end

  def weightBetween(srcVertex, dstVertex, newWeight = nil)
    @adjList.weightBetween(srcVertex, dstVertex, newWeight)
  end

  def displayUndirected(fileName = "graph")
    File.open(fileName + ".dot", "w") do |f|
      f.puts "graph {"

      @adjList.vertices.each do |vertex|
        f.puts "  #{vertex.id} [label=\"#{vertex.value}\"];"
      end

      f.puts

      @adjList.uedgeList.each do |edge|
        f.print "  #{edge[:src].id} -- #{edge[:dst].id}"
        f.print " [label=\"#{edge[:weight]}\"]" unless edge[:weight].nil?
        f.puts ";"
      end

      f.puts "}"
      makePng fileName
    end
  end

  def displayDirected(fileName = "graph")
    File.open(fileName + ".dot", "w") do |f|
      f.puts "digraph {"

      @adjList.vertices.each do |vertex|
        f.puts "  #{vertex.id} [label=\"#{vertex.value}\"];"
      end

      f.puts

      @adjList.vertices.each do |src|
        # Go to next vertex if current has no outgoing edges
        next if @adjList.dirEdgeList[src].nil?

        # For each of src vertex's edge...
        @adjList.dirEdgeList[src].each do |edge|
          f.print "  #{src.id} -> #{edge[:dst].id}"
          f.print " [label=\"#{edge[:weight]}\"]" unless edge[:weight].nil?
          f.puts ";"
        end
      end

      f.puts "}"
      makePng fileName
    end
  end

  private

  def makePng(fileName)
    %x[dot -Tpng "#{fileName}".dot -o "#{fileName}".png]
    # Commented out for cross-platformness
    #%x[rm graph.dot]
    #%x[open graph.png]
    puts "Graph representation saved to #{fileName}.png"
  end
end
{% endhighlight %}

![Graph using RubyGraphs](/assets/posts/graphExample1.png)


Simple, really! But this composition does exactly what I want! It outputs images as graphs such as the one pictured to the right with the DOT-language notation as shown below. As the DOT-language listing makes clear, each vertex has a unique ID (actually, it's their [object id](http://ruby-doc.org/core-1.9.3/Object.html#method-i-object_id), which is still unique for that object) so it's possible to change their values later, keeping the vertex's connecting edges in place. Neat!

```
digraph {
  70366898920260 [label="1"];
  70366898920200 [label="2"];
  70366898920140 [label="5"];

  70366898920260 -> 70366898920200 [label="3"];
  70366898920200 -> 70366898920200;
  70366898920200 -> 70366898920140 [label="9"];
  70366898920140 -> 70366898920260;
  70366898920140 -> 70366898920200 [label="4"];
}
```

Occasionally I might need to go back to the vertex and graph classes to expand them so they can be used in different algorithms (adding color and distance for use in depth-first search, for example), and the above source code will reflect those changes since they're just symlinked do the git versions.

If you have any comments or improvements, I'd love to hear them! Feel free to create a pull request on the newly created [GitHub repository](https://github.com/Fapper/RubyGraphs) I've made to keep track of my changes and additions to the project. It has received the _fantastic_ "RubyGraphs" name!

## To the future, and beyond!
In the next post or two, I plan to use this simple abstract data structure to implement various algorithms working on graphs. On Github I also plan to add the algorithms I decide to implement, though haven't decided which yet… though the first post will probably be about [depth-first search](http://en.wikipedia.org/wiki/Depth-first_search)!
