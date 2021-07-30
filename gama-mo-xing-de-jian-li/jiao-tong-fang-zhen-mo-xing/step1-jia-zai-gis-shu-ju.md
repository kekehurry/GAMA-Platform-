# Step1: 加载GIS数据

首先准备好进行模拟的GIS数据，本教程所使用的GIS数据可以在**GAMA导航栏**&gt;**Library models**&gt;**Tutorials**&gt;**Road Traffic**&gt;**includes**中找到，将其复制到新建项目文件夹中的includes文件夹即可，也可从[官网Github](https://github.com/gama-platform/gama/tree/master/msi.gama.models/models/Tutorials/Road%20Traffic/includes)下载。

### 基本模型

首先，和之前一样我们先将基本模型框架搭建出来。

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
    //定义输出
    output {
    display city_display type:opengl {
        species building aspect: base ;
        species road aspect: base ;
    }
    }
}
```

> **Step** : GAMA中默认的一步等于一秒，可以理解为GAMA里的时间每次更新一秒，可以通过重写全局变量**step**, 来更改GAMA模拟的速度，这里我们将step设置为10分钟，这样GAMA中的时间每次更新10分钟。  
>   
> **\#mn** : GAMA中的单位表示方式是`#+单位名`，如 `#km` 、`#m`等，更多单位详见官方文档中[Units and constants](https://gama-platform.github.io/wiki/UnitsAndConstants)。

### 加载GIS数据

和之间教程的**image\_file**类似，在全局定义中通过**file**关键字可以直接加载**shp**格式文件。这里我们通过GIS文件加载建筑数据、路网数据以及模拟边界。

```text
global{
    ...
    //读取GIS文件
    file shape_file_buildings <- file("../includes/building.shp");
    file shape_file_roads <- file("../includes/road.shp");
    file shape_file_bounds <- file("../includes/bounds.shp");
}
```

### 通过GIS数据创建代理

GIS数据是带属性信息的矢量文件，通过GIS数据创建代理时，我们不仅需要读取其图形数据，还需要与图形数据相关联的属性信息。

```text
global {
    ...
    init {
        //通过读取的建筑GIS文件创建建筑族，建筑族的type属性，读取自GIS文件的"NATURE"字段
        create building from: shape_file_buildings with: [type::read ("NATURE")] {
            //若建筑类型为工业建筑，显示为蓝色
            if type="Industrial" {
                color <- #blue ;
            }
        }
        //通过读取的道路GIS文件创建道路族
        create road from: shape_file_roads ;
    }
}
```

> **create ... form ... with:**  from 后接GIS数据，with 后接一个字典，字典内为属性名与其属性值。此处通过**read** 读取GIS文件中的NATURE字段作为属性值。

### 模拟范围

因为GIS文件可能偏离世界坐标原点非常远，因此我们需要定义一个模拟范围来帮助可视化窗口迅速定位模拟的位置，定义模拟范围也非常简单，只需为全局代理的shape变量定义一个图形。

```text
global {
    ...
    //为全局代理的shape变量定义一个图形
    geometry shape <- envelope(shape_file_bounds); 
    ...
}
```

> **envelop** : envelope函数可以自动求出包含给定图形的矩形。

如此，一个通过GIS数据创建的基本模型便已经搭建完毕。

![5.1.1 &#x4EA4;&#x901A;&#x4EFF;&#x771F;&#x57FA;&#x672C;&#x6A21;&#x578B;](../../.gitbook/assets/image%20%2823%29.png)

本节完整代码如下：

```text
//模型名称
model tutorial_gis_city_traffic

//全局定义
global {
    //定义模型时间
    float step <- 10 #mn;
    //读取GIS文件
    file shape_file_buildings <- file("../includes/building.shp");
    file shape_file_roads <- file("../includes/road.shp");
    file shape_file_bounds <- file("../includes/bounds.shp");
    //为全局代理的shape变量定义一个图形
    geometry shape <- envelope(shape_file_bounds); 
    //初始化
    init {
        //通过读取的建筑GIS文件创建建筑族，建筑族的type属性，读取自GIS文件的"NATURE"字段
        create building from: shape_file_buildings with: [type::read ("NATURE")] {
            //若建筑类型为工业建筑，显示为蓝色
            if type="Industrial" {
                color <- #blue ;
            }
        }
        //通过读取的道路GIS文件创建道路族
        create road from: shape_file_roads ;
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
    //将GIS文件位置设置为参数
    parameter "Shapefile for the buildings:" var: shape_file_buildings category: "GIS" ;
    parameter "Shapefile for the roads:" var: shape_file_roads category: "GIS" ;
    parameter "Shapefile for the bounds:" var: shape_file_bounds category: "GIS" ;
    //定义输出
    output {
    display city_display type:opengl {
        species building aspect: base ;
        species road aspect: base ;
    }
    }
}
```



