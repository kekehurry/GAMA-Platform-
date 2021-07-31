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
      
    ...
}
```

> **skills：** skills是GAMA为仿真模型预设的行为模块，每个skill都包含了相应的属性和行为动作，以**moving**为例，moving包含了speed、heading、destination等属性以及move、goto、follow、wander、wander\_3D等内置行为函数，使用者可以更加方便快捷地完成复杂行为设计，而无需从零编写。更多skills相关，可查看官方文档-[Attaching Skills](https://gama-platform.github.io/wiki/AttachingSkills)。

