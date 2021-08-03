# GAMA的三维特性

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

![6.1.1 &#x7B80;&#x5355;&#x7684;&#x4E09;&#x7EF4;&#x663E;&#x793A;](../../.gitbook/assets/image%20%2828%29.png)

### 三维移动

### 三维连接

