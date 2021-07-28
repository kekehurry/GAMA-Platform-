# Step1: 基本模型

### 

回顾一下基本模型的结构，在GAML中，一个完整的模型定义，包含三个部分：**全局代理（global）、族群或网格（species and grid\)、实验设置（experiment\)。**

### Model Header

在GAMA中新建一个项目“Predator Prey"，并在models文件夹新建一个模型文件，GAMA会自动生成一个模型名称，我们先将模型名称改成项目名：

```text
model predator_prey
```

接下来定义全局代理（global），全局代理包括全局变量以及模型的初始状态。

### Global

```text
global {
	// 定义食草动物数量
	int nb_preys_init <- 200;
	//初始化
	init {
		//创建食草动物数量为nb_preys_init
		create prey number: nb_preys_init;
	}
}
```

### Species and Grid

然后定义食草动物（pred）和草（vegetation\)，这里我们将食草动物定义为族群（species）草定义为网格（grid），因为草地是承载食草动物行为的环境。

```text
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
	//根据能量值的大小，网格的颜色也会发生变化
	rgb color <- rgb(int(255 * (1 - food)), 255, int(255 * (1 - food))) update: rgb(int(255 * (1 - food)), 255, int(255 * (1 - food)));
}
```

> GAML中颜色的表示可以用\#+颜色名表示，如\#blue、\#red 等（[颜色列表](https://gama-platform.github.io/wiki/Index#Constants_and_colors)可以查看官方文档）。也可以用三通道rgb\(red,green,blue\)表示，如rgb\(255,255,255\)。

```text
//创建食草动物族
species prey {
	//定义属性
	//显示大小
	float size <- 1.0;
	//颜色
	rgb color <- #blue;
	//所在的草地网格
	vegetation_cell my_cell <- one_of (vegetation_cell);
	//初始化位置为所在草地网格位置
	init {
		location <- my_cell.location;
	}
	//显示方式：大小为size的圆，颜色为color
	aspect base {
		draw circle(size) color: color;
	}
}
```

### Experiment

最后定义模型运行时的输入与输出。

```text
//实验名称为prey_predator，输出方式为图形界面
experiment prey_predator type: gui {
	//定义食草动物的数量为可在图形界面调整的参数
	parameter "Initial number of preys: " var: nb_preys_init min: 1 max: 1000 category: "Prey";
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

至此一个有两种代理的基本模型构建完成，点击编辑窗口上方的绿色按钮，GAMA将会启动模拟界面，模拟界面由控制栏，参数调整栏，信息栏以及图形窗口组成。

* 控制栏：控制模拟的启动、暂停、退出等
* 参数调整栏：控制在experiment中定义的可调整参数
* 图形界面：显示在experiment中定义的图形输出

![4.1.1 GAMA&#x7A97;&#x53E3;&#x7684;&#x6A21;&#x62DF;&#x754C;&#x9762;](../../.gitbook/assets/image%20%287%29.png)

本节的完整代码如下：

```text
model prey_predator

global {
	// 定义食草动物数量
	int nb_preys_init <- 200;
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
	//初始化位置为所在草地网格位置
	init {
		location <- my_cell.location;
	}
	//显示方式：大小为size的圆，颜色为color
	aspect base {
		draw circle(size) color: color;
	}
}
//创建一个50x50的四边形网格（注：neighbors控制网格的形状，如6边形、8边形等）
grid vegetation_cell width: 50 height: 50 neighbors: 4 {
	//定义网格属性
	//food代表每个网格的能量
	//max_food每个网格最大能量值
	float max_food <- 1.0;
	//每次模拟网格中能量随机增加的数值（0-0.01）
	float food_prod <- rnd(0.01);
	//每个网格初始能量值为（0-1）的随机数，每次模拟更新加food_prod，最大值为max_food
	float food <- rnd(1.0) max: max_food update: food + food_prod;
	//根据能量值的大小，网格的颜色也会发生变化
	rgb color <- rgb(int(255 * (1 - food)), 255, int(255 * (1 - food))) update: rgb(int(255 * (1 - food)), 255, int(255 * (1 - food)));
}

//实验名称为prey_predator，输出方式为图形界面
experiment prey_predator type: gui {
	//定义食草动物的数量为可在图形界面调整的参数
	parameter "Initial number of preys: " var: nb_preys_init min: 1 max: 1000 category: "Prey";
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

接下来我们为不同的代理编写行为特征。

