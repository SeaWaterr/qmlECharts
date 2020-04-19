#使用 QML WebChannel 实现和 HTML ECharts 页面之间交互

建议使用较新版本的QT如`QT5.13.0`。

## WebEngineView：
WebEngineView可以直接通过WebChannel，它有一个JavaScript库叫 Qt WebChannel JavaScript API。
在这里，我只是应用WebEngineView进行加载Echarts的HTML文件进行展示效果。
1. 下载[Echarts](https://www.echartsjs.com/zh/download.html)。
2. 导入WebEngineView模块，在.pro文件中写入
```
QT += qml quick webview webengine
```
3. 创建QML
WebEngineView可以直接使用WebChanne,用来和HTML通信,
注意在`QtObject`我们定义了一个`someSignal`信号.
```
Window {
    visible: true
    width: 800
    minimumWidth: 600
    height: 500
    minimumHeight: 400
    title: qsTr("WebChannel example")  //countDown.start();
    property int y_: 6
    property int x_: 1300
    ColumnLayout {
        anchors.fill: parent
        anchors.margins: 5
        // 一个具有属性、信号和方法的对象——就像任何普通的Qt对象一样
        QtObject {
            id: someObject
            // ID，在这个ID下，这个对象在WebEngineView端是已知的
            WebChannel.id: "backend"
            signal someSignal(int y,int x);
        }
        Timer{
            id:countDown;
            interval: 1000;
            repeat: true;
            triggeredOnStart: true;
            onTriggered: {
                someObject.someSignal(text)
            }
        }
        Rectangle {
            Layout.fillWidth: true
            Layout.fillHeight: true
            border.width: 2
            border.color: "blue"
            WebEngineView {
                id: webView
                anchors.fill: parent
                anchors.margins: 5
                url: "qrc:/html/index.html"
                onLoadingChanged: {
                    if (loadRequest.errorString)
                    { console.error(loadRequest.errorString); }
                }
                webChannel: channel
            }
            WebChannel {
                id: channel
                registeredObjects: [someObject]
            }
        }
        Button {
            id: button
            text: qsTr("添加数据")
            onClicked: {
                y_ += 1
                x_ += 100
                someObject.someSignal(y_,x_)
            }
        }
    }
}
```
4. 创建HTML
在这里利用`QWebChannel`将QML中绑定ID的对象拿到,
绑定信号和槽进行通信,同时也可以在HTML修改QML中`QtObject`的属性和调用函数.
```
![1450986-20190108142847874-413014297](C:\Users\dngfe\Pictures\1450986-20190108142847874-413014297.png)    // 这是QML端的QtObject
    var backend;
    window.onload = function()
    {
        new QWebChannel(qt.webChannelTransport, function(channel) {
        // 在channel.object下，所有发布的对象在通道中都是可用的
        // 在附加的WebChannel.id属性中设置的标识符。
            backend = channel.objects.backend;
            //连接信号
            backend.someSignal.connect(function(y,x) {
                       data.push([y,x]);
                   myChart.setOption({
                       series: [{
                           data: data
                       }]
                   });
            });
        });
    }
    var data = [[0,820], [1,932], [2,901], [3,934], [4,1290], [5,1330], [6,1320]]
    // 基于准备好的dom，初始化echarts实例
    var myChart = echarts.init(document.getElementById('main'));
    option = {
        xAxis: {
        },
        yAxis: {
        },
        series: [{
            data: data,
            type: 'line'
        }]
    };
    // 使用刚指定的配置项和数据显示图表。
    myChart.setOption(option)
```
- 这是它的模式：  

  ![](C:\Users\dngfe\Pictures\1450986-20190108142847874-413014297.png)

- 运行效果

  ![](C:\Users\dngfe\Pictures\Snipaste_2020-04-18_23-13-46.png)

  ![](C:\Users\dngfe\Pictures\Snipaste_2020-04-18_23-13-53.png)

------

   

