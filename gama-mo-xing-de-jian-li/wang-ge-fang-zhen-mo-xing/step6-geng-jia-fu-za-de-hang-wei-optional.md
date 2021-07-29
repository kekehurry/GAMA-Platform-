# Step6: 更加复杂的行为（Optional）

现在一个简单的捕食者仿真模型已经基本搭建完成，我们还可以优化一下，实现更加复杂的行为。

![4.6.1 &#x6355;&#x98DF;&#x8005;&#x4EFF;&#x771F;&#x6A21;&#x578B;](../../.gitbook/assets/image%20%2820%29.png)

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
	
```

> TIPS: 图像文件读取后可以直接看成一个matrix多维数组进行操作，通过matix\[i,j\]可以读取图像在第i行第j列数据的值。这里通过matrix at {i,j} 返回第i行第j列的值。

如此，草地初始化为与图像一致的分布。





![4.6.2 &#x4F18;&#x5316;&#x540E;&#x7684;&#x6355;&#x98DF;&#x8005;&#x4EFF;&#x771F;&#x6A21;&#x578B;](../../.gitbook/assets/image%20%2817%29.png)

### 向附近的食物移动

之前在编写食草与食肉动物的移动时，我们移动的逻辑是向周围相距为2的网格随机移动，现在我们利用条件语句实现更加复杂的移动逻辑。



