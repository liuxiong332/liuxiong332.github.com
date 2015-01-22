---
layout: post
title:  "图的基本算法"
date:   2014-03-25 16:06:49
categories: algorithm
---
* ###图的基本表示

	>众所周知，图`G=(V,E)有两种表示方式，分别是邻接表和邻接矩阵表示法。

	![图的两种表示方法](/images/graph_present_method.jpg "图的两种表示方法")
	-  **邻接表表示法**

		对于图`G=(V,E)`的每个顶点u，与u相邻的任意节点v，都被挂在u的链接表上。例如上图，因为`(A,B),(A,D)`均是图的邻接边，因此`B,D`都在A对应的邻接表中。

	-  **邻接矩阵表示法**

		对于图的任意邻接边`(u,v)`，都将邻接矩阵的第u行第v列置为1，表示`u->v`是相邻的，其余的不相邻全部因此置0.

	对于图，我们可以使用如下C++代码来表示：

	````cpp

	struct Node
	{
		int data;
		std::list<Node*>	adjacent_link;
	};

	class  Graph
	{
	public:
		void  AddNode(Node* node);
	private:
		std::list<Node*>	node_list_;
	};
	```
		
	上述代码中，Node表示每一个顶点，Graph则代表图`G=(V,E)`。Node中的adjacent_link成员变量链接着此节点的所有邻接顶点。

* ###广度优先搜索(breadth-first search)
	
	>在给定的图`G=(V,E)`和源节点s，广度优先搜索将从s开始找寻所有可达的边，搜索完s的所有邻接节点后，将会递归的搜索邻接节点的邻接节点，直到将所有的顶点都搜索完成。

	![DFS演示](/images/bfs_algorithm.jpg)

	通俗的讲，BFS首先搜索与s距离为k的节点，然后搜索距离为k+1的节点。在找寻节点的过程中，放了防止搜索岛相同节点造成无限递归，我们会对每个节点进行标记，例如我们令`(white, gray, black)`表示节点的三个状态：未被访问的节点、正在被访问的节点、访问完成的节点。其中对于gray节点u，表示当前正在访问u或者u的后嗣节点。例如上图，如果用BFS进行搜索，将会是A->B->D->C。具体的源码如下：

	```cpp
	void  BreadthFirstSearch(Graph* graph)
	{
		if(graph->node_list_.empty())
			return ;
		for(auto node: graph->node_list_) {		//reset all the data structure
			node->color = Node::WHITE;
			node->distance = 0;
			node->parent_node = NULL;
		}
		std::queue<Node*>  node_queue;
		Node*  source_node = *(graph->node_list_.begin());
		source_node->color = Node::GRAY;	//default, the source node has visited
		node_queue.push(  );
		while(!node_queue.empty()) {
			Node* node = node_queue.front();
			node_queue.pop();
			for(auto adjacent_node: node->adjacent_link) {
				if(adjacent_node->color == Node::WHITE) {
					adjacent_node->color = Node::GRAY;
					adjacent_node->distance = node->distance+1;
					adjacent_node->parent_node = node;
					node_queue.push(adjacent_node);
				}
			}
			node->color = Node::BLACK;	//the node's adjacent node has visited completely. 
		}
	}
	```

	上面的函数首先将图的所有节点数据结构清空，令节点颜色为white(还没有被访问)，父节点为NULL。然后从队列中取出一个节点进行遍历，并设置邻接节点的父节点和颜色参数，然后将此邻接节点放入队列，以便下次访问其邻接节点。当所有邻接节点访问完毕之后，将节点的颜色属性设为BLACK。其实在这里，BLACK枚举值是无关紧要的，只是后面的BFS需要这个枚举值，因此我们索性就把他加上了。

* ###深度优先搜索(depth-first search)
	
	>深度优先搜索是尽可能深的搜索，它是从一个节点的邻接节点一直往深处探测，直到没有节点可以访问，然后进行回溯，递归上述过程，直到搜索整个图。

	DFS和BFS一样，为了防止重复搜索某一个节点，他们对节点进行了着色。当节点未被访问时，节点颜色为white，当首次被访问时，颜色将会变为gray，当节点的后裔节点都访问完毕，回溯到此节点时，节点颜色变为black。

	除了颜色标记外，DFS还需要为每个节点加上时间戳，后面就会看到这个时间戳的用处了。每个顶点当首次被访问时，需要记录下第一个时间戳visit\_begin\_time，当后裔节点均访问完毕回溯到此节点时，需要记录此时时间戳visit\_end\_time，许多图算法将会用到它。

	![BFS演示](/images/dfs_algorithm.jpg)

	如上图所示，DFS从A->B->C一直往深处搜索，然后回溯到A点，最后访问D点。时间戳见图中节点内的数字对(开始时间/结束时间)。
	DFS代码片段如下：

	```cpp
	void  DFSNode(Node* node,int* time) 
	{
		node->color = Node::GRAY;
		node->visit_begin_time = ++*time;	//record the begin time visited
		for(auto adjacent_node: node->adjacent_link) {		//traverse all the adjacent node
			if(adjacent_node->color == Node::WHITE) {
				DFSNode(adjacent_node,time);				//recursively visit the descendant 	
			}
		}
		node->visit_end_time = ++*time;	//record the end time visited
		node->color = Node::BLACK;			//the node has visited completely
	}
	void  DepthFirstSearch(Graph* graph) 
	{
		for(auto node: graph->node_list_) {
			node->color = Node::WHITE;
			node->distance = 0;
			node->visit_begin_time = node->visit_end_time = 0;
			node->parent_node = NULL;
		}
		int time = 0;
		for(auto node: graph->node_list_) {
			if( node->color == Node::WHITE ) {
				DFSNode(node,&time);				
			}
		}
	}
	```

	上述代码对图中的任意顶点进行DFS，记录下访问的开始时间和结束时间。注意，我们并未对node\_parent和distance属性进行处理，这是因为这些属性是被BFS用来获取最短路径用的，DFS就没有使用这些属性了。

