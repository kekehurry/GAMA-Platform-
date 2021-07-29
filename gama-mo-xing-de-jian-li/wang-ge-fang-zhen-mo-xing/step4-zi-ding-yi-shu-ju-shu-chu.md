# Step5:  自定义数据输出

除了在运行过程中呈现动态模拟窗口，GAMA还支持以其他形式输出模拟数据，包括代理检查器（Inspector）、数据监管器（Monitor）、图表、文件、图像等

### 数据检查器\(Inspector\)

GAMA内置了数据检查器，无需进行额外的设置。数据检查器包含两种检索信息的方式： **代理族浏览器（Species browser）** 和  **代理检查器（** **Agent inspector）**。

* **代理族浏览器（Species browser）**可通过点击模拟界面右上角的图标打开，代理族浏览器中，包含了同一代理族中各代理的所有属性信息。

![4.5.1 &#x4EE3;&#x7406;&#x6D4F;&#x89C8;&#x5668;&#x56FE;&#x6807;](../../.gitbook/assets/image%20%2810%29.png)

![4.5.2 &#x4EE3;&#x7406;&#x6D4F;&#x89C8;&#x5668;](../../.gitbook/assets/image%20%2811%29.png)

* **代理检查器（** **Agent inspector）**可通过在模拟界面右键选择某个代理，点击**Inspect** 查看某一代理的所有属性信息，并可以进行实时修改。

![4.5.3 &#x4EE3;&#x7406;&#x68C0;&#x67E5;&#x5668;](../../.gitbook/assets/image%20%2812%29.png)

### 数据监管器\(Monitor\)

数据监管器可以用来实时查看指定数据的输出，比如我们希望在运行过程中实时查看所有食草动物和食肉动物的数量，可以通过以下操作实现：

首先，在全局代理中添加全局变量分别表示食草动物数量和食肉动物数量：

```text
global {
	...
	//添加返回食草与食肉动物数量的全局参数
	int nb_preys -> {length (prey)};
	int nb_predators -> {length (predator)};
	...
}
```

> **length\(list\)** : 使用length函数将会返回列表的长度，这里分别返回当前食草动物与食肉动物族中代理的数量。

然后在实验设置的output中，添加数据监管器\(Monitor\)，监管nb\_preys、nb\_predators两个参数的变化。

```text
experiment prey_predator type: gui {

	output {
		//主窗口
		display main_display {
		...
		}
		//新增一个信息展示窗口
		display info_display {
		...
    }
    //添加数据监管器
    monitor "Number of preys" value: nb_preys;
    monitor "Number of predators" value: nb_predators;
    }
}
```

再次运行模拟，模拟界面将会出现Monitors窗口。

![4.5.4 &#x6570;&#x636E;&#x76D1;&#x7BA1;&#x5668;](../../.gitbook/assets/image%20%2819%29.png)

### 图表输出

如果希望通过更加直观的方式输出可视化的数据，GAMA支持输出折线图、拼图、直方图的输出。图标的输出定义在**实验设置\(experiment\)**中，使用**chart**关键字来构建不同的图表，接下来我们分别为捕食者模型添加折线图、饼图和直方图：

```text
    output {
        display main_display {
            ...
        }

        display info_display {
            ...
        }
        //创建一个新窗口显示图表，每五个循环刷新一次
        display Population_information refresh: every(5#cycles) {
            //创建名为‘Species evolution’的图表，类型为折线图，图表大小为宽1高0.5，图表左上角位置为0,0
            chart "Species evolution" type: series size: {1,0.5} position: {0, 0} {
                //填充数据，数据名为number_of_preys，数据值为nb_preys，显示为蓝色
                data "number_of_preys" value: nb_preys color: #blue;
                //填充数据，数据名为number_of_predator，数据值为nb_predators，显示为红色
                data "number_of_predator" value: nb_predators color: #red;
            }
            ...
        }
        ...
    }
}
```

> **cycles**: cycle是gama仿真模型中的时间度量，1个cycle表示仿真模型运行了一步，通常通过指定一天或者一个小时对应多少cycle来将仿真模型中模拟的时间和现实时间对应。

> **size:{1,0.5}**: 这里大小的表示，相当于将图表所在窗口的宽和高设为1，size:{1,0.5}表示图表的宽等于窗口宽，图表高等于窗口的一半。这种设定方式使得窗口改变时图表也会适应窗口跟着变化。同理，位置的表示也是一样的。

```text
//创建名为‘Prey Energy Distribution’的图表，类型为直方图，背景为浅灰色，图表大小为长0.5宽0.5，图表左上角位置为0,0.5
chart "Prey Energy Distribution" type: histogram background: #lightgray size: {0.5,0.5} position: {0, 0.5} {
    //数据区间0-0.025 的值是食草动物中能量值小于等于0.25的数量,颜色为红色
    data "0-0.25" value: prey count (each.energy <= 0.25) color:#red;
    //数据区间0.25-0.5 的值是食草动物中能量值大于0.25且小于等于0.5的数量，颜色为蓝色
    data "0.25-0.5" value: prey count ((each.energy > 0.25) and (each.energy <= 0.5)) color:#blue;
    //数据区间0.5-0.75 的值是食草动物中能量值大于0.5且小于等于0.75的数量,颜色为绿色
    data "0.5-0.75" value: prey count ((each.energy > 0.5) and (each.energy <= 0.75)) color:#green;
    //数据区间0.75-1 的值是食草动物中能量值大于0.75且小于等于1的数量，颜色为黄色
    data "0.75-1" value: prey count (each.energy > 0.75) color:#yellow;
    }
```

> list **count** condition**:**  计算列表list中符合condition条件的个体的数量。

```text
//创建名为‘Predator Energy Distribution’的图表，类型为饼图，背景为浅灰色，图表大小为长0.5宽0.5，图表左上角位置为0,0.5
chart "Predator Energy Distribution" type: pie background: #lightgray size: {0.5,0.5} position: {0.5, 0.5} {
    //数据区间0-0.025 的值是食肉动物中能量值小于等于0.25的数量,颜色为红色
    data "0-0.25" value: predator count (each.energy <= 0.25) color:#red;
    //数据区间0.25-0.5 的值是食肉动物中能量值大于0.25且小于等于0.5的数量，颜色为蓝色
    data "0.25-0.5" value: predator count ((each.energy > 0.25) and (each.energy <= 0.5)) color:#blue;
    //数据区间0.5-0.75 的值是食肉动物中能量值大于0.5且小于等于0.75的数量,颜色为绿色
    data "0.5-0.75" value: predator count ((each.energy > 0.5) and (each.energy <= 0.75)) color:#green;
    //数据区间0.75-1 的值是食肉动物中能量值大于0.75且小于等于1的数量，颜色为黄色
    data "0.75-1" value: predator count (each.energy > 0.75) color:#yellow;
    }
```

最终结果如下图所示：

![4.5.5 &#x56FE;&#x8868;&#x8F93;&#x51FA;](../../.gitbook/assets/image%20%2816%29.png)

### 文件输出

文件输出的基本句式如下：

```text
save my_data type: file_type to: file_name;
```

my\_data为要保存的数据，type 定义保存数据类型，to定义保存文件路径。常用的文件类型有shp\(gis文件格式）, csv 和 txt。save语句不一定要定义在experiment中，可以写在全局定义或者族定义里。下面我们在全局定义中，增加一个函数来保存模型运行数据。

```text
//定义一个save_result函数，每10个cycle 当食草动物和食肉动物数量都大于0时，
	//将食草动物最小、最大能量、食肉动物数量、食肉动物最小、最大能量保存至txt文件
	reflex save_result when: every(10#cycles) and (nb_preys > 0) and (nb_predators > 0){
    	save ("cycle: "+ cycle + "; nbPreys: " + nb_preys
      	+ "; minEnergyPreys: " + (prey min_of each.energy)
      	+ "; maxSizePreys: " + (prey max_of each.energy) 
      	+ "; nbPredators: " + nb_predators           
      	+ "; minEnergyPredators: " + (predator min_of each.energy)          
      	+ "; maxSizePredators: " + (predator max_of each.energy)) 
      	to: "results.txt" type: "text" ;
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
	//添加返回食草与食肉动物数量的全局参数
	int nb_preys -> {length (prey)};
	int nb_predators -> {length (predator)};
	//初始化
	init {
		//创建食草动物数量为nb_preys_init
		create prey number: nb_preys_init;
		//创建食肉动物数量为nb_predators_init
		create predator number: nb_predators_init;
	}
	//定义一个save_result函数，每10个cycle 当食草动物和食肉动物数量都大于0时，
	//将食草动物最小、最大能量、食肉动物数量、食肉动物最小、最大能量保存至txt文件
	reflex save_result when: every(10#cycles) and (nb_preys > 0) and (nb_predators > 0){
    	save ("cycle: "+ cycle + "; nbPreys: " + nb_preys
      	+ "; minEnergyPreys: " + (prey min_of each.energy)
      	+ "; maxSizePreys: " + (prey max_of each.energy) 
      	+ "; nbPredators: " + nb_predators           
      	+ "; minEnergyPredators: " + (predator min_of each.energy)          
      	+ "; maxSizePredators: " + (predator max_of each.energy)) 
      	to: "results.txt" type: "text" ;
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
	reflex reproduce when:(energy >= energy_reproduce) and (flip(proba_reproduce)) {
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
    	//创建一个新窗口显示图表，每五个循环刷新一次
        display Population_information refresh: every(5#cycles) {
            //创建名为‘Species evolution’的图表，类型为折线图，图表大小为宽1高0.5，图表左上角位置为0,0
            chart "Species evolution" type: series size: {1,0.5} position: {0, 0} {
                //填充数据，数据名为number_of_preys，数据值为nb_preys，显示为蓝色
                data "number_of_preys" value: nb_preys color: #blue;
                //填充数据，数据名为number_of_predator，数据值为nb_predators，显示为红色
                data "number_of_predator" value: nb_predators color: #red;
            }
            //创建名为‘Prey Energy Distribution’的图表，类型为直方图，背景为浅灰色，图表大小为长0.5宽0.5，图表左上角位置为0,0.5
			chart "Prey Energy Distribution" type: histogram background: #lightgray size: {0.5,0.5} position: {0, 0.5} {
    			//数据区间0-0.025 的值是食草动物中能量值小于等于0.25的数量,颜色为红色
    			data "0-0.25" value: prey count (each.energy <= 0.25) color:#red;
    			//数据区间0.25-0.5 的值是食草动物中能量值大于0.25且小于等于0.5的数量，颜色为蓝色
    			data "0.25-0.5" value: prey count ((each.energy > 0.25) and (each.energy <= 0.5)) color:#blue;
    			//数据区间0.5-0.75 的值是食草动物中能量值大于0.5且小于等于0.75的数量,颜色为绿色
    			data "0.5-0.75" value: prey count ((each.energy > 0.5) and (each.energy <= 0.75)) color:#green;
    			//数据区间0.75-1 的值是食草动物中能量值大于0.75且小于等于1的数量，颜色为黄色
    			data "0.75-1" value: prey count (each.energy > 0.75) color:#yellow;
    		}
       		//创建名为‘Predator Energy Distribution’的图表，类型为饼图，背景为浅灰色，图表大小为长0.5宽0.5，图表左上角位置为0,0.5
			chart "Predator Energy Distribution" type: pie background: #lightgray size: {0.5,0.5} position: {0.5, 0.5} {
    			//数据区间0-0.025 的值是食肉动物中能量值小于等于0.25的数量,颜色为红色
    			data "0-0.25" value: predator count (each.energy <= 0.25) color:#red;
    			//数据区间0.25-0.5 的值是食肉动物中能量值大于0.25且小于等于0.5的数量，颜色为蓝色
    			data "0.25-0.5" value: predator count ((each.energy > 0.25) and (each.energy <= 0.5)) color:#blue;
    			//数据区间0.5-0.75 的值是食肉动物中能量值大于0.5且小于等于0.75的数量,颜色为绿色
    			data "0.5-0.75" value: predator count ((each.energy > 0.5) and (each.energy <= 0.75)) color:#green;
    			//数据区间0.75-1 的值是食肉动物中能量值大于0.75且小于等于1的数量，颜色为黄色
    			data "0.75-1" value: predator count (each.energy > 0.75) color:#yellow;
    		}
    		
        }
    	//添加数据监管器
    	monitor "Number of preys" value: nb_preys;
    	monitor "Number of predators" value: nb_predators;
    	}
}
```



