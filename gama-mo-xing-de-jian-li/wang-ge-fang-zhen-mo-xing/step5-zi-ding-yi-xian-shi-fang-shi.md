# Step4: 自定义显示方式

在GAML中代理族的显示方式，由族定义中的aspect关键字控制，在之前的定义中食草动物（prey\)和食肉动物\(predator\)都显示为圆形。

```text
	//显示方式：大小为size的圆，颜色为color
	aspect base {
		draw circle(size) color: color;
	}
```

而除了显示为几何图形（常见的有**circle**，**square**，**triangle**等）之外，GAML还能实现其他几种方式：

* **icon：**通过**image\_file** 将代理显示为自定义图标。
* **info：** 通过文字信息直接显示数据信息。
* 更多使用方式，可在官方文档搜索 **”draw"**查询

