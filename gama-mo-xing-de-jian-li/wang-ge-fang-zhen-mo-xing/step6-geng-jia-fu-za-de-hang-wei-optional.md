# Step6: 更加复杂的行为（Optional）

现在一个简单的捕食者仿真模型已经基本搭建完成，我们还可以优化一下，实现更加复杂的行为。

![](../../.gitbook/assets/image%20%2819%29.png)

### 通过图像控制网格分布

现在的模型中，草地网格的初始分布时随机的，为了让其更加符合自然状态，我们通过图像的数值来控制网格中食物量的多少，使得草地的分布和图像一致。

原始图像（[下载链接](https://gama-platform.github.io/resources/images/tutorials/predator_prey_raster_map.png)）如下，原始图像大小为50x50，和草地网格的大小一致

![4.6.1 &#x8349;&#x5730;&#x5206;&#x5E03;&#x56FE;&#x50CF;](../../.gitbook/assets/image%20%2812%29.png)

```text
global {
	...
	//读取图像文件
	file map_init <- image_file("../includes/data/raster_map.png");
	//初始化
	init {
		...
		//通过ask函数在初始化时修改vegetation_cell的属性
		ask vegetation_cell {
		//grid_x,grid_y时vegetation_cell的内置属性，返回其位置
		
    color <- rgb (map_init at {grid_x,grid_y}) ;
    food <- 1 - (((color as list) at 0) / 255) ;
    food_prod <- food / 100 ; 
    }
	}
	
```

> TIPS: 图像文件读取后可以直接看成一个matrix二维数组进行操作，通过matix\[i,j\]可以读取图像在第i行第j列数据的值。

### 向附近的食物移动

