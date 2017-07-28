```
/**
 * Created by whobird on 17/7/27.
 */
Number.prototype.formatMoney = function (c, d, t) {
    var n = this,
        c = isNaN(c = Math.abs(c)) ? 2 : c,
        d = d == undefined ? "." : d,
        t = t == undefined ? "," : t,
        s = n < 0 ? "-" : "",
        i = String(parseInt(n = Math.abs(Number(n) || 0).toFixed(c))),
        j = (j = i.length) > 3 ? j % 3 : 0;
    return s + (j ? i.substr(0, j) + t : "") + i.substr(j).replace(/(\d{3})(?=\d)/g, "$1" + t) + (c ? d + Math.abs(n - i).toFixed(c).slice(2) : "");
};
var chartRender = (function ($, cr) {
    var chartRender = cr;
    chartRender.data_init = {
        lineData: {
            labels: [],
            datasets: [
                {
                    type: "bar",
                    name: "WIFI客流",
                    xAxisIndex: 0,
                    yAxisIndex: 0,
                    symbol: 'emptyCircle',
                    symbolSize: 6,
                    showSymbol: true,
                    barWidth:"20px",
                    label:{
                        normal:{
                            show:true,
                            position:"top",
                            textStyle:{
                                color:"#999"
                            }
                        }
                    },
                    lineStyle: {
                        show: true,
                        color: "#3aabf3",
                        width: 2,
                        type: "solid",
                    },
                    data: [],
                }, {
                    type: "bar",
                    name: "客流计数器",
                    xAxisIndex: 0,
                    yAxisIndex: 0,
                    symbol: 'emptyCircle',
                    symbolSize: 6,
                    showSymbol: true,
                    barWidth:"20px",
                    label:{
                        normal:{
                            show:true,
                            position:"top",
                            textStyle:{
                                color:"#999"
                            }
                        }
                    },
                    lineStyle: {
                        show: true,
                        color: "#70d3f7",
                        width: 2,
                        type: "solid",
                    },
                    data: [],
                }, {
                    type: "bar",
                    name: "验票数",
                    xAxisIndex: 0,
                    yAxisIndex: 0,
                    symbol: 'emptyCircle',
                    symbolSize: 6,
                    showSymbol: true,
                    barWidth:"20px",
                    label:{
                        normal:{
                            show:true,
                            position:"top",
                            textStyle:{
                                color:"#999"
                            }
                        }
                    },
                    lineStyle: {
                        show: true,
                        color: "#66b56a",
                        width: 2,
                        type: "solid",
                    },
                    data: [],
                }
            ]
        },
        pieData: {
            datasets: [
                {
                    name: '',
                    type: 'pie',
                    radius : ['60%','86%'],
                    center: ['50%', '50%'],
                    selectedOffset:0,
                    data:[
                    ],
                    label:{
                        normal:{show:false}
                    },
                    itemStyle: {
                        emphasis: {
                            shadowBlur: 10,
                            shadowOffsetX: 0,
                            shadowColor: 'rgba(0, 0, 0, 0.5)'
                        }
                    }
                },
            ]
        },//piedata
    };
    chartRender.opt = {
        lineChartOpt: {
            title: {
                show: false,
            },
            legend: {
                show: false,
            },
            toolbox: {
                show: false,
            },
            grid: {
                top: 30,
                left: 80,
                right: 30,
                bottom: 50
            },
            xAxis: {
                position: "bottom",
                type: "category",
                /* name:"年",
                 nameLocation:"middle",
                 nameTextStyle:{
                 color:"#acadb0",
                 fontStyle:"normal",
                 fontSize:14
                 },
                 nameGap:25,*/
                boundaryGap: true,
                axisLine: {
                    show: true,
                    lineStyle: {
                        color: "#ececec",
                        width: 1,
                        type: "solid"
                    }
                },
                axisTick: {
                    show: false,
                    inside: true,
                    length: 3,
                    lineStyle: {
                        color: "#666",
                        width: 1,
                        type: "solid"
                    }
                },
                axisLabel: {
                    show: true,
                    //formatter:null,
                    formatter: '{value}月',
                    margin: 12,
                    textStyle: {
                        color: "#666",
                        fontStyle: "normal",
                        fontSize: 12
                    }
                },
                splitLine: {
                    show: false,
                    lineStyle: {
                        color: "#ececec",
                        width: 1,
                        type: "solid"
                    }
                },
                data: [],//labels
            },
            yAxis: {
                /*name:"万元",
                 nameLocation:"end",
                 nameGap:15,
                 nameTextStyle:{
                 color:"#acadb0",
                 fontStyle:"normal",
                 fontSize:14
                 },*/
                min: 0,
                //max:"auto",
                //splitNumber:7,
                axisLine: {
                    show: false,
                    lineStyle: {
                        color: "#535861",
                        width: 1,
                        type: "solid"
                    }
                },
                axisTick: {
                    show: false,
                    inside: false,
                    length: 6,
                    linStyle: {
                        color: "#535861",
                        width: 1,
                        type: "solid"
                    }
                },
                axisLabel: {
                    show: true,
                    formatter: '{value}',
                    margin: 15,
                    textStyle: {
                        color: "#878787",
                        fontStyle: "normal",
                        fontSize: 12
                    }
                },
                splitLine: {
                    show: true,
                    lineStyle: {
                        color: "#ececec",
                        width: 1,
                        type: "solid"
                    }
                },

            },
            color: ['#3aabf3', '#70d3f7', '#66b56a', '#feb739', '#91c7ae', '#749f83', '#ca8622', '#bda29a', '#6e7074', '#546570', '#c4ccd3'],
            backgroundColor: "transparent",
            tooltip: {
                show: true,
                showContent: true,
                //formatter:"{a}:<br/>{b}年:{c}万",
                formatter: function (params, ticket, callback) {

                    //if(parmase.seriesName=""){}
                    var value = parseFloat(params.data).formatMoney(0, ".", ",");
                    return params.seriesName + "<br/>" + params.name + "月 : " + value;
                },
                textStyle: {
                    fontSize: 12,
                    color: "#fff"
                }
            },
            series: [
                //dataset
            ]
        },
        pieChartOpt: {
            title : {
                text: '项目Dount',
                show:false,
            },
            tooltip : {
                show:true,
                trigger: 'item',
                formatter: "{a} <br/>{b} : {c}% "
            },
            legend: {
                show:false
            },
            hoverAnimation:false,
            color:['#018def','#01b0f1', '#2cd9dd', '#a6ec67', '#ddf7a0','#adbaca',  '#45546b', '#bda29a','#6e7074', '#546570', '#c4ccd3'],
            series : [
                {
                    name: '',
                    type: 'pie',
                    radius : ['60%','86%'],
                    center: ['62%', '50%'],
                    selectedOffset:0,
                    /*
                    * data:[{
                    *   name:'',
                    *   value:,
                    * }]
                    * */

                    data:[],
                    label:{
                        normal:{show:false}
                    },
                    itemStyle: {
                        emphasis: {
                            shadowBlur: 10,
                            shadowOffsetX: 0,
                            shadowColor: 'rgba(0, 0, 0, 0.5)'
                        }
                    }
                }
            ]
        }
    };

    chartRender.setOption = function (chartType,data,labels) {
            if(chartType=="pie"){

                chartRender.data_init[chartType+"Data"].datasets[0].name=data.name;
                chartRender.data_init[chartType+"Data"].datasets[0].data=data.values;
                chartRender.opt[chartType+"ChartOpt"].series[0]=chartRender.data_init[chartType+"Data"].datasets[0];
            }

            if(chartType=="line"){
                $.each(chartRender.data_init[chartType+"Data"].datasets,function(i,dataset){
                    dataset.data=data[i].values;
                });

                chartRender.data_init[chartType+"Data"].labels=labels;
                chartRender.opt[chartType+"ChartOpt"].xAxis.data=chartRender.data_init[chartType+"Data"].labels;
                chartRender.opt[chartType+"ChartOpt"].series=chartRender.data_init[chartType+"Data"].datasets;
            }
            return chartRender.opt[chartType+"ChartOpt"];
    };

    chartRender.update=function(chart,data,chartType){
        var labels=data["labels"];
        var data=data["chart"];
        var opt=chartRender.setOption(chartType,data,labels);
        chart.setOption(opt);
    };

    //init
    chartRender.init = function (id,chartType) {

        //todo:chartType:pie,line
        var $elem=$(id);
        var chart=echarts.init($elem.get(0));
        var labels=$elem.data("labels");
        var data=$elem.data("chart");

        var opt=chartRender.setOption(chartType,data,labels);
        chart.setOption(opt);
        return chart;
    };

    return chartRender;
})(jQuery, chartRender || {})
```

