# Step3: 父、子族群的继承

在我们的模型中食草动物和食肉动物有几乎一样的属性和行为，只是两者吃的东西不一样，比起重新编写一次食肉动物的行为，利用父族群和子族群的继承关系，避免重复的代码是更高效的写法。

### 父、子族群

在GAML中子族群可以继承族群所有的属性和行为，我们可以定义一个通用族设为父族，将食肉动物和食草动物共有的属性定义在父族中，并让他们从父族继承这些共有属性，而将他们不同的属性分别定义在各自族群里。

![4.3.1 &#x7236;&#x65CF;&#x7FA4;&#x548C;&#x5B50;&#x65CF;&#x7FA4;](../../.gitbook/assets/image%20%285%29.png)

### 共有属性

* 共有变量：
  * 大小：                        size
  * 颜色：                        color
  * 所在网格:                    my\_cell
  * 最大能量值：            max\_energy
  * 每次最大进食量：    max\_transfer
  * 每次能量消耗量：    energy\_consum
  * 所在网格:                    my\_cell
  * 能量值:                        energy
  * 繁殖概率:                    proba\_reproduce
  * 最大繁殖数：             nb\_max\_offsprings
  * 繁殖能量:                    energy\_reproduce                  
* 共有行为：
  * 移动：                         basic\_move
  * 进食：                         eat
  * 死亡：                         die
  * 繁殖：                         reproduce
* 共有显示：
  * 显示方式：                 base

注意食草动物和食肉动物都有吃的行为，但是两者具体操作稍有差异，因此我们在父族中实现一个函数energy\_from\_eat，并在子族中重写这个函数，子族重写的函数会覆盖父族同样的函数，从而实现食草动物和食肉动物不同的吃的行为。

### 父族代码实现

```text
//创建通用族
species generic_species {
	//定义属性
	//显示大小
	float size <- 1.0;
	//颜色
	rgb color;
	//所在的草地网格
	vegetation_cell my_cell;
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
	
	//显示方式：大小为size的圆，颜色为color
	aspect base {
		draw circle(size) color: color;
	}
}
```

