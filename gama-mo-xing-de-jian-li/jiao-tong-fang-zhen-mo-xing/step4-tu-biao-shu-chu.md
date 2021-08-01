# Step4: 图表输出

为了更直观了解模型运行状况，我们为仿真模型创建图标输出：

* 最大道路损耗度折线图
* 平均道路损耗度折现图
* 人群活动状态饼状图

```text
experiment road_traffic type: gui {
    ...
    display chart_display refresh: every(10#cycles) { 
        //创建图表Road Status，类型为折线图，大小为宽1，高0.5,左上角位置为0，0
        chart "Road Status" type: series size: {1, 0.5} position: {0, 0} {
            //第一系列数据为道路平均损耗度
            data "Mean road destruction" value: mean (road collect each.destruction_coeff) style: line color: #green ;
            //第二系列数据为道路最大损耗度
            data "Max road destruction" value: road max_of each.destruction_coeff style: line color: #red ;
        }
        //创建图表People Objectif，类型为饼图，大小为宽1，高0.5,左上角位置为0，0.5
        chart "People Objectif" type: pie style: exploded size: {1, 0.5} position: {0, 0.5}{
            //第一系列数据为工作人群数量
            data "Working" value: people count (each.objective="working") color: #magenta ;
            //第一系列数据为休息人群数量
            data "Resting" value: people count (each.objective="resting") color: #blue ;
        }
    }
    }
}
```

> **mean\(**`container`**\)、max\(**`container`**\)**: 返回列表container的平均值/最大值。  
>   
> `container` **mean\_of/max\_of** `expression`: 返回对container元素进行expression操作后的平均值/最大值。  
>   
> `container` **collect** `expression`: 返回对列表container元素进行expression操作后的新列表。  
>   
> `container` **count** `expression:` 返回列表container中满足expression条件的元素的数量。

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
    display chart_display refresh: every(10#cycles) { 
        //创建图表Road Status，类型为折线图，大小为宽1，高0.5,左上角位置为0，0
        chart "Road Status" type: series size: {1, 0.5} position: {0, 0} {
            //第一系列数据为道路平均损耗度
            data "Mean road destruction" value: mean (road collect each.destruction_coeff) style: line color: #green ;
            //第二系列数据为道路最大损耗度
            data "Max road destruction" value: road max_of each.destruction_coeff style: line color: #red ;
        }
        //创建图表People Objectif，类型为饼图，大小为宽1，高0.5,左上角位置为0，0.5
        chart "People Objectif" type: pie style: exploded size: {1, 0.5} position: {0, 0.5}{
            //第一系列数据为工作人群数量
            data "Working" value: people count (each.objective="working") color: #magenta ;
            //第一系列数据为休息人群数量
            data "Resting" value: people count (each.objective="resting") color: #blue ;
        }
    }
    }
}
```

