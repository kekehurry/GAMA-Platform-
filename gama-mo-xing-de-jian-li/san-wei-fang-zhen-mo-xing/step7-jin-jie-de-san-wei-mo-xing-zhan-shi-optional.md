# Step7: 进阶的三维模型展示（Optional）

### 显示道路中心线

通过道路端点画出道路中心线。



![6.7.1 &#x9053;&#x8DEF;&#x663E;&#x793A;](../../.gitbook/assets/image%20%2849%29.png)

```text
species road {
...
    //显示道路中心线
    aspect geom3D {
    draw display_shape color: #grey;
    draw line(shape.points, 0.5) color: #black;
    }
...
}
```

### 显示建筑肌理

使用纹理图像赋予三维物体材质。

![6.2 &#x5EFA;&#x7B51;&#x808C;&#x7406;](../../.gitbook/assets/image%20%2841%29.png)

```text
species building {
    ....
    //使用屋顶和立面两种纹理图像赋予建筑材质
    aspect geom3D {
    draw shape depth: 20 #m border: #black texture: ["../includes/roof_top.png","../includes/texture.jpg"];
    }
}
```

### 人群显示

使用obj文件显示人群的三维形体。

![6.7.3 &#x4E09;&#x7EF4;&#x4EBA;&#x5F62;&#x5C55;&#x793A;](../../.gitbook/assets/image%20%2835%29.png)

```text
species people skills:[moving]{
    ....
    //使用obj文件定义人群显示
    aspect geom3D {
    if target != nil {
        draw obj_file("../includes/people.obj", 90::{-1,0,0}) size: 5
        at: location + {0,0,7} rotate: heading - 90 color: is_infected ? #red : #green;
    }
    }
}
```

最后在实验设置中更改显示方式：

```text
experiment main_experiment type: gui {
    ...
    output {
    ...
    display map_3D_plus type: opengl {
        //设置全局照明，如果是晚上亮度为50，白天为255
        light 1 color:(is_night ? 50 : 255);
        //添加背景图
        image "../includes/soil.jpg";
        //设置显示方式
        species road aspect: geom3D;
        species people aspect: geom3D;          
        species building aspect: geom3D;
    }
    }
}
```

![6.7.4 &#x8FDB;&#x9636;&#x7684;&#x4E09;&#x7EF4;&#x663E;&#x793A;](../../.gitbook/assets/image%20%2844%29.png)

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
    //传播率
    float beta <- 0.01;
    //时间步长
    float h <- 0.1;
    //易感人群数量
    float S;
    //感染人群数量
    float I;
    //总人群数量
    float T;
    //方程时间变量
    float t;   
    //保存微分方程中的小数部分
    float I_to1;
    
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
    
    //定义微分方程SI
    equation SI{ 
    //S对t的导数方程
    diff(S,t) = (- beta * S * I / T) ;
    //I对t的导数方程
    diff(I,t) = (  beta * S * I / T) ;
    }
    
    //当建筑内总人数大于1时
    reflex epidemic when: nb_total > 0 {
    //总人数为T
    T <- float(nb_total);
    //易感人数S为总人数-感染人数
    S <- float(nb_total - nb_infected);
    //感染人数为I
    I <- T - S;
    //I0等于当前感染人数
    float I0 <- I;
    //当建筑内易感人数S和感染人数I大于0时
    if (I > 0 and S > 0) {
        //求解微分方程SI,求解方法为 Runge-Kutta 4，步长为h
        solve SI method: #rk4 step_size: h;
        //I_to1为新增感染人数+之前的小数部分
        I_to1 <- I_to1 + (I - I0);
        //I_int为新增感染人数和当前易感人数中的最小值取整
        int I_int <- min([int(S), int(I_to1)]);
        //设置I_to1为新增感染人数-取整后的新增感染人数（保存小数部分）
        I_to1 <- I_to1 - I_int;
        //将未感染中的I_int个人设置为感染
        ask (I_int among (people_in_building where (!each.is_infected))) {
        is_infected <- true;
        }
    }
    }
    
    //更改建筑显示方式
    aspect default {
    //无人为灰，感染率小于0.5为绿，感染率大于0.5为红
    draw shape color: nb_total = 0 ? #gray : (float(nb_infected) / nb_total > 0.5 ? #red : #green) border: #black depth: height;
    }
    aspect geom3D {
    draw shape depth: 20 #m border: #black texture: ["../includes/roof_top.jpg","../includes/texture.jpg"];
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
    //显示道路中心线
    aspect geom3D {
    draw display_shape color: #grey;
    draw line(shape.points, 0.5) color: #black;
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
    //使用obj文件定义人群显示
    aspect geom3D {
    if target != nil {
        draw obj_file("../includes/people.obj", 90::{-1,0,0}) size: 5
        at: location + {0,0,7} rotate: heading - 90 color: is_infected ? #red : #green;
    }
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
        species road ;
        species people aspect: sphere3D;  
        //为建筑添加50%的透明度          
        species building transparency: 0.5;
    }
    display map_3D_plus type: opengl {
        //设置全局照明，如果是晚上亮度为50，白天为255
        light 1 color:(is_night ? 50 : 255);
        //添加背景图
        image "../includes/soil.jpg";
        //设置显示方式
        species road aspect: geom3D;
        species people aspect: geom3D;          
        species building aspect: geom3D;
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

