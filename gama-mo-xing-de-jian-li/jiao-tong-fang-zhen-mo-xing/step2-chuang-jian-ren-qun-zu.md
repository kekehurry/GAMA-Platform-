# Step2:  人群代理的编写

### 创建人群族

首先，我们将人群族的基本属性编写完成。

```text
global {
    ...
    //初始人群数量
    int nb_people <- 100;
    
    init {
    ...
        
    //初始化人群族，初始位置为随机一个居住建筑内的任意位置
    create people number: nb_people {
        location <- any_location_in (one_of (residential_buildings));
    }
    }
}

species people {
    //显示颜色
    rgb color <- #yellow ;
    //显示方式
    aspect base {
    draw circle(10) color: color border: #black;
    }
}

experiment road_traffic type: gui {
    output {
    ...
    display city_display type:opengl {
        ...
        //增加人群族的显示
        species people aspect: base ;
    }
    }
}
```

### 编写人群行为

我们希望人群在上班时间沿着道路出发去工作地点，下班时间从工作地点回到居住地点，为了实现这些行为，我们为人群族添加必要的技能**moving**，然后为人群族增加必要的属性。

```text
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
    //路网图形（之后会在初始化时赋值）
    graph the_graph;
    ...
}
```

> **skills：** skills是GAMA为仿真模型预设的行为模块，每个skill都包含了相应的属性和行为动作，以**moving**为例，moving包含了**speed**、**heading**、**destination**等属性以及**move**、**goto**、**follow**、**wander**、**wander\_3D**等内置行为函数，使用者可以更加方便快捷地完成复杂行为设计，而无需从零编写。更多skills相关，可查看官方文档-[Attaching Skills](https://gama-platform.github.io/wiki/AttachingSkills)。

接下来，我们编写具体的移动规则。

```text
species people skills: [moving]{  
    ...
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
    //沿着道路向目的地移动
    do goto target: the_target on: the_graph ; 
    //到达目的地时将移动目标设置为空值
    if the_target = location {
        the_target <- nil ;
    }
    }
    ...
}
```

> **current\_date** : current\_date是一个内置的全局变量，调用current\_date系统会根据step的值返回当前时间，**current\_date.hour**即返回当前的小时数。  
>   
> **do goto target: ... on: ...** : goto是skill的内置函数，代理将会沿着关键字**on**所设置的图形以最短路径前往目的地。这里the\_graph是一个全局变量，我们会在初始化时为它赋值，详见下一小节。

### 初始化人群族

接下来，我们在全局设置中初始化人群族，为了仿真模型更好的模拟真实状态，我们通过初始化为人群族中每个代理分配不同的速度、工作时间、下班时间、生活地点与居住地点，实现代码如下：

```text
global {
    ...
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
    init {
    ...  
    //通过道路族创建路网图形用于人群族的移动
    the_graph <- as_edge_graph(road);
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
}
```

> **as\_edge\_graph\(...\)** : 通过给定的边的列表，创建一个由边组成的网状图形，更多使用方式，详见[官方文档](https://gama-platform.github.io/wiki/OperatorsAA#as_edge_graph)。

### 实验设置

最后我们在实验设置中，设置好可调整参数，并设置好人群组的显示。

```text
experiment road_traffic type: gui {
    ...
    //与人群有关的参数
    parameter "Earliest hour to start work" var: min_work_start category: "People" min: 2 max: 8;
    parameter "Latest hour to start work" var: max_work_start category: "People" min: 8 max: 12;
    parameter "Earliest hour to end work" var: min_work_end category: "People" min: 12 max: 16;
    parameter "Latest hour to end work" var: max_work_end category: "People" min: 16 max: 23;
    parameter "minimal speed" var: min_speed category: "People" min: 0.1 #km/#h ;
    parameter "maximal speed" var: max_speed category: "People" max: 10 #km/#h;
    ...
    output {
    display city_display type: opengl {
        ...
        species people aspect: base ;
    }
    }
}
```

至此一个基本的交通仿真模型编写完成，本节最终代码如下：

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
        //通过道路族创建路网图形用于人群族的移动
    the_graph <- as_edge_graph(road);
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
    //路网图形（之后会在初始化时赋值）
    graph the_graph;
    
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
    //沿着道路向目的地移动
    do goto target: the_target on: the_graph ; 
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

