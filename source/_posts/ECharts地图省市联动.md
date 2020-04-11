title: ECharts地图省市联动
tags:
  - ECharts
categories:
  - Dev
  - Front
date: 2020-03-05 20:22:00
cover: true

---

![](https://cdn.jsdelivr.net/gh/coder-lida/CDN/img/chart.png)
<!-- more -->
>最近需要做一个省市联动的地图，来随时观看各地区的用户数量。

记录实现代码。

主页面`china.html`：
```
<!DOCTYPE html>
<html>

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=EDGE">
    <style>
        #china-map {
            width: 1000px;
            height: 1000px;
            margin: auto;
        }

        #box {
            display: none;
            background-color: goldenrod;
            width: 180px;
            height: 30px;
        }

        #box-title {
            display: block;
        }
    </style>
    <script src="http://www.jq22.com/jquery/jquery-1.10.2.js"></script>
    <script type="text/javascript" src="https://cdn.bootcss.com/echarts/4.2.0-rc.2/echarts.min.js"></script>
    <script type="text/javascript" src="./js/map/china.js"></script>
</head>

<body>
<div class="wrapper wrapper-content">
    <div class="row">
        <button id="back">返回全国</button>

        <div id="china-map"></div>
    </div>
</div>


<script>
    var myChart = echarts.init(document.getElementById('china-map'));
    var oBack = document.getElementById("back");

    var provinces = ['shanghai', 'hebei', 'shanxi', 'neimenggu', 'liaoning', 'jilin', 'heilongjiang', 'jiangsu', 'zhejiang', 'anhui', 'fujian', 'jiangxi', 'shandong', 'henan', 'hubei', 'hunan', 'guangdong', 'guangxi', 'hainan', 'sichuan', 'guizhou', 'yunnan', 'xizang', 'shanxi1', 'gansu', 'qinghai', 'ningxia', 'xinjiang', 'beijing', 'tianjin', 'chongqing', 'xianggang', 'aomen'];

    var provincesText = ['上海', '河北', '山西', '内蒙古', '辽宁', '吉林', '黑龙江', '江苏', '浙江', '安徽', '福建', '江西', '山东', '河南', '湖北', '湖南', '广东', '广西', '海南', '四川', '贵州', '云南', '西藏', '陕西', '甘肃', '青海', '宁夏', '新疆', '北京', '天津', '重庆', '香港', '澳门'];

    oBack.onclick = function () {
        initEcharts("china", "点亮中国");
    };
    initEcharts("china", "点亮中国");

    // 初始化echarts
    function initEcharts(pName, Chinese_) {
        $.ajax({
            type: "GET",
            url: "http://localhost:4000/user/userCount?searchName="+pName,
            success: function (res) {
                // var seriesData = res.data;
                // var tmpSeriesData = pName === "china" ? seriesData : [];
                var tmpSeriesData = res.data;
                var option = {
                    title: {
                        text: Chinese_ || pName,
                        left: 'center'
                    },
                    toolbox: {
                        show: !0,
                        orient: "vertical",
                        x: "right",
                        y: "center",
                        feature: {
                            mark: {
                                show: !0
                            },
                            dataView: {
                                show: !0,
                                readOnly: !1
                            },
                            restore: {
                                show: !0
                            },
                            saveAsImage: {
                                show: !0
                            }
                        }
                    },
                    tooltip: {
                        trigger: 'item',
                        formatter: '{b}<br/>{c} (人)',
                        backgroundColor: "#ff7f50",//提示标签背景颜色
                        textStyle: {color: "#fff"} //提示标签字体颜色
                    },
                    series: [
                        {
                            name: Chinese_ || pName,
                            type: 'map',
                            mapType: pName,
                            roam: false,//是否开启鼠标缩放和平移漫游
                            data: tmpSeriesData,
                            top: "3%",//组件距离容器的距离
                            zoom: 1.1,
                            selectedMode: 'single',

                            label: {
                                normal: {
                                    show: true,//显示省份标签
                                    textStyle: {color: "#fbfdfe"}//省份标签字体颜色
                                },
                                emphasis: {//对应的鼠标悬浮效果
                                    show: true,
                                    textStyle: {color: "#323232"}
                                }
                            },
                            itemStyle: {
                                normal: {
                                    borderWidth: .5,//区域边框宽度
                                    borderColor: '#0550c3',//区域边框颜色
                                    areaColor: "#4ea397",//区域颜色

                                },
                                emphasis: {
                                    borderWidth: .5,
                                    borderColor: '#4b0082',
                                    areaColor: "#ece39e",
                                }
                            },
                        }
                    ]

                };

                myChart.setOption(option);

                myChart.off("click");

                if (pName === "china") { // 全国时，添加click 进入省级
                    myChart.on('click', function (param) {
                        console.log(param.name);
                        // 遍历取到provincesText 中的下标  去拿到对应的省js
                        for (var i = 0; i < provincesText.length; i++) {
                            if (param.name === provincesText[i]) {
                                //显示对应省份的方法
                                showProvince(provinces[i], provincesText[i]);
                                break;
                            }
                        }
                        if (param.componentType === 'series') {
                            var provinceName = param.name;
                            $('#box').css('display', 'block');
                            $("#box-title").html(provinceName);

                        }
                    });
                } else { // 省份，添加双击 回退到全国
                    myChart.on("dblclick", function () {
                        initEcharts("china", "点亮中国");
                    });
                }
            }
        });

    }

    // 展示对应的省
    function showProvince(pName, Chinese_) {
        //这写省份的js都是通过在线构建工具生成的，保存在本地，需要时加载使用即可，最好不要一开始全部直接引入。
        loadBdScript('$' + pName + 'JS', './js/map/province/' + pName + '.js', function () {
            initEcharts(Chinese_);
        });
    }

    // 加载对应的JS
    function loadBdScript(scriptId, url, callback) {
        var script = document.createElement("script");
        script.type = "text/javascript";
        if (script.readyState) {  //IE
            script.onreadystatechange = function () {
                if (script.readyState === "loaded" || script.readyState === "complete") {
                    script.onreadystatechange = null;
                    callback();
                }
            };
        } else {  // Others
            script.onload = function () {
                callback();
            };
        }
        script.src = url;
        script.id = scriptId;
        document.getElementsByTagName("head")[0].appendChild(script);
    };
</script>
</body>

</html>

```

效果图：
![](https://cdn.jsdelivr.net/gh/coder-lida/CDN/img/assert/echart-1.png)

![](https://cdn.jsdelivr.net/gh/coder-lida/CDN/img/assert/echart-2.png)

源码地址：https://github.com/coder-lida/chinamap.git