* ###拓扑排序(topological sort)

	拓扑排序并不是传统的排序，他只针对**有向无环图(directional acyclic graph)**有效。拓扑排序是将所有顶点排成一列，使得所有有向边都从左指向右。拓扑排序是用于确定事件的先后顺序，例如假设每个顶点代表一件事，顶点间的有向线段代表着事件的发生顺序，那么拓扑排序之后就确定了一个将所有事件进行先后顺序排列的序列。

	拓扑排序方法是很简单的。对于任意有向边`(u,v)`，我们首先对它进行DFS，然后比较其结束时间即可。具体分析如下：
	* v未被访问

		由于u->v，访问完v之后就会回溯到u，因此end\_time(u)>end\_time(v)；

	* v正在搜索它的后裔

		此时u是v的后裔(v在搜索它的后裔时发现了u)，于是构成了v->u->v的环路，与有向无环图的定义矛盾，因此这种情况不可能发生。

	* v已经被访问

		这种情况下，先访问v节点，然后访问u节点，很显然end\_time(u)>end\_time(v)

	由上面分析，我们知道对于有向边`(u,v)`，u的结束时间大于v的结束时间。要使得u排在v之前，只需要根据结束时间按照从大到小进行排列即可。C++代码如下：

	```cpp
	//Node compared function, if node1<node2, return true
	//when the node's end time greater, it should arrange in the front. 
	bool NodeLessCompare(Node* node1, Node* node2)
	{
		return node1->visit_end_time>node2->visit_end_time;
	}

	void  TopologicalSort(Graph* graph)
	{
		DepthFirstSearch(graph);
		std::sort(graph->node_list_.begin(),graph->node_list_.end(),NodeLessCompare);
	}
	```

* ###强联通分支(strong connected components)
	
	>有向图中的强联通分支是指分支中的`(u,v)`，若`u~v`(~表示可达)，则`v~u`，也就是说强分支中的任意两个顶点都相互可达。在很多算法中，都是将图分解为几个强联通分支，对每个分支进行分别求解，然后合并每个分支即得到整个图的解。

	对于如下的图，我们可以发现它由两个强联通分支组成，分别是`ABD`和`C`。

	![强联通分支图](/images/strong_connected_basic.jpg "强联通分支图")

	我们知道，对于强联通分支B，它的转置Bt也是强联通的。如果我们将图G进行DFS，形成类似拓扑排序(拓扑排序只针对DAG，但是此处忽略反转边)的序列，当对G进行反转时，就会形成下图所示情形：

	![强联通变形图](/images/strong_connected_transform.jpg "强联通变形图")

	我们假设对于两个强联通分支之间的有向边`(u,v)`有`u~v`，将图反转之后就成了`v~u`，如果对反转之后的图按照类拓扑排序的顺序从左到右进行DFS，我们将首先访问u，因为u所在的时强联通分支，因此进行DFS时他能访问到整个分支的顶点，而跟他相邻的顶点v无法到达(只有`v~u`)，因此DFS将获得此联通分支的所有节点，遍历图中的所有节点，即可获取所有的强联通分支。

	```cpp
	//reverse the orientation of the graph edge
	void  ReverseGraph(Graph* graph)
	{

	}

	void  GetComponents(Node* node, Graph* component)
	{
		node->color = Node::GRAY;
		component->node_list_.push_back(node);
		for(auto adjacent_node: node->adjacent_link) {		//traverse all the adjacent node
			if(adjacent_node->color == Node::WHITE) {
				GetComponents(adjacent_node,component);				//recursively visit the descendant 	
			}
		}
		node->color = Node::BLACK;
	}
	//component_list: hold all the strong connected component tree.
	void  StrongConnectedComponents(Graph* graph, std::list<Graph*>* component_list)
	{
		TopologicalSort(graph);
		ReverseGraph(graph);
		ClearGraph(graph);
		for(auto node: graph->node_list_) {
			if( node->color == Node::WHITE ) {
				Graph* graph = new Graph;
				component_list->push_back(graph);
				GetComponents(node,graph);
			}
		}
	}
	```

	上面代码首先进行拓扑排序，然后反转图，最后进行一次DFS，即可获得强联通分支。

	完整源代码请见 [basic\_graph\_algorithm.cpp](/code/basic_graph_algorithm.cpp)