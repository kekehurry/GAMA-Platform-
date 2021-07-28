# GAMA模型结构

### GAMA模型的文件结构

GAMA模型的文件结构主要由工作空间（Workspace）、项目（Project）、模型\(Model\)组成。

* **工作空间是（Workspace）**所有GAMA模型存储的文件夹，首次启动GAMA时，便会要求设置工作空间路径。
* **项目（Project）**是一个包括原始数据和模型文件的文件夹，一个项目可以包含多个模型。在GAMA的导航栏&gt;User models&gt;右键New&gt;Gama Project，可以新建一个项目文件夹，新建的项目文件夹一般包含两个文件夹includes（用来存放关联数据的文件夹）和models（用来存放模型的文件夹）。
* **模型（Model\)** 在models文件夹右键New&gt;Model file即可新建一个模型文件，进行仿真模拟程序的编写。

![3.1 GAMA&#x6A21;&#x578B;&#x7684;&#x6587;&#x4EF6;&#x7ED3;&#x6784; ](../.gitbook/assets/image%20%285%29.png)

### 导入现有模型

GAMA提供了详尽的教程和测试模型，在导航栏中Library&gt;Tutorials文件夹里有官方教程的所有模型文件，直接双击打开即可。

![3.2 GAMA&#x5B98;&#x65B9;&#x6559;&#x7A0B;&#x6A21;&#x578B;](../.gitbook/assets/image%20%286%29.png)

如需导入自己已编写好的模型，在User models 右键&gt;Import&gt;Gama Project 选择模型文件的项目文件夹路径即可。

### GAML语言模型结构

在GAML中，一个完整的模型定义，包含三个部分：**全局代理（global）、族群或网格（species and grid\)、实验设置（experiment\)。**除此之外，在模型最开始，还需要一个**模型头（Model Header）**定义模型名称。

* **全局代理\(global\)**：全局设置实际上是一个特殊的代理，在其中定义的所有变量、行为等都对模型全局有效，同时，在全局代理中可以定义模型初始运行时的状态。
* **族群或网格（species and grid\)：**族群（species）是同一类代理的总称，在族群（species）中定义了此类代理的共同属性。一个族群（species）的定义一般包含三个部分：
  * 族群中代理内部属性
  * 族群中代理的行为
  * 族群中代理的显示方式
* **实验设置（experiment\)：** 实现设置一般包含两部分内容:输入（input\)和输出（output\)。
  * 输入（input\)：输入部分定义了一些模型运行的参数，用户可通过图形界面修改这些参数。
  * 输出（ouput\)：输出部分定义了模型运行时的输出方式，包括图形输出、三维输出、数据文件输出等等，具体设置方式详见之后的案例。



