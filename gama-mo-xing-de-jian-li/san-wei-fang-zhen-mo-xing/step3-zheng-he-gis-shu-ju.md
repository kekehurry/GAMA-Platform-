# Step3: 整合GIS数据

现在，我们在之前简单的传播模型基础上，结合交通仿真的知识，载入GIS数据并使代理人在不同建筑之间沿着路网移动，完成一个城市环境的病毒传播模拟。

### 载入GIS数据并初始化

```text
global {
    //载入GIS数据
    file roads_shapefile <- file("../includes/road.shp");
    file buildings_shapefile <- file("../includes/building.shp");
    //将全局代理形状设置为包括所有道路的矩形
    geometry shape <- envelope(roads_shapefile); 
    ...
    init{
    //从GIS数据创建建筑和道路
    create road from: roads_shapefile;
    create building from: buildings_shapefile;
    //代理人的初始位置设置为随机建筑的任意位置
    create people number:nb_people {
            speed <- agent_speed;
        location <- any_location_in(one_of(building));
    }
    }
}
```

接下来，创建道路和建筑族。

```text
//创建建筑族
species building {
    aspect default {
    draw shape color: #gray border: #black;
    }
}
//创建道路族
species road {
    aspect default {
    draw shape color: #black;
    }
}
```

然后在实验设置中设置显示建筑与道路族。

```text
experiment main_experiment type:gui{
    ...
    output {
    display map {
        species road ;
        species building ;
        species people ;            
    }
    }
}
```

### 编写人群的移动行为

我们将人群的移动简化为人群在接近9点、12点、18点时为移动高峰期，其他时间呆在建筑里。这里我们引入两个变量： **`staying_counter`** 和  **`staying_coeff` 。**

* **`staying_counter`：**当代理人达到建筑时开始计数，随着时间递增，开始移动时恢复为0。
* **`staying_coeff` ：**接近9点、12点、18点时数值最小，在三个时间段之间递增。
* **`staying_counter/staying_coeff`:** 代理人开始移动的概率，随着代理人在建筑中的停留时间递增，并在9点、12点、18点时概率最大。

首先在全局定义中，定义**`staying_coeff`** 以及路网**`road_network`**，并在创建道路后初始化路网。

```text
global{
    ...
    //路网
    graph road_network;
    //定义staying_coeff，随着时间递增，并在9点、12点、18点时数值最小
    float staying_coeff update: 10.0 ^ (1 + min([abs(current_date.hour - 9), abs(current_date.hour - 12), abs(current_date.hour - 18)]));
    ...
    init{
    create road from: roads_shapefile;
    //初始化路网
    road_network <- as_edge_graph(road);
    }
}
```

然后在人群族中，为代理人编写**`stay`**和**`move`**行为。

```text
species people skills: [moving] {   
    //增加属性目标点和stay_counter
    point target;
    int staying_counter;    
    ...
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
    ...
}
```

![6.3.1 &#x6574;&#x5408;GIS&#x6570;&#x636E;](../../.gitbook/assets/image%20%2833%29.png)

本节完整代码如下：

```text
model SI_city

global {
    //模拟设置
    float step <- 1 #minutes;
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
}

//创建建筑族
species building {
    aspect default {
    draw shape color: #gray border: #black;
    }
}
//创建道路族
species road {
    aspect default {
    draw shape color: #black;
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
}

//实验设置
experiment main_experiment type: gui {
    //参数设置
    parameter "Infection distance" var: infection_distance;
    parameter "Proba infection" var: proba_infection min: 0.0 max: 1.0;
    parameter "Nb people infected at init" var: nb_infected_init;
    //输出
    output {
    //显示设置
    display map {
        species road ;
        species building ;
        species people ;       
    }
    }
}
```

