# Step3: 动态的道路损耗度模拟

### 路网权值

接下来我们来为道路族添加损耗度属性，来模拟道路在使用中的磨损情况。在此之前我们先了解一下道路权值，在未设定道路权值之前，代理人会寻找最短路径从A点出发前往B点，过程中的道路对其路径的选择并无影响，而在设定道路权值之后，代理人会优先选择权值较高的道路。 简单来说，道路权值越高，会有越多人选择经过这条路。

现在我们为道路添加几个属性：

* **损耗度（destruction\_coeff）：**我们设定损耗度最小为1，最大为2，初始值为1-2之间的随机值。 ****
* **道路颜色（color\)：**取决于道路损耗度，损耗度越高，颜色越红，，损耗度越低，颜色越绿。

```text
species road  {
    //添加属性损耗度，初始值为1-2之间的随机值，最大值为2
    float destruction_coeff <- rnd(1.0,2.0) max: 2.0;
    //设定一个中间变量colorValue,destruction_coeff越大，colorValue越接近255
    int colorValue <- int(255*(destruction_coeff - 1)) update: int(255*(destruction_coeff - 1));
    //设定道路颜色,colorValue越大,颜色越接近红色，反之，越接近绿色
    rgb color <- rgb(min([255, colorValue]),max ([0, 255 - colorValue]),0)  update: rgb(min([255, colorValue]),max ([0, 255 - colorValue]),0) ;
    ...
}
```

然后在全局定义中为路网添加权值：

* **道路权值（weight）：**在道路系统中，道路损耗度越大，说明越多人经过了这条道路，我们将道路权值设定为 **“道路长度x损耗度”。**

```text
    init {
        ...
        create road from: shape_file_roads ;
        //创建一个字典，字典内为每条道路::对应道路的损耗度x长度
        map<road,float> weights_map <- road as_map (each:: (each.destruction_coeff * each.shape.perimeter));
        //通过创建的字典为路网权重赋值
        the_graph <- as_edge_graph(road) with_weights weights_map;
        ...
    }
```

### 路网权值的动态更新

在初始化路网权值之后，我们希望在模拟过程中，路网权值也能得到动态更新，即实现**“走的人越多——路网权值越高——越多人选择走这条路”**这样一个动态过程。

首先，我们定义当一个人走过一条路时，这条路的道路磨损度增加0.02，这个变量用**destroy**表示。

```text
global{
    ...
    //设定一个人经过一条路是道路磨损度的增加值
    float destroy <- 0.02;
    ...
    }
```

接下在人群族的移动行为中，为其每次移动经过的道路更新磨损度。

```text
species people skills: [moving]{
    ...
    reflex move when: the_target != nil {
    //使用return_path返回goto行为所经过的路径
    path path_followed <- goto(target: the_target, on:the_graph, return_path: true);
    //将路径中的元素存储到segments列表
    list<geometry> segments <- path_followed.segments;
    //遍历列表中的每个元素
    loop line over: segments {
        //dist等于此元素的长度
        float dist <- line.perimeter;
        //找到道路族中与此元素符合的代理
        ask road(path_followed agent_from_geometry line) { 
        //更新道路磨损度
        destruction_coeff <- destruction_coeff + (destroy * dist / shape.perimeter);
        }
    }
    if the_target = location {
        the_target <- nil ;
    }
    }
    ...
}   
```

> TIPS：这里更新道路磨损度是使用的**destruction\_coeff + \(destroy \* dist / shape.perimeter\)，**而不是直接使用**destruction\_coeff + destroy \* dist，**这是因为GAMA是根据设定的step更新的，因此一步更新的路径长度**dist=step x speed**，并不会等于道路长度**（shape.perimeter）**，所以这里取路径长度（dist\)和道路长度（shape.perimeter\)的比值乘以每次损耗量作为一步之内更新的道路损耗度。这样当路径长度之和等于道路长度时，即代理人走过这条道路之后，道路损耗量更新量为**destroy**。

实现道路损耗度的动态更新之后，我们再实现路网权值的动态更新。

