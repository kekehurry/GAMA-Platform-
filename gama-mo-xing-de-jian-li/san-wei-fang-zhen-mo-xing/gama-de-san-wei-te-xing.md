# Step1: GAMA的三维特性

### 三维显示

首先，我们来编写一个简单的模型，这个模型是一个100x100x100的立方体空间，在此空间内，随机分布着100个半径为1的球体。

```text
model Tuto3D
//全局定义
global {
  //定义cell族代理数量
  int nb_cells <- 100;	
  //初始化模型
  init { 
    //创建cell族
    create cell number: nb_cells { 
      //每个cell代理的初始位置为100x100x100内的随机位置
      location <- {rnd(100), rnd(100), rnd(100)};       
    } 
  }  
} 
//创建cell族
species cell {  
  //cell族显示为蓝色的半径为1的小球                    
  aspect default {
    draw sphere(1) color: #blue;   
  }
}
//实验设置
experiment Tuto3D  type: gui {
  parameter "Initial number of cells: " var: nb_cells min: 1 max: 1000 category: "Cells" ;	
  output {
    display View1 type: opengl {
      species cell;
    }
  }
}
```

运行模型，模拟界面将默认显示模型的顶视图，此时按住`ctrl +鼠标左键` 可以转动模型。

![6.1.1 &#x7B80;&#x5355;&#x7684;&#x4E09;&#x7EF4;&#x663E;&#x793A;](../../.gitbook/assets/image%20%2833%29.png)

此时模型只在x,y平面有自动显示出来的边界线，接下来，我们为模型添加三维边界的显示。

```text
global{
    ...
    //在全局定义中增加环境大小参数
    int environment_size <-100;
    //定义全局代理的形状为边长为100的立方体
    geometry shape <- cube(environment_size);  
    ...
}
//实验设置
experiment Tuto3D  type: gui {
  ...
  output {
    display View1 type: opengl {
      ...
      //在显示中增加环境的显示
      graphics "env" {
        draw cube(environment_size) color: #black empty: true;  
      }
    }
  }
}
```







![6.1.2 &#x5E26;&#x8FB9;&#x754C;&#x7684;&#x4E09;&#x7EF4;&#x663E;&#x793A;](../../.gitbook/assets/image%20%2831%29.png)

### 三维移动

除了三维显示，GAMA也内置了三维移动的函数，通过给代理添加`moving3D`的技能便能实现三维移动。

```text
species cell skills: [moving3D]{ 
  //实现代理在三维空间中随机移动
  reflex move{
    do wander;
  } 
  //cell族显示为蓝色的半径为1的小球                    
  aspect default {
    draw sphere(1) color: #blue;   
  }
}
```

> **wander**:  `wander`为GAMA内置的运动方式，表现为无目的地漫游。

### 三维连接

最后，我们来实现当代理小球之间的距离小于一定值时，为这些相距较近的代理添加连接线。

```text
species cell skills: [moving3D] {
	...
	//创建一个列表neighbors
	list<cell> neighbors;
	//每次更新列表值为与自身相距10以内的其他代理
	reflex compute_neighbors {
		neighbors <- cell select ((each distance_to self) < 10);
	}

	aspect default {
		draw sphere(environment_size * 0.01) color: #orange;
		//遍历neighbors列表，在列表元素与自身之间连线
		loop pp over: neighbors {
			draw line([self.location, pp.location]);
		}
	}
```

同时，修改实验设置中的显示。

```text
experiment Tuto3D type: gui {
	...
	output {
	  //将显示背景设置为深蓝色
		display View1 type: opengl background: rgb(10, 40, 55) {
			...
		}
	}
}
```

至此，一群在三维空间中随机运动的粒子，且粒子相距较近时会自动连接旁边其他粒子的仿真模型便搭建完毕。

![6.1.3 &#x4E09;&#x7EF4;&#x8FDE;&#x63A5;](../../.gitbook/assets/image%20%2832%29.png)

本节完整代码如下：

```text
model Tuto3D
//全局定义
global {
  //定义cell族代理数量
  int nb_cells <- 100;
  //在全局定义中增加环境大小参数
  int environment_size <-100;
  //定义全局代理的形状为边长为100的立方体
  geometry shape <- cube(environment_size);	
  //初始化模型
  init { 
    //创建cell族
    create cell number: nb_cells { 
      //每个cell代理的初始位置为100x100x100内的随机位置
      location <- {rnd(100), rnd(100), rnd(100)};       
    } 
  }  
} 
//创建cell族
species cell skills: [moving3D]{  
  //创建一个列表neighbors
	list<cell> neighbors;
	//每次更新列表值为与自身相距10以内的其他代理
	reflex compute_neighbors {
		neighbors <- cell select ((each distance_to self) < 10);
	}  
  //实现代理在三维空间中随机移动
  reflex move{
    do wander;
  }
  //cell族显示为蓝色的半径为1的小球                    
  aspect default {
    draw sphere(1) color: #blue; 
    //遍历neighbors列表，在列表元素与自身之间连线
		loop pp over: neighbors {
			draw line([self.location, pp.location]);
		}  
  }
}
//实验设置
experiment Tuto3D  type: gui {
  parameter "Initial number of cells: " var: nb_cells min: 1 max: 1000 category: "Cells" ;	
  output {
    display View1 type: opengl background: rgb(10, 40, 55){
      species cell;
      //在显示中增加环境的显示
      graphics "env" {
        draw cube(environment_size) color: #white empty: true;  
      }
    }
  }
}
```



