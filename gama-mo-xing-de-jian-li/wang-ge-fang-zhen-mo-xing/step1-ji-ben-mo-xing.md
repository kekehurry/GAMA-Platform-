# Step1: 基本模型

### 

回顾一下基本模型的结构，在GAML中，一个完整的模型定义，包含三个部分：**全局代理（global）、族群或网格（species and grid\)、实验设置（experiment\)。**

在GAMA中新建一个项目“Predator Prey"，并在models文件夹新建一个模型文件，GAMA会自动生成一个模型名称，我们先将模型名称改成项目名：

```text
model predator_prey
```

接下来定义全局代理（global），全局代理包括全局变量以及模型的初始状态。

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

然后定义食草动物（pred）和草（vegetation\)，这里我们将食草动物定义为族群（spices）草定义为网格（grid），因为草地是承载食草动物行为的环境。

```text
//创建一个50x50的四边形网格（注：neighbors控制网格的形状，如6边形、8边形等）
grid vegetation_cell width: 50 height: 50 neighbors: 4 {
	//定义网格属性
	//food代表每个网格的能量
	//max_food每个网格最大能量值
	float max_food <- 1.0;
	//每次模拟网格中能量随机增加的数值（0-0.01）
	float food_prod <- rnd(0.01);

	float food <- rnd(1.0) max: max_food update: food + food_prod;
	rgb color <- rgb(int(255 * (1 - food)), 255, int(255 * (1 - food))) update: rgb(int(255 * (1 - food)), 255, int(255 * (1 - food)));
}
```

