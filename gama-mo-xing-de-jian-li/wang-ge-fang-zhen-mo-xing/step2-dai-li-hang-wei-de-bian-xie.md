# Step2: 代理行为的编写

为了实现食草动物的移动，我们先为草地（vegetation\_cell）添加一个属性neighbors2，用来存储每个网格周边相距为2的所有网格。

```text
grid vegetation_cell width: 50 height: 50 neighbors: 4 {
	...
	//将周边距离自己为2的其他vegetation_cell存到neighbors2列表中
	list<vegetation_cell> neighbors2  <- (self neighbors_at 2);
}
```

> **neighbors\_at** : 语句为 A neighbors_at distance , 返回与A相距distance的所有物体。在代码中self，指代每个vegetation_\_cell自身。

接下来编写食草动物的行为，食草动物的行为包括移动、进食、死亡和繁殖。

```text
//定义移动行为
reflex basic_move {
		my_cell <- one_of(my_cell.neighbors2);
		location <- my_cell.location;
	}

reflex eat when: my_cell.food > 0 {
		float energy_transfert <- min([max_transfert, my_cell.food]);
		my_cell.food <- my_cell.food - energy_transfert;
		energy <- energy + energy_transfert;
	}

reflex die when: energy <= 0 {
		do die;
	}
```

