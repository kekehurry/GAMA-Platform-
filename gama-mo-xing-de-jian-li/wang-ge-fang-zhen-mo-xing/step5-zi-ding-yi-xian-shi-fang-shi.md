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

![&#x98DF;&#x8349;&#x52A8;&#x7269;&#x56FE;&#x6807;](../../.gitbook/assets/image%20%2810%29.png)

食草动物图标如上，为了代码生效，需将食草动物图标下载到项目文件夹的 `includes/data/`  文件夹中，并将图片名称修改为`predator_prey_sheep.png`

### 子族-食肉动物

```text
species predator parent: generic_species {
    ...
    //定义my_icon 为狼的图标
    image_file my_icon <- image_file("../includes/data/predator_prey_wolf.png") ;
    ...
}
```

![&#x98DF;&#x8089;&#x52A8;&#x7269;&#x56FE;&#x6807;](../../.gitbook/assets/image%20%2811%29.png)

同样的，将食肉动物图标下载到项目文件夹的 `includes/data/`  文件夹中，并将图片名称修改为`predator_prey_wolf.png`

### 实验显示

接下来更改**experiment**中的显示方式：

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

