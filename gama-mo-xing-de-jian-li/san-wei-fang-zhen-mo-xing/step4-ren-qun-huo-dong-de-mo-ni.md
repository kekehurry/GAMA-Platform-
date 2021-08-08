# Step4: 三维可视化与图表输出

### 三维可视化

现在，我们将GAMA的实时模拟输出从二维转变为三维：

* 为建筑族添加高度信息，并将其显示为三维模型
* 为道路族添加宽度信息
* 将人群组显示为三维球体
* 为模型添加昼夜变换

首先为建筑添加高度属性，其初始值为10m-20m之间的任意值。

```text
species building {
    ...
    //为建筑添加高度属性
    float height <- rnd(10#m, 20#m) ;
    aspect default {
    //在默认显示方式中，设置depth属性，其值为高度
    draw shape color: #gray border: #black depth: height;
    }
}
```

然后设置道路的显示宽度为2m。

```text
species road {
    ...
    //为道路增加属性display_shape，其值为原图形增加2m
    geometry display_shape <- shape + 2.0;
    aspect default {
    //更改默认显示图形为display_shape，设置道路高度为3m
    draw display_shape color: #black depth: 3.0;
    }
}
```

接下来，将人群族显示方式设置为球体。

```text
species people skills:[moving]{     
    ...
    //增加3D显示方式
    aspect sphere3D{
    //因为道路高度为3m，此处注意高度为原高度+3m
    draw sphere(3) at: {location.x,location.y,location.z + 3} color:is_infected ? #red : #green;
    }
}
```

为模型添加全局属性判断昼夜，并在实验设置中增加三维显示方式。

```text
global{
    ...
    //小于7点或者大于20点为夜
    bool is_night <- true update: current_date.hour < 7 or current_date.hour > 20;
    ...
}

experiment main_experiment type:gui{
    ...
    output {
    ...
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
    ...
    }
}
```

> TIPS: 三维显示方式时，显示窗口类型为`opengl`

### 图表输出

最后，我们为模型添加一个感染人数和未被感染人数变化的折线图以及一个数据监管器，监控模拟过程中时间和感染人数的变化。

```text
global{
    ...
    //增加全局变量统计感染人数
    int nb_people_infected <- nb_infected_init update: people count (each.is_infected);
    //增加全局变量统计未感染人数
    int nb_people_not_infected <- nb_people - nb_infected_init update: nb_people - nb_people_infected;
    //感染率为感染人数/总人数
    float infected_rate update: nb_people_infected/nb_people;
    ...
    //当感染率为1时，停止模拟
    reflex end_simulation when: infected_rate = 1.0 {
    do pause;
    }
}


experiment main_experiment type: gui {
    ...
    output {
    //增加当前时间的数据监管器
    monitor "Current hour" value: current_date.hour;
    //增加当前感染人数的数据监管器
    monitor "Infected people rate" value: infected_rate;
    ...
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

![6.4.1 &#x4E09;&#x7EF4;&#x53EF;&#x89C6;&#x5316;&#x754C;&#x9762;](../../.gitbook/assets/image%20%2836%29.png)

![6.4.2 &#x56FE;&#x6807;&#x8F93;&#x51FA;&#x754C;&#x9762;](../../.gitbook/assets/image%20%2835%29.png)

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
    //增加全局变量统计感染人数
    int nb_people_infected <- nb_infected_init update: people count (each.is_infected);
    //增加全局变量统计未感染人数
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
    //为建筑添加高度属性
    float height <- rnd(10#m, 20#m) ;
    aspect default {
    //在默认显示方式中，设置depth属性，其值为高度
    draw shape color: #gray border: #black depth: height;
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