```text
global {
    ...
    //创建更新路网的函数,此函数每个step运行一次
    reflex update_graph{
        //创建一个字典，字典内为每条道路::对应道路的损耗度x长度
        map<road,float> weights_map <- road as_map (each:: (each.destruction_coeff * each.shape.perimeter));
        //通过创建的字典为路网权重赋值
        the_graph <- the_graph with_weights weights_map;
     }
}
```

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
    //初始人群数量
    int nb_people <- 100;
    //最小上班时间
    int min_work_start <- 6;
    //最大上班时间
    int max_work_start <- 8;
    //最小下班时间
    int min_work_end <- 16; 
    //最大下班时间
    int max_work_end <- 20; 
    //最小速度
    float min_speed <- 1.0 #km / #h;
    //最大速度
    float max_speed <- 5.0 #km / #h; 
    //路网图形
    graph the_graph;
    //设定一个人经过一条路是道路磨损度的增加值
    float destroy <- 0.02;
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
        //创建一个字典，字典内为每条道路::对应道路的损耗度x长度
        map<road,float> weights_map <- road as_map (each:: (each.destruction_coeff * each.shape.perimeter));
        //通过创建的字典为路网权重赋值
        the_graph <- as_edge_graph(road) with_weights weights_map;
        //将建筑族中类型为居住建筑的代理放入列表residential_buildings
        list<building> residential_buildings <- building where (each.type="Residential");
        //将建筑族中类型为工业建筑的代理放入列表residential_buildings
        list<building> industrial_buildings <- building  where (each.type="Industrial") ;
        //创建人群族
        create people number: nb_people {
            //每个代理的速度是最小速度和最大速度之间的随机数
            speed <- rnd(min_speed, max_speed);
            //每个代理的上班时间是最小上班时间和最大上班时间之间的随机数
            start_work <- rnd (min_work_start, max_work_start);
            //每个代理的下班时间是最小下班时间和最大下班时间之间的随机数
            end_work <- rnd(min_work_end, max_work_end);
            //每个代理的生活场所是居住建筑列表中的随机一个
            living_place <- one_of(residential_buildings) ;
            //每个代理的工作场所是工业建筑列表中的随机一个
            working_place <- one_of(industrial_buildings) ;
            //初始化活动状态为休息
            objective <- "resting";
            //初始位置为生活场所的任意位置
            location <- any_location_in (living_place); 
        }
    }
    //创建更新路网的函数,此函数每个step运行一次
    reflex update_graph{
        //创建一个字典，字典内为每条道路::对应道路的损耗度x长度
        map<road,float> weights_map <- road as_map (each:: (each.destruction_coeff * each.shape.perimeter));
        //通过创建的字典为路网权重赋值
        the_graph <- the_graph with_weights weights_map;
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
    //添加属性损耗度，初始值为1-2之间的随机值，最大值为2
    float destruction_coeff <- rnd(1.0,2.0) max: 2.0;
    //设定一个中间变量colorValue,destruction_coeff越大，colorValue越接近255
    int colorValue <- int(255*(destruction_coeff - 1)) update: int(255*(destruction_coeff - 1));
    //设定道路颜色,colorValue越大,颜色越接近红色，反之，越接近绿色
    rgb color <- rgb(min([255, colorValue]),max ([0, 255 - colorValue]),0)  update: rgb(min([255, colorValue]),max ([0, 255 - colorValue]),0) ;
    //显示方式
    aspect base {
    draw shape color: color ;
    }
}
//创建人群组，技能：moving
species people skills: [moving]{
    //显示颜色
    rgb color <- #yellow ;
    //居住地点为建筑族的一员
    building living_place <- nil ;
    //生活地点为建筑族的一员
    building working_place <- nil ;
    //上班时间
    int start_work ;
    //下班时间
    int end_work  ;
    //活动状态
    string objective ; 
    //移动的目的地
    point the_target <- nil ;
    
    //当目前时间为上班时间并且活动状态为休息时
    reflex time_to_work when: current_date.hour = start_work and objective = "resting" {
        //将移动目的地设置为工作场所的任意位置
        the_target <- any_location_in (working_place);
        //将活动状态改为工作
        objective <- "working" ;
    }
    //当目前时间为下班时间并且活动状态为工作时    
    reflex time_to_go_home when: current_date.hour = end_work and objective = "working" {
        //将移动目的地设置为生活场所的任意位置
        the_target <- any_location_in (living_place); 
        //将活动状态改为休息
        objective <- "resting" ;
    } 
    //当移动目的地不为空值时
    reflex move when: the_target != nil {
    	//使用return_path返回goto行为所经过的路径
    	path path_followed <- goto(target: the_target, on:the_graph, return_path: true);
    	//将路径中的元素存储到segments列表
    	list<geometry> segments <- path_followed.segments;
    	//遍历列表中的每个元素
    	loop line over: segments {
        	//dist等于此元素的长度
        	float dist <- line.perimeter;
        	//找到道路族中与此元素符合的代理
        	ask road(path_followed agent_from_geometry line) { 
        	//更新道路磨损度
        	destruction_coeff <- destruction_coeff + (destroy * dist / shape.perimeter);
        	}
    	}
    	//到达目的地时将移动目标设置为空值
    	if the_target = location {
        	the_target <- nil ;
    	}
    }
    //显示方式
    aspect base {
    draw circle(10) color: color border: #black;
    }
}
//实验设置
experiment road_traffic type: gui {
    //将GIS文件位置设置为参数
    parameter "Shapefile for the buildings:" var: shape_file_buildings category: "GIS" ;
    parameter "Shapefile for the roads:" var: shape_file_roads category: "GIS" ;
    parameter "Shapefile for the bounds:" var: shape_file_bounds category: "GIS" ;
    //与人群有关的参数
    parameter "Earliest hour to start work" var: min_work_start category: "People" min: 2 max: 8;
    parameter "Latest hour to start work" var: max_work_start category: "People" min: 8 max: 12;
    parameter "Earliest hour to end work" var: min_work_end category: "People" min: 12 max: 16;
    parameter "Latest hour to end work" var: max_work_end category: "People" min: 16 max: 23;
    parameter "minimal speed" var: min_speed category: "People" min: 0.1 #km/#h ;
    parameter "maximal speed" var: max_speed category: "People" max: 10 #km/#h;
    //与道路有关的参数
    parameter "Value of destruction when a people agent takes a road" var: destroy category: "Road" ;
    //定义输出
    output {
    display city_display type:opengl {
        species building aspect: base ;
        species road aspect: base ;
        species people aspect: base ;
    }
    }
}
```

