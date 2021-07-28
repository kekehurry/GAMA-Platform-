# Step2: 代理行为的编写

为了实现食草动物的移动，我们先为草地（vegetation\_cell）添加一个属性neighbors2，用来存储每个网格周边相距为2的所有网格

```text
grid vegetation_cell width: 50 height: 50 neighbors: 4 {
	...
	//将周边距离自己为2的其他vegetation_cell存到neighbors2列表中
	list<vegetation_cell> neighbors2  <- (self neighbors_at 2);
}
```

> **neighbors\_at** : 语句为 A neighbors_at distance , 返回与A相距distance的所有物体。在代码中self，指代每个vegetation_\_cell自身。

接下来编写食草动物的行为，食草动物的行为包括移动、进食、死亡和繁殖。首先为食草动物添加能量值（energy\)、最大能量值（max\_energy\)、每次最大进食量（max\_transfert）、每次能量消耗值（energy\_consum）、繁殖概率\(proba\_reproduce\)、最大繁殖数（nb\_max\_offsprings）、繁殖能量（energy\_reproduce）等属性。

```text
species prey {
	...
	//最大能量值
	float max_energy <- prey_max_energy;
	//每次进食量
	float max_transfert <- prey_max_transfert;
	//每次能量消耗值
	float energy_consum <- prey_energy_consum;
	//初始化能量为（0-max_energy)之间的随机数，每次模拟消耗energy_consum的能量，能量的最大值为max_energy
	float energy <- rnd(max_energy) update: energy - energy_consum max: max_energy;
	//繁殖概率
	float proba_reproduce <- prey_proba_reproduce;
	//最大繁殖数
	int nb_max_offsprings <- prey_nb_max_offsprings;
	//繁殖能量
	float energy_reproduce <- prey_energy_reproduce;
	...
}
```

### 移动行为

使用reflex句式编写每次模拟进行的行为。

```text
//定义移动行为
	reflex basic_move {
		//从当前所在网格的周边相距为2的网格中选一个设置为当前网格
		my_cell <- one_of(my_cell.neighbors2);
		//改变位置至当前网格的位置
		location <- my_cell.location;
	}
```

> **reflex:** 使用reflex关键字定义的函数，会在每次模拟（every step\) 都运行，这里定义的移动行为表示，食草动物会在每次模拟随机移动到周边距离为2的网格中。

### 进食和死亡行为

```text
//定义进食行为，当当前网格的食物量大于0时，发生进食行为
	reflex eat when: my_cell.food > 0 {
		//每次进食转移的能量取最大进食量和当前网格中食物量中的最小值
		float energy_transfert <- min([max_transfert, my_cell.food]);
		//进食后当前网格的食物量=食物量-转移的能量
		my_cell.food <- my_cell.food - energy_transfert;
		//食草动物的能量=能量+转移的能量
		energy <- energy + energy_transfert;
	}
	//定义死亡行为，当能量小于0时，死亡
	reflex die when: energy <= 0 {
		do die;
	}
```

> **reflex...when:**  reflex后面接when关键词表示当某个条件达成时，应该进行的行为。

### 繁殖行为

```text
//当能量大于繁殖能量，并且繁殖概率为真时，进行繁殖行为
	reflex reproduce when: (energy >= energy_reproduce) and (flip(proba_reproduce)) {
		//繁殖数量为1-最大繁殖数之间的随机数
		int nb_offsprings <- rnd(1, nb_max_offsprings);
		//生成nb_offsprings个与自身相同族群的代理
		create species(self) number: nb_offsprings {
			//新代理网格为当前网格
			my_cell <- myself.my_cell;
			//新代理位置为当前位置
			location <- my_cell.location;
			//新代理能量为当前能量除以繁殖数
			energy <- myself.energy / nb_offsprings;
		}
		//自身能量变为当前能量除以繁殖数
		energy <- energy / nb_offsprings;
	}
```

> **flip\(float\)**: 此处用flip函数将0-1之间的浮点数按其大小的概率转换为True或者False

同时为了能在模型运行时对各项参数进行调整，我们将食草动物最大能量值（max\_energy\)、每次最大进食量（max\_transfert）、每次能量消耗值（energy\_consum）、繁殖概率\(proba\_reproduce\)、最大繁殖数（nb\_max\_offsprings）、繁殖能量（energy\_reproduce）等属性设为全局属性，并在experiment中设为可调整参数。

```text
global {
	...
	float prey_max_energy <- 1.0;
	float prey_max_transfert <- 0.1;
	float prey_energy_consum <- 0.05;
	float prey_proba_reproduce <- 0.01;
	int prey_nb_max_offsprings <- 5;
	float prey_energy_reproduce <- 0.5;
  ...
}

experiment prey_predator type: gui {
	...
	parameter "Prey max energy: " var: prey_max_energy category: "Prey";
	parameter "Prey max transfert: " var: prey_max_transfert category: "Prey";
	parameter "Prey energy consumption: " var: prey_energy_consum category: "Prey";
	parameter 'Prey probability reproduce: ' var: prey_proba_reproduce category: 'Prey';
	parameter 'Prey nb max offsprings: ' var: prey_nb_max_offsprings category: 'Prey';
	parameter 'Prey energy reproduce: ' var: prey_energy_reproduce category: 'Prey';
	...
}
```

如此，便完成了食草动物行为的编写，本节完整代码如下：

