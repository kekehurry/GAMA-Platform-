# Step7 模型参数的优化\(Optional\)

在之前的模型中我们定义了许多可以手动调整的参数，如食草动物最大进食量（prey\_max\_transfert）、食草动物繁殖能量（prey\_energy\_reproduce）等，如果我们选择模拟一段时间后食草动物和食肉动物的数量作为观测目标，我们可以通过调整不同参数获得其对应的食草动物与食肉动物的数量，经过对比来选择能使最终数量较大的参数。

那么有没有一种方法能够自动帮我们找到最大化食草与食肉动物数量的参数呢，答案当然是肯定的，下面我们来实现模型参数的优化。

### batch experiment

在之前的实验设置中，我们一直使用的**gui** 类型的实验，**gui**可以实现动态可视化，但是在模型参数优化过程中，因为要进行大量实验，使用可视化界面会降低计算速度，因此这里我们使用 **batch**类型的实验，**batch**没有可视化界面，并且允许多线程运行，大大加快了模拟速度。

```text
//新建一个名为Optimization的实验，类型为batch,同一组参数运行2次，每次运行的随机种子相同，运行200次停止
experiment Optimization type: batch repeat: 2 keep_seed: true until: ( time > 200 ) {
}
```

> **repeat**:  同一组参数重复运行次数，仿真模拟运行有一定的随机性，因此多次运行可以让实验结果更可靠  
> **keep\_seed**: 是否使用相同的随机种子，GAMA的随机数是通过随机种子生成的，使用相同的随机种子可以生成相同的随机数。  
> **until  :**   定义模型停止运行的条件。

接下来添加要优化的参数，注意每个参数都要定义最大、最小值以及更次更新的步长。

```text
//新建一个名为Optimization的实验，类型为batch,同一组参数运行2次，每次运行的随机种子相同，运行200次停止
experiment Optimization type: batch repeat: 2 keep_seed: true until: ( time > 200 ) {
    //添加需要优化的参数
    parameter "Prey max transfert:" var: prey_max_transfert min: 0.05 max: 0.5 step: 0.05;
    parameter "Prey energy reproduce:" var: prey_energy_reproduce min: 0.05 max: 0.75 step: 0.05;
    parameter "Predator energy transfert:" var: predator_energy_transfert min: 0.1 max: 1.0 step: 0.1;
    parameter "Predator energy reproduce:" var: predator_energy_reproduce min: 0.1 max: 1.0 step: 0.1;
}
```

然后选择优化方式，gama内置了几种参数优化方式，如 [`exhaustive`](https://gama-platform.github.io/wiki/ExplorationMethods#exhaustive-exploration-of-the-parameter-space-exhaustive) 、[`hill_climbing`](https://gama-platform.github.io/wiki/ExplorationMethods#hill-climbing-hill-climbing) 、[`annealing`](https://gama-platform.github.io/wiki/ExplorationMethods#simulated-annealing-annealing) [`tabu`](https://gama-platform.github.io/wiki/ExplorationMethods#tabu-search-tabu) 、[`reactive_tabu`](https://gama-platform.github.io/wiki/ExplorationMethods#reactive-tabu-search-reactive-tabu) 、[`genetic`](https://gama-platform.github.io/wiki/ExplorationMethods#genetic-algorithm-genetic) 等，默认优化方式为[`exhaustive`](https://gama-platform.github.io/wiki/ExplorationMethods#exhaustive-exploration-of-the-parameter-space-exhaustive) ，即使用穷举法遍历所有可能的参数组合。具体优化方法的解释可以查看官方文档，这里我们采用[`tabu`](https://gama-platform.github.io/wiki/ExplorationMethods#tabu-search-tabu)优化方法。

```text
//采用tabu优化参数，优化目标是最大化nb_preys + nb_predators
method tabu maximize: nb_preys + nb_predators iter_max: 10 tabu_list_size: 3;
```

最后我们将每一的模拟结果保存下来：

```text
//保存模拟结果
reflex save_results_explo {
    ask simulations {
    save [int(self),prey_max_transfert,prey_energy_reproduce,predator_energy_transfert,predator_energy_reproduce,self.nb_predators,self.nb_preys] 
          to: "results.csv" type: "csv" rewrite: (int(self) = 0) ? true : false header: true;
    }       
}
```

> **simulations**:  simulations是一个内置变量，保存了所有模拟。

本节完整代码如下

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
	//读取图像文件
	file map_init <- image_file("../includes/data/predator_prey_raster_map.png");
	//初始化
	init {
		//创建食草动物数量为nb_preys_init
		create prey number: nb_preys_init;
		//创建食肉动物数量为nb_predators_init
		create predator number: nb_predators_init;
		//通过ask函数在初始化时修改vegetation_cell的属性
		ask vegetation_cell {
		//grid_x,grid_y是vegetation_cell的内置属性，返回其位置
		//将草地网格的颜色设置为相同位置上图像的颜色
    		color <- rgb (map_init at {grid_x,grid_y}) ;
   		 	//color as list 即将颜色的r、g、b转换为列表
    		//将食物量设置为1-红色通道值/255
    		food <- 1 - (((color as list) at 0) / 255) ;
    		//食物增长量为食物量/100
    		food_prod <- food / 100 ;
		}
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
		//使用choose_cell函数来确定移动行为的目的地
    	my_cell <- choose_cell();
    	location <- my_cell.location; 
    } 
    //初始化hoose_cell函数返回值为空值
    vegetation_cell choose_cell {
    return nil;
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
    //重写choose_cell函数
    vegetation_cell choose_cell {
        //选择相距为2的网格中食物量最大的那个
        return (my_cell.neighbors2) with_max_of (each.food);
    }
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
    //重写choose_cell函数
    vegetation_cell choose_cell {
        // 选择第一个遇到的，相距为2且其中有食草动物的网格
        vegetation_cell my_cell_tmp <- shuffle(my_cell.neighbors2) first_with (!(empty (prey inside (each))));
    	// 如果这个网格存在
    	if my_cell_tmp != nil {
        	//将这个网格作为目的地
        	return my_cell_tmp;
    	//否则
    	} else {
        	//随机选择一个周边相距为2的网格作为目的地
        return one_of (my_cell.neighbors2);
    	} 
    }
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
//新建一个名为Optimization的实验，类型为batch,同一组参数运行2次，每次运行的随机种子相同，运行200次停止
experiment Optimization type: batch repeat: 2 keep_seed: true until: ( time > 200 ) {
    //添加需要优化的参数
    parameter "Prey max transfert:" var: prey_max_transfert min: 0.05 max: 0.5 step: 0.05;
    parameter "Prey energy reproduce:" var: prey_energy_reproduce min: 0.05 max: 0.75 step: 0.05;
    parameter "Predator energy transfert:" var: predator_energy_transfert min: 0.1 max: 1.0 step: 0.1;
    parameter "Predator energy reproduce:" var: predator_energy_reproduce min: 0.1 max: 1.0 step: 0.1;
    //采用tabu优化参数，优化目标是最大化nb_preys + nb_predators
	method tabu maximize: nb_preys + nb_predators iter_max: 10 tabu_list_size: 3;
	//保存模拟结果
	reflex save_results_explo {
    	ask simulations {
    	save [int(self),prey_max_transfert,prey_energy_reproduce,predator_energy_transfert,predator_energy_reproduce,self.nb_predators,self.nb_preys] 
          	to: "results.csv" type: "csv" rewrite: (int(self) = 0) ? true : false header: true;
    }       
}
}
```

