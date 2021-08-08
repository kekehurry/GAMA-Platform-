# Step5: 多层次代理的交互

目前为止，我们编写的代理行为都是代理本身来控制，那么如何实现代理族B来控制代理族A的行为呢？比如，我们为建筑添加一个人群管控的功能，当人群进入建筑时，其行为受到管控，直到建筑允许放行，才能从建筑中出来，要实现这样的复杂交互，我们要做的有以下几点：

* 为建筑添加人群管控行为： **`let_people_leave`** 和 **`let_people_enter`**。
* 在**建筑族**中创建**人群族的子族群** **`people_in_building`** ，当人群进入建筑后，变为建筑族的一员，但保留了人群族的所有属性。

```text
species building {
    ...
    //在建筑族中添加人群族的子族群people_in_building，其行为为空
    species people_in_building parent: people schedules: [] {
    }
    ...
    //定义人群进入建筑的行为
    reflex let_people_enter {
    //捕获进入建筑的人群，将其改为子族群people_in_building的一员
    capture (people inside self where (each.target = nil)) as: people_in_building;
    }
    //定义人群离开建筑的行为
    reflex let_people_leave {
    //计算进入建筑的人的停留时长
    ask people_in_building {
        staying_counter <- staying_counter + 1;
    }
    //以staying_counter / staying_coeff的概率释放子族群people_in_building的代理至世界代理中的people族
    release people_in_building where (flip(each.staying_counter / staying_coeff)) as: people in: world {
        //其目标为随机一栋建筑的任意地点
        target <- any_location_in(one_of(building));
    }
    }
    ...
}
```

> **`schedules`：**可以理解为时间调度表，将schedules设置为空，则代理与时间有关的行为均失效。

现在，我们将建筑的显示方式改为：

* 建筑中没有人时显示为灰色
* 建筑中感染人数比例小于0.5时显示为绿色
* 建筑中感染人数比例大于0.5时显示为红色

```text
species building {
    //统计建筑中感染人群的数量
    int nb_infected <- 0 update: self.people_in_building count each.is_infected;
    //统计建筑中所有人的数量
    int nb_total <- 0 update: length(self.people_in_building);
    //更改建筑显示方式
    aspect default {
    draw shape color: nb_total = 0 ? #gray : (float(nb_infected) / nb_total > 0.5 ? #red : #green) border: #black depth: height;
    }
}

```

因为人群分离出了子族群，因此我们在全局定义中统计感染人数的方法也需要更新。

```text
global  {
    ...
    //返回people_in_building族的列表
    list<people_in_building> list_people_in_buildings update: (building accumulate each.people_in_building);
    //统计people族和所有people_in_building族中感染人数
    int nb_people_infected <- nb_infected_init update: (people + list_people_in_buildings) count (each.is_infected);
    ...
}
```

> **`accumulate`**：accumulate 和 collect 类似，区别在与accumulate 返回的是列表，collect 返回值可以是任何容器。

