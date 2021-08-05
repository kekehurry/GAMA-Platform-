# Step2: 简单的传播模拟

接下来，我们来完成一个有三维可视化的病毒传播模型。先从最简单的开始——我们忽略掉复杂的环境，完成病毒传染的逻辑：人群中有被感染者和未被感染者两种人，当未被感染者距离被感染者一定距离时，有一定概率被感染。

### 全局定义

首先，我们在全局定义中，申明**感染距离（infection\_distance ）、感染概率（proba\_infection）、初始人群数量（nb\_people）、初始感染人数（nb\_infected\_init）**等全局变量，并初始化人群代理。

```text
global {
    //模拟设置
    float step <- 1 #minutes;
    geometry shape <- envelope(square(500 #m));
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
    //初始化
    init {
    create people number: nb_people {
        speed <- agent_speed;
    }
    }
}
```

### 族群设置

现在，我们来编写人群族的属性与行为，人群中有被感染者和未被感染者两种，应有**是否感染的属性**，且感染者与被感染者**显示为不同颜色**，其行为为**移动**以及**距感染者一定距离时有一定概率被感染**。

```text
//人群族定义
species people skills: [moving] {
    //是否感染
    bool is_infected <- false;
    //移动行为
    reflex move {
    //移动行为为无目的漫游
    do wander;
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
```

然后，我们在初始化时，将一部分人设置为感染状态。

```text
global {
    ...
    init {
    //在人群族中随机寻找nb_infected_init人
    ask nb_infected_init among people {
        //将其感染状态设置为真
        is_infected <- true;
    } 
    }
}
```

### 实验设置

最后，在实验设置中编写好需在图形界面调整的参数以及代理族的显示。

```text
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
        species people;       
    }
    }
}
```

![6.2.1 &#x7B80;&#x5355;&#x7684;&#x4F20;&#x64AD;&#x6A21;&#x578B;](../../.gitbook/assets/image%20%2830%29.png)

本节完整代码如下：

```text
model SI_city

global {
    //模拟设置
    float step <- 1 #minutes;
    geometry shape <- envelope(square(500 #m));
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
    //初始化
    init {
    create people number: nb_people {
        speed <- agent_speed;
    }
    //在人群族中随机寻找nb_infected_init人
    ask nb_infected_init among people {
        //将其感染状态设置为真
        is_infected <- true;
    }
    }
}

//人群族定义
species people skills: [moving] {
    //是否感染
    bool is_infected <- false;
    //移动行为
    reflex move {
    //移动行为为无目的漫游
    do wander;
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
        species people;       
    }
    }
}
```