```text
model prey_predator

global {
	// 定义食草动物的全局参数
	int nb_preys_init <- 200;
	float prey_max_energy <- 1.0;
	float prey_max_transfert <- 0.1;
	float prey_energy_consum <- 0.05;
	float prey_proba_reproduce <- 0.01;
	int prey_nb_max_offsprings <- 5;
	float prey_energy_reproduce <- 0.5;
	//初始化
	init {
		//创建食草动物数量为nb_preys_init
		create prey number: nb_preys_init;
	}
}

//创建食草动物族
species prey {
	//定义属性
	//显示大小
	float size <- 1.0;
	//颜色
	rgb color <- #blue;
	//所在的草地网格
	vegetation_cell my_cell <- one_of (vegetation_cell);
	//最大能量值
	float max_energy <- prey_max_energy;
	//每次进食量
	float max_transfert <- prey_max_transfert;
	//每次能量消耗值
	float energy_consum <- prey_energy_consum;
	//初始化能量为（0-max_energy)之间的随机数，每次模拟消耗energy_consum的能量，能量的最大值为max_energy
	float energy <- rnd(max_energy) update: energy - energy_consum max: max_energy;
	//繁殖概率
	float proba_reproduce <- prey_proba_reproduce;
	//最大繁殖数
	int nb_max_offsprings <- prey_nb_max_offsprings;
	//繁殖能量
	float energy_reproduce <- prey_energy_reproduce;
	
	//初始化位置为所在草地网格位置
	init {
		location <- my_cell.location;
	}
	
	//定义移动行为
	reflex basic_move {
		//从当前所在网格的周边相距为2的网格中选一个设置为当前网格
		my_cell <- one_of(my_cell.neighbors2);
		//改变位置至当前网格的位置
		location <- my_cell.location;
	}
	
	//定义进食行为，当当前网格的食物量大于0时，发生进食行为
	reflex eat when: my_cell.food > 0 {
		//每次进食转移的能量取最大进食量和当前网格中食物量中的最小值
		float energy_transfert <- min([max_transfert, my_cell.food]);
		//进食后当前网格的食物量=食物量-转移的能量
		my_cell.food <- my_cell.food - energy_transfert;
		//食草动物的能量=能量+转移的能量
		energy <- energy + energy_transfert;
	}
	
	//定义死亡行为，当能量小于0时，死亡
	reflex die when: energy <= 0 {
		do die;
	}
	
	//当能量大于繁殖能量，并且繁殖概率为真时，进行繁殖行为
	reflex reproduce when: (energy >= energy_reproduce) and (flip(proba_reproduce)) {
		//繁殖数量为1-最大繁殖数之间的随机数
		int nb_offsprings <- rnd(1, nb_max_offsprings);
		//生成nb_offsprings个与自身相同族群的代理
		create species(self) number: nb_offsprings {
			//新代理网格为当前网格
			my_cell <- myself.my_cell;
			//新代理位置为当前位置
			location <- my_cell.location;
			//新代理能量为当前能量除以繁殖数
			energy <- myself.energy / nb_offsprings;
		}
		//自身能量变为当前能量除以繁殖数
		energy <- energy / nb_offsprings;
	}
	
	//显示方式：大小为size的圆，颜色为color
	aspect base {
		draw circle(size) color: color;
	}
}
//创建一个50x50的四边形网格（注：neighbors控制网格的形状，如6边形、8边形等）
grid vegetation_cell width: 50 height: 50 neighbors: 4 {
	//定义网格属性
	//food代表每个网格的食物量
	//max_food每个网格最大食物量
	float max_food <- 1.0;
	//每次模拟网格中食物量随机增加的数值（0-0.01）
	float food_prod <- rnd(0.01);
	//每个网格初始食物量为（0-1）的随机数，每次模拟更新加food_prod，最大值为max_food
	float food <- rnd(1.0) max: max_food update: food + food_prod;
	//将周边距离自己为2的其他vegetation_cell存到neighbors2列表中
	list<vegetation_cell> neighbors2  <- (self neighbors_at 2);
	//根据食物量的大小，网格的颜色也会发生变化
	rgb color <- rgb(int(255 * (1 - food)), 255, int(255 * (1 - food))) update: rgb(int(255 * (1 - food)), 255, int(255 * (1 - food)));
}

//实验名称为prey_predator，输出方式为图形界面
experiment prey_predator type: gui {
	//定义食草动物可在图形界面调整的参数
	parameter "Initial number of preys: " var: nb_preys_init min: 1 max: 1000 category: "Prey";
	parameter "Prey max energy: " var: prey_max_energy category: "Prey";
	parameter "Prey max transfert: " var: prey_max_transfert category: "Prey";
	parameter "Prey energy consumption: " var: prey_energy_consum category: "Prey";
	parameter 'Prey probability reproduce: ' var: prey_proba_reproduce category: 'Prey';
	parameter 'Prey nb max offsprings: ' var: prey_nb_max_offsprings category: 'Prey';
	parameter 'Prey energy reproduce: ' var: prey_energy_reproduce category: 'Prey';
	//定义输出
	output {
		//输出窗口名为main_display
		display main_display {
			//显示vegetation_cell网格，线型为黑色
			grid vegetation_cell lines: #black;
			//以prey族中aspect定义的base方式显示prey族
			species prey aspect: base;
		}
	}
}
```

此时运行模型，已经可以看到食草动物吃草的仿真模拟，并且左侧的参数调整栏有了更多的可调整参数。

![4.2.1 &#x98DF;&#x8349;&#x52A8;&#x7269;&#x884C;&#x4E3A;&#x6A21;&#x62DF;](../../.gitbook/assets/image%20%286%29.png)

接下来，我们通过子、父族群来完成食肉动物行为的编写。

