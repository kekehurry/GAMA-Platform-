# Step7 模型参数的优化\(Optional\)

在之前的模型中我们定义了许多可以手动调整的参数，如食草动物最大进食量（prey\_max\_transfert）、食草动物繁殖能量（prey\_energy\_reproduce）等，如果我们选择模拟一段时间后食草动物和食肉动物的数量作为观测目标，我们可以通过调整不同参数获得其对应的食草动物与食肉动物的数量，经过对比来选择能使最终数量较大的参数。

那么有没有一种方法能够自动帮我们找到最大化食草与食肉动物数量的参数呢，答案当然是肯定的，下面我们来实现模型参数的优化。

### batch experiment

在之前的实验设置中，我们一直使用的**gui** 类型的实验，**gui**可以实现动态可视化，但是在模型参数优化过程中，因为要进行大量实验，使用可视化界面会降低计算速度，因此这里我们使用 **batch**类型的实验，**batch**没有可视化界面，并且允许多线程运行，大大加快了模拟速度。

```text
//新建一个名为Optimization的实验，类型为batch,同一组参数运行2次，每次运行的随机种子相同，运行200次停止
experiment Optimization type: batch repeat: 2 keep_seed: true until: ( time > 200 ) {
}
```

> **repeat**:  同一组参数重复运行次数，仿真模拟运行有一定的随机性，因此多次运行可以让实验结果更可靠  
> **keep\_seed**: 是否使用相同的随机种子，GAMA的随机数是通过随机种子生成的，使用相同的随机种子可以生成相同的随机数。  
> **until  :**   定义模型停止运行的条件。

接下来添加要优化的参数，注意每个参数都要定义最大、最小值以及更次更新的步长。

```text
//新建一个名为Optimization的实验，类型为batch,同一组参数运行2次，每次运行的随机种子相同，运行200次停止
experiment Optimization type: batch repeat: 2 keep_seed: true until: ( time > 200 ) {
    //添加需要优化的参数
    parameter "Prey max transfert:" var: prey_max_transfert min: 0.05 max: 0.5 step: 0.05;
    parameter "Prey energy reproduce:" var: prey_energy_reproduce min: 0.05 max: 0.75 step: 0.05;
    parameter "Predator energy transfert:" var: predator_energy_transfert min: 0.1 max: 1.0 step: 0.1;
    parameter "Predator energy reproduce:" var: predator_energy_reproduce min: 0.1 max: 1.0 step: 0.1;
    parameter "Batch mode:" var: is_batch <- true;
}
```

然后选择优化方式，gama内置了几种参数优化方式，如 [`exhaustive`](https://gama-platform.github.io/wiki/ExplorationMethods#exhaustive-exploration-of-the-parameter-space-exhaustive) [`hill_climbing`](https://gama-platform.github.io/wiki/ExplorationMethods#hill-climbing-hill-climbing) [`annealing`](https://gama-platform.github.io/wiki/ExplorationMethods#simulated-annealing-annealing) [`tabu`](https://gama-platform.github.io/wiki/ExplorationMethods#tabu-search-tabu) [`reactive_tabu`](https://gama-platform.github.io/wiki/ExplorationMethods#reactive-tabu-search-reactive-tabu) [`genetic`](https://gama-platform.github.io/wiki/ExplorationMethods#genetic-algorithm-genetic) 等，默认具体优化方式为[`exhaustive`](https://gama-platform.github.io/wiki/ExplorationMethods#exhaustive-exploration-of-the-parameter-space-exhaustive)及其解释，可以查看官方文档。

```text
method tabu maximize: nb_preys + nb_predators iter_max: 10 tabu_list_size: 3;
```

