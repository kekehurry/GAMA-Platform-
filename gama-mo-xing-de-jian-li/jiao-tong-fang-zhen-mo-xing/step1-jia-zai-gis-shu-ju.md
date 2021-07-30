# Step1: 加载GIS数据

首先准备好进行模拟的GIS数据，本教程所使用的GIS数据可以在**GAMA导航栏**&gt;**Library models**&gt;**Tutorials**&gt;**Road Traffic**&gt;**includes**中找到，将其复制到新建项目文件夹中的includes文件夹即可，也可从[官网Github](https://github.com/gama-platform/gama/tree/master/msi.gama.models/models/Tutorials/Road%20Traffic/includes)下载。

### 基本模型

首先，和之前一样我们现将包含三种代理的基本模型框架搭建出来。

```text
//模型名称
model tutorial_gis_city_traffic

//全局定义
global {
    //定义模型时间
    float step <- 10 #mn;
    //初始化
    init {
    }
}
//创建建筑族
species building {
    //类型属性
    string type; 
    //颜色
    rgb color <- #gray  ;
    //定义显示方式
    aspect base {
    draw shape color: color ;
    }
}
//创建道路族
species road  {
    //颜色
    rgb color <- #black ;
    //显示方式
    aspect base {
    draw shape color: color ;
    }
}

//实验设置
experiment road_traffic type: gui {
    output {
    display city_display type:opengl {
        species building aspect: base ;
        species road aspect: base ;
    }
    }
}
```

> **Step** : GAMA中默认的一步等于一秒，可以理解为GAMA里的时间每次更新一秒，可以通过重写全局变量**step**, 来更改GAMA模拟的速度，这里我们将step设置为10分钟，这样GAMA中的时间每次更新10分钟。  
> **\#mn** : GAMA中的单位表示方式是`#+单位名`，如 `#km` 、`#m`等，更多单位详见官方文档中[Units and constants](https://gama-platform.github.io/wiki/UnitsAndConstants)。



### 加载GIS数据

和之间教程的**image\_file**类似，在全局定义中通过**file**关键字可以直接加载**shp**格式文件。

```text
global{
    ...
    file shape_file_buildings <- file("../includes/building.shp");
    file shape_file_roads <- file("../includes/road.shp");
    file shape_file_bounds <- file("../includes/bounds.shp");
}
```



