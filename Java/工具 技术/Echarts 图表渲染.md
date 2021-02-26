# Echarts 图表渲染

常用的图标渲染技术由Echarts、HighChart、JFreeChart等，其中Echarts与HighChart是浏览器端渲染技术，而JFreeChart是服务器端渲染。

ECharts官网：[Apache ECharts (incubating)](https://echarts.apache.org/zh/index.html)

## Quick Start

### 获取ECharts

你可以通过以下几种方式获取 Apache ECharts (incubating)TM。

-   从 [Apache ECharts (incubating) 官网下载界面](https://echarts.apache.org/zh/download.html) 获取官方源码包后构建。

-   在 ECharts 的 [GitHub](https://github.com/apache/incubator-echarts/releases) 获取。

-   引入依赖

    ```html
    <!-- 引入 ECharts 文件 -->
    <script src="echarts.min.js"></script>
    ```

### 绘制一个简单的图表

步骤如下：

1.  准备一个放置图表的容器（div）

    ```html
    <!-- 为 ECharts 准备一个具备大小（宽高）的 DOM -->
    <div id="main" style="width: 600px;height:400px;"></div>
    ```

2.  通过 `echarts.init(<DOM元素>)` 来初始化（获取）一个echarts的实例对象

    ```js
    // 获取刚刚创建的容器Dom 
    var myChart = echarts.init(document.getElementById('main'));
    ```

3.  配置图标的配置项与数据

    ```js
    // 指定图表的配置项和数据
    var option = {
        title: {
            text: 'ECharts 入门示例'
        },
        tooltip: {},
        legend: {
            data:['销量']
        },
        xAxis: {
            data: ["衬衫","羊毛衫","雪纺衫","裤子","高跟鞋","袜子"]
        },
        yAxis: {},
        series: [{
            name: '销量',
            type: 'bar',
            data: [5, 20, 36, 10, 10, 20]
        }]
    };
    ```

    其中：

    -   xAxis中的data为 x轴的数据
    -   serise中的data为 图表中柱状图的数据

4.  通过 echarts 实例的 `setOption(<JSON对象>)` 的方式来生成柱状图

    ```js
    // 使用刚指定的配置项和数据显示图表。
    myChart.setOption(option);
    ```

编写完上述步骤之后，直接在浏览器中显示即可。



## 配置项手册

可以参照官网的配置项手册：[Documentation - Apache ECharts(incubating)](https://echarts.apache.org/zh/option.html#title)	