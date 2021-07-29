# Step4: 自定义显示方式

在GAML中代理族的显示方式，由族定义中的aspect关键字控制，在之前的定义中食草动物（prey\)和食肉动物\(predator\)都显示为圆形。

```text
	//显示方式：大小为size的圆，颜色为color
	aspect base {
		draw circle(size) color: color;
	}
```

而除了显示为几何图形（常见的有**circle**，**square**，**triangle**等）之外，GAML还能实现其他几种方式：

* **icon：**通过**image\_file** 将代理显示为自定义图标。
* **info：** 通过文字信息直接显示数据信息。
* 更多使用方式，可在官方文档搜索 [**"draw"**](https://gama-platform.github.io/wiki/Statements#draw)查询

下面我们来为模型定义图标显示及数据信息显示：

### 在父族中添加显示方式

同样为了避免重复的编写我们在父族中定义不同的显示方式，通过子族中初始化不同的变量来实现，子族显示的不同。

```text
species generic_species {
    ...
    //添加my_icon属性，其数据类型为image_file
    image_file my_icon;
    ...
    //显示方式base,显示为圆形,大小为size,颜色为color
    aspect base {
        draw circle(size) color: color ;
    }
    //显示方式icon,显示为my_icon,大小为2*size
    aspect icon {
        draw my_icon size: 2 * size ;
    }
    //显示方式info,显示为方形+能量值（精度到小数点后两位）
    aspect info {
        draw square(size) color: color ;
        draw string(energy with_precision 2) size: 3 color: #black ;
    }
}
```

### 子族-食草动物

```text
species prey parent: generic_species {
    ...  
    //定义my_icon 为羊的图标
    image_file my_icon <- image_file("../includes/data/predator_prey_sheep.png") ;
    ...
}
```

> TIPS：为了方便管理，GAMA的依赖文件都存储在includes文件夹

![4.4.1 &#x98DF;&#x8349;&#x52A8;&#x7269;&#x56FE;&#x6807;](../../.gitbook/assets/image%20%2816%29.png)

食草动物图标如上，为了代码生效，需将食草动物图标下载（[下载链接](https://gama-platform.github.io/resources/images/tutorials/predator_prey_sheep.png)）到项目文件夹的 `includes/data/`  文件夹中，并将图片名称修改为`predator_prey_sheep.png`

### 子族-食肉动物

```text
species predator parent: generic_species {
    ...
    //定义my_icon 为狼的图标
    image_file my_icon <- image_file("../includes/data/predator_prey_wolf.png") ;
    ...
}
```

![4.4.2 &#x98DF;&#x8089;&#x52A8;&#x7269;&#x56FE;&#x6807;](../../.gitbook/assets/image%20%2818%29.png)

同样的，将食肉动物图标下载（[下载链接](https://gama-platform.github.io/resources/images/tutorials/predator_prey_wolf.png)）到项目文件夹的 `includes/data/`  文件夹中，并将图片名称修改为`predator_prey_wolf.png`

### 实验显示

接下来更改**experiment**中的显示方式，在主窗口中将食草动物和食肉动物的显示方式改为icon，然后新建一个信息展示窗口，信息展示窗口中，食草动物和食肉动物的显示方式设置为info。

```text
experiment prey_predator type: gui {
	...
	//定义输出
	output {
		display main_display {
			grid vegetation_cell lines: #black;
			//将prey族的显示方式更改为icon
			species prey aspect: icon;
			//将predator族的显示方式更改为icon
			species predator aspect:icon;
		}
		//新增一个信息展示窗口
		display info_display {
    		grid vegetation_cell lines: #black ;
    		//将prey族的显示方式更改为info
        species prey aspect: info;
        //将predator族的显示方式更改为info
        species predator aspect: info;
    }
	}
}
```

本节完整代码如下：

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
	// 定义食肉动物的全局参数
	int nb_predators_init <- 20;
	float predator_max_energy <- 1.0;
	float predator_energy_transfert <- 0.5;
	float predator_energy_consum <- 0.02;
	float predator_proba_reproduce <- 0.01;
	int predator_nb_max_offsprings <- 3;
	float predator_energy_reproduce <- 0.5;
	//初始化
	init {
		//创建食草动物数量为nb_preys_init
		create prey number: nb_preys_init;
		//创建食肉动物数量为nb_predators_init
		create predator number: nb_predators_init;
	}
}

//创建通用族
species generic_species {
	//定义属性
	//显示大小
	float size;
	//颜色
	rgb color;
	//所在的草地网格
	vegetation_cell my_cell <- one_of(vegetation_cell);
	//最大能量值
	float max_energy;
	//每次进食量
	float max_transfert;
	//每次能量消耗值
	float energy_consum;
	//初始化能量为（0-max_energy)之间的随机数，每次模拟消耗energy_consum的能量，能量的最大值为max_energy
	float energy <- rnd(max_energy) update: energy - energy_consum max: max_energy;
	//繁殖概率
	float proba_reproduce;
	//最大繁殖数
	int nb_max_offsprings;
	//繁殖能量
	float energy_reproduce;
	//添加my_icon属性，其数据类型为image_file
    image_file my_icon;
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
	
	//定义进食行为,子族群进食行为不一样，通过重写energy_from_eat实现不一样的进食行为
	reflex eat {
		energy <- energy + energy_from_eat();
	}
	float energy_from_eat {
    return 0.0;
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
	
	//显示方式base,显示为圆形,大小为size,颜色为color
    aspect base {
        draw circle(size) color: color ;
    }
    //显示方式icon,显示为my_icon,大小为2*size
    aspect icon {
        draw my_icon size: 2 * size ;
    }
    //显示方式info,显示为方形+能量值（精度到小数点后两位）
    aspect info {
        draw square(size) color: color ;
        draw string(energy with_precision 2) size: 3 color: #black ;
    }
}

//创建食草动物族，其父族为generic_species 
species prey parent: generic_species {
    //初始化显示大小
    float size <- 1.0;
    //初始化颜色
    rgb color <- #blue; 
    //初始化最大能量值
    float max_energy <- prey_max_energy ;
    //初始化每次最大进食量
    float max_transfert <- prey_max_transfert ;
    //初始化每次能量消耗值
    float energy_consum <- prey_energy_consum ;
    //初始化繁殖概率
    float proba_reproduce <- prey_proba_reproduce ;
    //初始化最大繁殖数
    int nb_max_offsprings <- prey_nb_max_offsprings ;
    //初始化繁殖能量
    float energy_reproduce <- prey_energy_reproduce ;
    //定义my_icon 为羊的图标
    image_file my_icon <- image_file("../includes/data/predator_prey_sheep.png") ;
    //重写energy_from_eat函数
    float energy_from_eat {
    //初始化能量转移量
    float energy_transfert <- 0.0;
    //当所在网格的食物量大于0时
    if(my_cell.food > 0) {
        //能量转移量为每次最大进食量与网格内食物量的最小值
        energy_transfert <- min([max_transfert, my_cell.food]);
        //更新网格内食物量
        my_cell.food <- my_cell.food - energy_transfert;
    } 
    //返回能量转移量         
    return energy_transfert;
    }
}

//创建食肉动物族，其父族为generic_species 
species predator parent: generic_species {
    //初始化显示大小
    float size <- 1.0;
    //初始化颜色
    rgb color <- #red; 
    //初始化最大能量值
    float max_energy <- predator_max_energy ;
    //初始化每次进食量
    float energy_transfert <- predator_energy_transfert ;
    //初始化每次能量消耗值
    float energy_consum <- predator_energy_consum ;
    //初始化繁殖概率
    float proba_reproduce <- predator_proba_reproduce ;
    //初始化最大繁殖数
    int nb_max_offsprings <- predator_nb_max_offsprings ;
    //初始化繁殖能量
    float energy_reproduce <- predator_energy_reproduce ;
    //定义my_icon 为狼的图标
    image_file my_icon <- image_file("../includes/data/predator_prey_wolf.png");
    //重写energy_from_eat函数
    float energy_from_eat {
    //列出所在网格内的食草动物
    list<prey> reachable_preys <- prey inside (my_cell); 
    //如果食草动物列表不为空   
    if(! empty(reachable_preys)) {
        //随机吃掉一个食草动物，那个食草动物死去
        ask one_of (reachable_preys) {
        do die;
        }
        //返回食肉动物每次进食量
        return energy_transfert;
    }
    //如果食草动物列表为空，返回0
    return 0.0;
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
	//定义食肉动物可在图形界面调整的参数
	parameter "Initial number of predators: " var: nb_predators_init min: 0 max: 200 category: "Predator";
	parameter "Predator max energy: " var: predator_max_energy category: "Predator";
	parameter "Predator energy transfert: " var: predator_energy_transfert category: "Predator";
	parameter "Predator energy consumption: " var: predator_energy_consum category: "Predator";
	parameter 'Predator probability reproduce: ' var: predator_proba_reproduce category: 'Predator';
	parameter 'Predator nb max offsprings: ' var: predator_nb_max_offsprings category: 'Predator';
	parameter 'Predator energy reproduce: ' var: predator_energy_reproduce category: 'Predator';
	//定义输出
	output {
		//主窗口
		display main_display {
			grid vegetation_cell lines: #black;
			//将prey族的显示方式更改为icon
			species prey aspect: icon;
			//将predator族的显示方式更改为icon
			species predator aspect:icon;
		}
		//新增一个信息展示窗口
		display info_display {
    		grid vegetation_cell lines: #black ;
    		//将prey族的显示方式更改为info
        species prey aspect: info;
        //将predator族的显示方式更改为info
        species predator aspect: info;
    	}
    	}
}
```

开始模拟，模拟界面将会出现两个窗口：main_\_display 和 info\_display:_

![4.4.3 main\_display](../../.gitbook/assets/image%20%2820%29.png)

![4.4.4 info\_display](../../.gitbook/assets/image%20%2814%29.png)