```
<div class="panel col-xs-6">
                    <div class="panel-header">
                        <h5>销售额(万元)</h5>
                    </div>
                    <div class="panel-chart">
                        <div class="col-xs-6 chart-wrapper" id="pie-chart-sales-wrapper">
                            <em class="pie-chart-info">1759</em>
                            <div id="pie-chart-sales" style="height:280px;" data-labels=[]
                                 data-chart={"name":"销售额","values":[{"name":"A","value":30},{"name":"B","value":99}]}>
                            </div>
                        </div><!--noi-chart-wrapper-->

                        <div class="chart-legend-wrapper col-xs-6">
                            <div class="chart-legend chart-legend-vertical">
                                <span><em class="color-label" style="background-color:#018def;"></em><i class="name">OTA</i><i class="value">154</i></span>
                                <span><em class="color-label" style="background-color:#01b0f1;"></em><i class="name">TA</i><i class="value">154</i></span>
                                <span><em class="color-label" style="background-color:#2cd9dd;"></em><i class="name">门票直销</i><i class="value">234</i></span>

                                <span><em class="color-label" style="background-color:#a6ec67;"></em><i class="name">其他直销</i><i class="value">154</i></span>
                                <span><em class="color-label" style="background-color:#ddf7a0;"></em><i class="name">零售</i><i class="value">154</i></span>

                                <span><i class="name">预算</i><i class="value">1200(58%)</i></span>
                            </div>
                        </div>
                    </div>
                </div><!--panel col-xs-6-->
```