![6.5.1 &#x5728;&#x5EFA;&#x7B51;&#x65CF;&#x4E2D;&#x521B;&#x5EFA;&#x4EBA;&#x7FA4;&#x5B50;&#x65CF;](../../.gitbook/assets/image%20%2836%29.png)

本节完整代码如下：

```text
model SI_city

global {
    //模拟设置
    float step <- 1 #minutes;
    //小于7点或者大于20点为夜
    bool is_night <- true update: current_date.hour < 7 or current_date.hour > 20;
    //载入GIS数据
    file roads_shapefile <- file("../includes/road.shp");
    file buildings_shapefile <- file("../includes/building.shp");
    //将全局代理形状设置为包括所有道路的矩形
    geometry shape <- envelope(roads_shapefile);
    //初始人群数量
    int nb_people <- 500;
    //人群速度
    float agent_speed <- 5.0 #km/#h;  
    //感染距离  
    float infection_distance <- 2.0 #m;
    //感染概率
    float proba_infection <- 0.05;
    //初始感染人数
    int nb_infected_init <- 5;
    //路网
    graph road_network;
    //定义staying_coeff，随着时间递增，并在9点、12点、18点时数值最小
    float staying_coeff update: 10.0 ^ (1 + min([abs(current_date.hour - 9), abs(current_date.hour - 12), abs(current_date.hour - 18)]));
    
    //返回people_in_building族的列表
    list<people_in_building> list_people_in_buildings update: (building accumulate each.people_in_building);
    //统计people族和所有people_in_building族中感染人数
    int nb_people_infected <- nb_infected_init update: (people + list_people_in_buildings) count (each.is_infected);
    //统计未感染人数
    int nb_people_not_infected <- nb_people - nb_infected_init update: nb_people - nb_people_infected;
    //感染率为感染人数/总人数
    float infected_rate update: nb_people_infected/nb_people;
    //初始化
    init {
    create road from: roads_shapefile;
    create building from: buildings_shapefile;
    //初始化路网
    road_network <- as_edge_graph(road);
    create people number:nb_people {
            speed <- agent_speed;
        location <- any_location_in(one_of(building));
    }
    //在人群族中随机寻找nb_infected_init人
    ask nb_infected_init among people {
        //将其感染状态设置为真
        is_infected <- true;
    }
    }
    //当感染率为1时，停止模拟
    reflex end_simulation when: infected_rate = 1.0 {
    do pause;
    }
}

//创建建筑族
species building {
    //在建筑族中添加人群族的子族群people_in_building，其行为为空
    species people_in_building parent: people schedules: [] {
    }
    //统计建筑中感染人群的数量
    int nb_infected <- 0 update: self.people_in_building count each.is_infected;
    //统计建筑中所有人的数量
    int nb_total <- 0 update: length(self.people_in_building);
    //为建筑添加高度属性
    float height <- rnd(10#m, 20#m) ;
    //定义人群进入建筑的行为
    reflex let_people_enter {
    //捕获进入建筑的人群，将其改为子族群people_in_building的一员
    capture (people inside self where (each.target = nil)) as: people_in_building;
    }
    //定义人群离开建筑的行为
    reflex let_people_leave {
    //计算进入建筑的人的停留时长
    ask people_in_building {
        staying_counter <- staying_counter + 1;
    }
    //以staying_counter / staying_coeff的概率释放子族群people_in_building的代理至世界代理中的people族
    release people_in_building where (flip(each.staying_counter / staying_coeff)) as: people in: world {
        //其目标为随机一栋建筑的任意地点
        target <- any_location_in(one_of(building));
    }
    }
    //更改建筑显示方式
    aspect default {
    //无人为灰，感染率小于0.5为绿，感染率大于0.5为红
    draw shape color: nb_total = 0 ? #gray : (float(nb_infected) / nb_total > 0.5 ? #red : #green) border: #black depth: height;
    }
}
//创建道路族
species road {
    //为道路增加属性display_shape，其值为原图形增加2m
    geometry display_shape <- shape + 2.0;
    aspect default {
    //更改默认显示图形为display_shape，设置道路高度为3m
    draw display_shape color: #black depth: 3.0;
    }
}

//人群族定义
species people skills: [moving] {
    //是否感染
    bool is_infected <- false;
    //增加属性目标点和stay_counter
    point target;
    int staying_counter;
   //当没有目标时停留
    reflex stay when: target = nil {
    //staying_counter随时间递增
    staying_counter <- staying_counter + 1;
    //以staying_counter / staying_coeff的概率将目标变为随机建筑中的任意地点
    if flip(staying_counter / staying_coeff) {
        target <- any_location_in (one_of(building));
    }
    }
    //当有目标时开始移动
    reflex move when: target != nil{
    //沿着路网向目标点出发
    do goto target: target on: road_network;
    //达到目标点时将目标设为空值，将staying_counter清零
    if (location = target) {
        target <- nil;
        staying_counter <- 0;
    } 
    }
    //当代理人被感染时
    reflex infect when: is_infected {
    //寻找与其相距infection_distance的其他代理
    ask people at_distance infection_distance {
        //以proba_infection的概率感染
        if (flip(proba_infection)) {
        is_infected <- true;
        }
    }
    }
    //默认显示设置
    aspect default {
    //显示为半径为5的圆形，感染者为红色，未感染者为绿色
    draw circle(5) color: is_infected ? #red : #green;
    }
    //增加3D显示方式
    aspect sphere3D{
    //因为道路高度为3m，此处注意高度为原高度+3m
    draw sphere(3) at: {location.x,location.y,location.z + 3} color:is_infected ? #red : #green;
    }
}

//实验设置
experiment main_experiment type: gui {
    //参数设置
    parameter "Infection distance" var: infection_distance;
    parameter "Proba infection" var: proba_infection min: 0.0 max: 1.0;
    parameter "Nb people infected at init" var: nb_infected_init;
    //输出
    output {
    //增加当前时间的数据监管器
    monitor "Current hour" value: current_date.hour;
    //增加当前感染人数的数据监管器
    monitor "Infected people rate" value: infected_rate;
    //显示设置
    display map {
        species road ;
        species building ;
        species people ;       
    }
    //将显示类型设置为opengl
    display map_3D type: opengl {
        //设置全局照明，如果是晚上亮度为50，白天为255
        light 1 color:(is_night ? 50 : 255);
        //添加背景图
        image "../includes/soil.jpg";
        //设置显示方式
        species road;
        species people aspect: sphere3D;  
        //为建筑添加50%的透明度          
        species building transparency: 0.5;
    }
    //增加感染人数和未感染人数的折线图
    display chart refresh: every(10#cycles) {
        chart "Disease spreading" type: series style: spline {
        data "susceptible" value: nb_people_not_infected color: #green;
        data "infected" value: nb_people_infected color: #red;
        }
    }
    }
}
```