```
 <div class="panel col-xs-12">
                    <div class="panel-header">
                        <h5>WIFI客流/客流计数器/验票数</h5>
                    </div>
                    <div class="panel-chart ">
                        <div class="col-xs-12 chart-wrapper" id="line-chart-flow-wrapper" >
                            <!-- <em class="chart-axis-info chart-axis-y-info">万元</em>-->
                            <div id="line-chart-flow" style="height:280px;" data-labels=["1","2","3","4","5","6","7"]
                                 data-chart=[{"name":"charttest","values":[122,144,154,113,222,165,143]},{"name":"charttest","values":[132,154,134,133,122,185,173]},{"name":"charttest","values":[112,134,124,123,112,155,133]}]>
                            </div>
                        </div><!--noi-chart-wrapper-->

                        <div class="chart-legend-wrapper col-xs-12">
                            <div class="chart-legend">
                                <span><em class="color-label" style="background-color:#3aabf3;"></em><i class="name">WIFI客流(人)</i><i class="value">845672</i></span>
                                <span><em class="color-label" style="background-color:#70d3f7;"></em><i class="name">客流计数器(人)</i><i class="value">845672</i></span>
                                <span><em class="color-label" style="background-color:#66b56a;"></em><i class="name">验票数(人)</i><i class="value">845672</i></span>
                            </div>
                        </div>
                    </div>
                </div><!--col-xs-12-->
```

```
/*chart-legend*/
.chart-legend-wrapper{position: relative;}
.chart-wrapper .chart-legend {
    text-align: right;
    padding: 0 35px;
}
.chart-legend>span {
    display: inline-block;
    height: 30px;
    line-height: 30px;
    margin-right: 15px;
    margin-left: 15px;
    font-size: 14px;
    color: #333;}

.chart-legend em.color-label {
    display: inline-block;
    position: relative;
    top: 1px;
    margin-right: 6px;
    width: 12px;
    height: 12px;
    background-color: #e5e5e5;
}
.chart-legend>span i.value{float: right;margin-left:25px;}


.chart-legend-vertical{position: absolute;width:220px;left:15%;top:10px;}
.chart-legend-vertical span{display:block;height:32px;line-height: 32px;}


#line-chart-flow-wrapper+.chart-legend-wrapper .chart-legend{ margin: 0 auto; width: 620px; }

```



