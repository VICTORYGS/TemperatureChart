<template>
    <div class="andyContain" style="display: flex;flex-flow: column;align-items: center;">
        <div id="top" style="width:100%;height:500px;background: #f6f2f6"></div>
        <div class="b">
            <!--<div id="hot" style="background: #f6f2f6"></div>-->
            <div id="cold" style="background: #f6f2f6"></div>
            <div id="wind" style="background: #f6f2f6"></div>
            <div id="Max_humidity" style="background: #f6f2f6"></div>
            <div id="Min_humidity" style="background: #f6f2f6"></div>
        </div>

        <!--{{province}} {{city}} {{district}}
        <addressInput></addressInput>-->

    </div>
</template>
<script>
    import addressData from '@/address'
    var jsonp = require(`jsonp`)
    var echarts = require('echarts');
    import addressInput from '@/components/address'
    import $ from "jquery/dist/jquery.js"

    export default {
        name: 'app',
        created() {
            this.ok=1
            let all=[]
            for(let i in addressData){
                let arr=[i]
                for(let ii in addressData[i]){
                    let arr1=[].concat(arr)
                    if(ii!==`市辖区`&&ii!==`县`){
                        arr1.push(ii)
                    }
                    if(ii!==`省直辖县级行政区划`&&ii!==`自治区直辖县级行政区划`) all.push(arr1)

                    /*for(let iii in addressData[i][ii]){
                        let newArr=[].concat(arr1)
                        newArr.push(addressData[i][ii][iii])
                        all.push(newArr)
                    }*/
                }
            }

            let singleArr=all.filter(v=>v.length===1)
            let notSingleArr=all.filter(v=>v.length>1)
            let downArr=Array.prototype.concat.apply([],singleArr)
            let removeArr=[...new Set(downArr)]
            all=notSingleArr
            removeArr.forEach(v=>{
                all.unshift([v])
            })
            // console.log('所有要请求的地址:',all)
            //处理地理位置数据,并发送请求
            let usedArr=[]
            all.forEach((v,index)=>{
                let arr=v.map(v=>{
                    let res=v
                    if(res.slice(-2,)==='地区'){
                        res=res.slice(0,-2)
                    }
                    if(res.slice(-3,)==='自治州'){
                        res=res.slice(0,-3)
                    }
                    /*let z=[`回族`,`傈僳族`,`白族`,`傣族`,`壮族`,`哈尼族`,`土家族`,`苗族`,`苗族`,`藏族`,`羌族`,`彝族`,`布依族`,`侗族`]

                    z.forEach(v=>{
                        res=res.replace(v,'')

                    })*/
                    let a= ['延边','恩施','湘西','黔南','黔西南','红河','德宏','怒江','海西','博尔塔拉','巴音郭楞','伊犁']
                    a.forEach(v=>{
                        if(res.slice(0,v.length)==v){
                            res=v
                        }
                    })
                    if(res.slice(-1,)==='族'){
                        res=res.slice(0,-2)
                    }
                    if(res.slice(-1,)==='族'){
                        res=res.slice(0,-2)
                    }
                    return res
                })
                if(arr.length===1){//直辖市
                    arr.push(arr[0],'')
                    // arr.unshift('')
                }else {
                    arr.push(...new Array(3-v.length).fill(''))
                }

                let [province,city,county]=arr//['','','']
                if(city.includes('克孜勒苏柯尔'))return;
                let param=`source=pc&weather_type=observe%7Cforecast_1h%7Cforecast_24h%7Cindex%7Calarm%7Climit%7Ctips%7Crise&province=${province}&city=${city}&county=${county}&callback=callback`
                jsonp(`https://wis.qq.com/weather/common`, {param}, (err, data)=>{
                    // console.log(index, err, data)
                    let o=data.data.observe
                    o.address=arr.join(" ")
                    usedArr.push(o)
                    /*if(!Object.keys(data.data.forecast_1h).length){
                        console.log(arr)//筛选出没有请求到数据的地址信息
                    }*/
                    if(index+1==all.length){//完成所有的请求之后
                        // console.log('usedArr:',usedArr)
                        this.hotArr=order(usedArr,'degree')
                        this.humidityArr= order(usedArr,'humidity')
                        this.windArr= order(usedArr,'wind_power')
                        // console.log('==>',this.hotArr,this.humidityArr,this.windArr)
                        this.renderChart()
                    }
                })

            })

            function order(a,p) {
                if(a.length==0)return a
                let pilot=a[0]
                let pilotValue=a[0][p]-0//要比较的值
                let small=[]
                let big=[]
                small=a.slice(1).filter(v=>v[p]<pilotValue)
                big=a.slice(1).filter(v=>v[p]>=pilotValue)
                // console.log(101,small,big,pilotValue)
                return [...order(small,p),pilot,...order(big,p)]
            }


        },
        data() {
            return {
                province: '北京', city: "", district: "",
                hotArr:[],humidityArr:[],windArr:[],ok:null,
            }
        },
        components: {addressInput},
        methods:{
            async renderChart(){
                /*var hot = echarts.init(document.getElementById('hot'));
                hot.setOption({
                    title: {
                        text: '最热城市排行Top8',
                        textStyle:{
                            color:'#ffffff',
                        },
                        backgroundColor:"#f39e8f"
                    },
                    color: ['#3398DB'],
                    tooltip : {
                        trigger: 'axis',
                        axisPointer : {            // 坐标轴指示器，坐标轴触发有效
                            type : 'shadow'        // 默认为直线，可选为：'line' | 'shadow'
                        }
                    },
                    grid: {
                        left: '3%',
                        right: '4%',
                        bottom: '3%',
                        containLabel: true
                    },
                    xAxis : [
                        {
                            type : 'category',
                            data : this.hotArr.slice(-8).map(v=>v.address.trim().split(' ').reverse()[0]),
                            axisTick: {
                                alignWithLabel: true
                            },
                            axisLabel:{
                                interval:0
                            }
                        }
                    ],
                    yAxis : [
                        {
                            type : 'value'
                        }
                    ],
                    series : [
                        {
                            name:'气温',
                            type:'bar',
                            barWidth: '60%',
                            data:this.hotArr.slice(-8).map(v=>v.degree),
                            itemStyle: {
                                normal: {
                                    color: new echarts.graphic.LinearGradient(
                                        0, 0, 0, 1,
                                        [
                                            {offset: 0, color: '#f6a394'},
                                            {offset: 0.5, color: '#f06955'},
                                            {offset: 1, color: '#f02b16'}
                                        ]
                                    )
                                },
                                emphasis: {
                                    color: new echarts.graphic.LinearGradient(
                                        0, 0, 0, 1,
                                        [
                                            {offset: 0, color: '#f02b16'},
                                            {offset: 0.7, color: '#f06955'},
                                            {offset: 1, color: '#f6a394'}
                                        ]
                                    )
                                }
                            },
                        }
                    ]
                });*/

                var cold = echarts.init(document.getElementById('cold'));
                cold.setOption({
                    title: {
                        text: '最冷城市排行Top8',
                        textStyle:{
                            color:'#ffffff',
                        },
                        backgroundColor:"#81bdf5"
                    },
                    color: ['#3398DB'],
                    tooltip : {
                        trigger: 'axis',
                        axisPointer : {            // 坐标轴指示器，坐标轴触发有效
                            type : 'shadow'        // 默认为直线，可选为：'line' | 'shadow'
                        }
                    },
                    grid: {
                        left: '3%',
                        right: '4%',
                        bottom: '3%',
                        containLabel: true
                    },
                    xAxis : [
                        {
                            type : 'category',
                            data : this.hotArr.slice(0,8).map(v=>v.address.trim().split(' ').reverse()[0]),
                            axisTick: {
                                alignWithLabel: true
                            },
                            axisLabel:{
                                interval:0
                            }
                        }
                    ],
                    yAxis : [
                        {
                            type : 'value'
                        }
                    ],
                    series : [
                        {
                            name:'气温',
                            type:'bar',
                            barWidth: '60%',
                            data:this.hotArr.slice(0,8).map(v=>v.degree),
                            itemStyle: {
                                normal: {
                                    color: new echarts.graphic.LinearGradient(
                                        0, 0, 0, 1,
                                        [
                                            {offset: 0, color: '#83bff6'},
                                            {offset: 0.5, color: '#188df0'},
                                            {offset: 1, color: '#188df0'}
                                        ]
                                    )
                                },
                                emphasis: {
                                    color: new echarts.graphic.LinearGradient(
                                        0, 0, 0, 1,
                                        [
                                            {offset: 0, color: '#2378f7'},
                                            {offset: 0.7, color: '#2378f7'},
                                            {offset: 1, color: '#83bff6'}
                                        ]
                                    )
                                }
                            },
                        }
                    ]
                });

                var Max_humidity = echarts.init(document.getElementById('Max_humidity'));
                Max_humidity.setOption({
                    title: {
                        text: '最湿润城市排行Top8',
                        textStyle:{
                            color:'#ffffff',
                        },
                        backgroundColor:"#b4f3b8"
                    },
                    color: ['#3398DB'],
                    tooltip : {
                        trigger: 'axis',
                        axisPointer : {            // 坐标轴指示器，坐标轴触发有效
                            type : 'shadow'        // 默认为直线，可选为：'line' | 'shadow'
                        }
                    },
                    grid: {
                        left: '3%',
                        right: '4%',
                        bottom: '3%',
                        containLabel: true
                    },
                    xAxis : [
                        {
                            type : 'category',
                            data : this.humidityArr.slice(-8).map(v=>v.address.trim().split(' ').reverse()[0]),
                            axisTick: {
                                alignWithLabel: true
                            },
                            axisLabel:{
                                interval:0
                            }
                        }
                    ],
                    yAxis : [
                        {
                            type : 'value'
                        }
                    ],
                    series : [
                        {
                            name:'湿度',
                            type:'bar',
                            barWidth: '60%',
                            data:this.humidityArr.slice(-8).map(v=>v.humidity),
                            itemStyle: {
                                normal: {
                                    color: new echarts.graphic.LinearGradient(
                                        0, 0, 0, 1,
                                        [
                                            {offset: 0, color: '#b7f6bb'},
                                            {offset: 0.5, color: '#80d47b'},
                                            {offset: 1, color: '#2ab62e'}
                                        ]
                                    )
                                },
                                emphasis: {
                                    color: new echarts.graphic.LinearGradient(
                                        0, 0, 0, 1,
                                        [
                                            {offset: 0, color: '#2ab62e'},
                                            {offset: 0.7, color: '#80d47b'},
                                            {offset: 1, color: '#b7f6bb'}
                                        ]
                                    )
                                }
                            },
                        }
                    ]
                });

                var Min_humidity = echarts.init(document.getElementById('Min_humidity'));
                Min_humidity.setOption({
                    title: {
                        text: '最干燥城市排行Top8',
                        textStyle:{
                            color:'#ffffff',
                        },
                        backgroundColor:"#ede1b0"
                    },
                    color: ['#3398DB'],
                    tooltip : {
                        trigger: 'axis',
                        axisPointer : {            // 坐标轴指示器，坐标轴触发有效
                            type : 'shadow'        // 默认为直线，可选为：'line' | 'shadow'
                        },
                        axisLabel:{
                            interval:0
                        }
                    },
                    grid: {
                        left: '3%',
                        right: '4%',
                        bottom: '3%',
                        containLabel: true
                    },
                    xAxis : [
                        {
                            type : 'category',
                            data : this.humidityArr.slice(0,8).map(v=>v.address.trim().split(' ').reverse()[0]),
                            axisTick: {
                                alignWithLabel: true
                            },
                            axisLabel:{
                                interval:0
                            }
                        }
                    ],
                    yAxis : [
                        {
                            type : 'value'
                        }
                    ],
                    series : [
                        {
                            name:'湿度',
                            type:'bar',
                            barWidth: '60%',
                            data:this.humidityArr.slice(0,8).map(v=>v.humidity),
                            itemStyle: {
                                normal: {
                                    color: new echarts.graphic.LinearGradient(
                                        0, 0, 0, 1,
                                        [
                                            {offset: 0, color: '#efe4b2'},
                                            {offset: 0.5, color: '#efcd68'},
                                            {offset: 1, color: '#efb922'}
                                        ]
                                    )
                                },
                                emphasis: {
                                    color: new echarts.graphic.LinearGradient(
                                        0, 0, 0, 1,
                                        [
                                            {offset: 0, color: '#efb922'},
                                            {offset: 0.7, color: '#efcd68'},
                                            {offset: 1, color: '#efe4b2'}
                                        ]
                                    )
                                }
                            },
                        }
                    ]
                });

                var wind = echarts.init(document.getElementById('wind'));
                wind.setOption({
                    title: {
                        text: '最强风速城市排行Top8',
                        backgroundColor:"#b4b8fe",
                        textStyle:{
                            color:'#ffffff',
                        }
                    },
                    color: ['#3398DB'],
                    tooltip : {
                        trigger: 'axis',
                        axisPointer : {            // 坐标轴指示器，坐标轴触发有效
                            type : 'shadow'        // 默认为直线，可选为：'line' | 'shadow'
                        }
                    },
                    grid: {
                        left: '3%',
                        right: '4%',
                        bottom: '3%',
                        containLabel: true
                    },
                    xAxis : [
                        {
                            type : 'category',
                            data : this.windArr.slice(-8).map(v=>v.address.trim().split(' ').reverse()[0]),
                            axisTick: {
                                alignWithLabel: true
                            },
                            axisLabel:{
                                interval:0
                            }
                        }
                    ],
                    yAxis : [
                        {
                            type : 'value'
                        }
                    ],
                    series : [
                        {
                            name:'风速等级',
                            type:'bar',
                            barWidth: '60%',
                            data:this.windArr.slice(-8).map(v=>v.wind_power),
                            itemStyle: {
                                color: '#b5b9ff'
                            },
                        }
                    ]
                });


                var top=echarts.init(document.getElementById('top'))
                top.setOption({
                    title: {
                        text: '全国主要城市最热TOP20',
                        subtext: '',
                        textStyle:{
                            color:'#ffffff',
                        },
                        backgroundColor:"#fdac33",
                        left: 'center',
                        padding:[15,60,15,60]
                    },
                })

                var zhongguo = "https://victorygs.github.io/MapOfChina/map/data-china.json";
                var hainan = "https://victorygs.github.io/MapOfChina/map/data-hainan.json";
                var xizang = "https://victorygs.github.io/MapOfChina/map/data-xizang.json";
                var zhejiang = "https://victorygs.github.io/MapOfChina/map/data-zhejiang.json";
                var yunnan = "https://victorygs.github.io/MapOfChina/map/data-yunnan.json";
                var xinjiang = "https://victorygs.github.io/MapOfChina/map/data-xinjiang.json";
                var tianjin = "https://victorygs.github.io/MapOfChina/map/data-tianjin.json";
                var sichuan = "https://victorygs.github.io/MapOfChina/map/data-sichuan.json";
                var shanxi = "https://victorygs.github.io/MapOfChina/map/data-shanxi.json";
                var shangxi = "https://victorygs.github.io/MapOfChina/map/data-shangxi.json";
                var shanghai = "https://victorygs.github.io/MapOfChina/map/data-shanghai.json";
                var shangdong = "https://victorygs.github.io/MapOfChina/map/data-shangdong.json";
                var qinghai = "https://victorygs.github.io/MapOfChina/map/data-qinghai.json";
                var ningxia = "https://victorygs.github.io/MapOfChina/map/data-ningxia.json";
                var neimenggu = "https://victorygs.github.io/MapOfChina/map/data-neimenggu.json";
                var liaoning = "https://victorygs.github.io/MapOfChina/map/data-liaoning.json";
                var jilin = "https://victorygs.github.io/MapOfChina/map/data-jilin.json";
                var jiangxi = "https://victorygs.github.io/MapOfChina/map/data-jiangxi.json";
                var jiangsu = "https://victorygs.github.io/MapOfChina/map/data-jiangsu.json";
                var hunan = "https://victorygs.github.io/MapOfChina/map/data-hunan.json";
                var hubei = "https://victorygs.github.io/MapOfChina/map/data-hubei.json";
                var henan = "https://victorygs.github.io/MapOfChina/map/data-henan.json";
                var heilongjiang = "https://victorygs.github.io/MapOfChina/map/data-heilongjiang.json";
                var hebei = "https://victorygs.github.io/MapOfChina/map/data-hebei.json";
                var guizhou = "https://victorygs.github.io/MapOfChina/map/data-guizhou.json";
                var guangxi = "https://victorygs.github.io/MapOfChina/map/data-guangxi.json";
                var guangdong = "https://victorygs.github.io/MapOfChina/map/data-guangdong.json";
                var gansu = "https://victorygs.github.io/MapOfChina/map/data-gansu.json";
                var chongqing = "https://victorygs.github.io/MapOfChina/map/data-chongqing.json";
                var aomen = "https://victorygs.github.io/MapOfChina/map/data-aomen.json";
                var anhui = "https://victorygs.github.io/MapOfChina/map/data-anhui.json";
                var beijing = "https://victorygs.github.io/MapOfChina/map/data-beijing.json";
                var fujian = "https://victorygs.github.io/MapOfChina/map/data-fujian.json";
                var xianggang = "https://victorygs.github.io/MapOfChina/map/data-xianggang.json";

                /*var $zhongguo = require("./assets/map/data-china.js")
                var $hainan = require("./assets/map/data-hainan.js")
                var $xizang = require("./assets/map/data-xizang.js")
                var $zhejiang =require( "./assets/map/data-zhejiang.js")
                var $yunnan = require("./assets/map/data-yunnan.js")
                var $xinjiang = require("./assets/map/data-xinjiang.js")
                var $tianjin = require("./assets/map/data-tianjin.js")
                var $sichuan = require("./assets/map/data-sichuan.js")
                var $shanxi = require("./assets/map/data-shanxi.js")
                var $shangxi = require("./assets/map/data-shangxi.js")
                var $shanghai = require("./assets/map/data-shanghai.js")
                var $shangdong = require("./assets/map/data-shangdong.js")
                var $qinghai = require("./assets/map/data-qinghai.js")
                var $ningxia = require("./assets/map/data-ningxia.js")
                var $neimenggu = require("./assets/map/data-neimenggu.js")
                var $liaoning = require("./assets/map/data-liaoning.js")
                var $jilin = require("./assets/map/data-jilin.js")
                var $jiangxi = require("./assets/map/data-jiangxi.js")
                var $jiangsu = require("./assets/map/data-jiangsu.js")
                var $hunan = require("./assets/map/data-hunan.js")
                var $hubei = require("./assets/map/data-hubei.js")
                var $henan = require("./assets/map/data-henan.js")
                var $heilongjiang = require("@/assets/map/data-heilongjiang.js")
                var $hebei = require("./assets/map/data-hebei.js")
                var $guizhou = require("./assets/map/data-guizhou.js")
                var $guangxi = require("./assets/map/data-guangxi.js")
                var $guangdong = require("./assets/map/data-guangdong.js")
                var $gansu = require("./assets/map/data-gansu.js")
                var $chongqing = require("./assets/map/data-chongqing.js")
                var $aomen = require("./assets/map/data-aomen.js")
                var $anhui = require("./assets/map/data-anhui.js")
                var $beijing = require("./assets/map/data-beijing.js")
                var $fujian = require("./assets/map/data-fujian.js")
                var $xianggang = require("./assets/map/data-xianggang.js")*/

                // https://victorygs.github.io/MapOfChina/map
                var showArr=this.hotArr.slice(-30)
                var need=showArr.map(v=>v.address.trim().split(' ').reverse()[0])
                var positionObj={}
                var promiseArr=[]
                need.forEach(async(v,index)=>{
                    promiseArr.push(fetch(`http://geo.market.alicloudapi.com/v3/geocode/geo?address=${v}`, {
                        headers: {
                            'Authorization': 'APPCODE f0e20085670b40798090cef073322b7e',
                        },
                    }))
                    /*let r=await fetch(`http://geo.market.alicloudapi.com/v3/geocode/geo?address=${v}`, {
                        // method: "POST",
                        // body: JSON.stringify({ aaa: 1 }),
                        headers: {
                            'Authorization': 'APPCODE f0e20085670b40798090cef073322b7e',
                        },
                    })
                    let d=await r.json()
                    if(d.geocodes[0]){
                        positionObj[d.geocodes[0]['city']]=d.geocodes[0].location.split(',').map(v=>v-0)
                    }*/

                })
                let fetchRes=await Promise.all(promiseArr)
                var promiseArr1=[]
                fetchRes.forEach(v=>{
                    promiseArr1.push(v.json())
                })
                // console.log('fetchRes',fetchRes)
                let fetchRes1=await Promise.all(promiseArr1)
                var finalData=[]
                fetchRes1.reverse().forEach(v=>{//从最热的开始
                    if(v.geocodes[0]){
                        if(Object.keys(positionObj).length>20)return
                        positionObj[v.geocodes[0]['city']]=v.geocodes[0].location.split(',').map(v=>v-0)
                        let name=v.geocodes[0]['city']
                        // console.log('showArr==',name)

                        let {degree}=showArr.find(v=>{
                            let judgeName=v.address.trim().split(' ').reverse()[0]

                            return judgeName.includes(name)||name.includes(judgeName)
                        })
                        finalData.push({degree,name})
                    }
                })
                // console.log('positionObj',positionObj)


                echarts.extendsMap = async function(id, opt) {
                    // 实例
                    var chart = this.init(document.getElementById(id));

                    var curGeoJson = {};
                    var cityMap = {
                        '中国': zhongguo,
                        '上海': shanghai,
                        '河北': hebei,
                        '山西': shangxi,
                        '内蒙古': neimenggu,
                        '辽宁': liaoning,
                        '吉林': jilin,
                        '黑龙江': heilongjiang,
                        '江苏': jiangsu,
                        '浙江': zhejiang,
                        '安徽': anhui,
                        '福建': fujian,
                        '江西': jiangxi,
                        '山东': shangdong,
                        '河南': henan,
                        '湖北': hubei,
                        '湖南': hunan,
                        '广东': guangdong,
                        '广西': guangxi,
                        '海南': hainan,
                        '四川': sichuan,
                        '贵州': guizhou,
                        '云南': yunnan,
                        '西藏': xizang,
                        '陕西': shanxi,
                        '甘肃': gansu,
                        '青海': qinghai,
                        '宁夏': ningxia,
                        '新疆': xinjiang,
                        '北京': beijing,
                        '天津': tianjin,
                        '重庆': chongqing,
                        '香港': xianggang,
                        '澳门': aomen
                    };


                    var geoCoordMap = positionObj  //"磴口": [107.012225,40.337792]

                    console.log("位置==",geoCoordMap)
                    var levelColorMap = {
                        '1': 'rgba(241, 109, 115, .8)',
                        '2': 'rgba(255, 235, 59, .7)',
                        '3': 'rgba(147, 235, 248, 1)'
                    };

                    var defaultOpt = {
                        mapName: 'china', // 地图展示
                        goDown: false, // 是否下钻
                        bgColor: '#404a59', // 画布背景色
                        activeArea: [], // 区域高亮,同echarts配置项
                        data: [],
                        // 下钻回调(点击的地图名、实例对象option、实例对象)
                        callback: function(name, option, instance) {}
                    };
                    if (opt) opt = this.util.extend(defaultOpt, opt);

                    // 层级索引
                    var name = [opt.mapName];
                    var idx = 0;
                    var pos = {
                        leftPlus: 115,
                        leftCur: 150,
                        left: 198,
                        top: 50
                    };

                    var line = [
                        [0, 0],
                        [8, 11],
                        [0, 22]
                    ];
                    // style
                    var style = {
                        font: '18px "Microsoft YaHei", sans-serif',
                        textColor: '#eee',
                        lineColor: 'rgba(147, 235, 248, .8)'
                    };

                    var handleEvents = {
                        /**
                         * i 实例对象
                         * o option
                         * n 地图名
                         **/
                        resetOption: function(i, o, n) {
                            var breadcrumb = this.createBreadcrumb(n);
                            var j = name.indexOf(n);
                            var l = o.graphic.length;
                            if (j < 0) {
                                o.graphic.push(breadcrumb);
                                o.graphic[0].children[0].shape.x2 = 145;
                                o.graphic[0].children[1].shape.x2 = 145;
                                if (o.graphic.length > 2) {
                                    var cityData = [];
                                    var cityJson;
                                    for (var x = 0; x < opt.data.length; x++) {
                                        if(n === opt.data[x].city){
                                            $([opt.data[x]]).each(function(index,data){
                                                cityJson = {city:data.city,name:data.name,value:data.value,crew:data.crew,company:data.company,position:data.position,region:data.region};
                                                cityData.push(cityJson)
                                            })
                                        }
                                    }

                                    if(cityData!=null){
                                        o.series[0].data = handleEvents.initSeriesData(cityData);
                                    }else{
                                        o.series[0].data = [];
                                    }


                                }
                                name.push(n);
                                idx++;
                            } else {
                                o.graphic.splice(j + 2, l);
                                if (o.graphic.length <= 2) {
                                    o.graphic[0].children[0].shape.x2 = 60;
                                    o.graphic[0].children[1].shape.x2 = 60;
                                    o.series[0].data = handleEvents.initSeriesData(opt.data);
                                };
                                name.splice(j + 1, l);
                                idx = j;
                                pos.leftCur -= pos.leftPlus * (l - j - 1);
                            };

                            o.geo.map = n;
                            o.geo.zoom = 0.4;
                            i.clear();
                            i.setOption(o);
                            this.zoomAnimation();
                            opt.callback(n, o, i);
                        },

                        /**
                         * name 地图名
                         **/
                        createBreadcrumb: function(name) {
                            var cityToPinyin = {
                                '中国': 'zhongguo',
                                '上海': 'shanghai',
                                '河北': 'hebei',
                                '山西': 'shangxi',
                                '内蒙古': 'neimenggu',
                                '辽宁': 'liaoning',
                                '吉林': 'jilin',
                                '黑龙江': 'heilongjiang',
                                '江苏': 'jiangsu',
                                '浙江': 'zhejiang',
                                '安徽': 'anhui',
                                '福建': 'fujian',
                                '江西': 'jiangxi',
                                '山东': 'shangdong',
                                '河南': 'henan',
                                '湖北': 'hubei',
                                '湖南': 'hunan',
                                '广东': 'guangdong',
                                '广西': 'guangxi',
                                '海南': 'hainan',
                                '四川': 'sichuan',
                                '贵州': 'guizhou',
                                '云南': 'yunnan',
                                '西藏': 'xizang',
                                '陕西': 'shanxi',
                                '甘肃': 'gansu',
                                '青海': 'qinghai',
                                '宁夏': 'ningxia',
                                '新疆': 'xinjiang',
                                '北京': 'beijing',
                                '天津': 'tianjin',
                                '重庆': 'chongqing',
                                '香港': 'xianggang',
                                '澳门': 'aomen'
                            };
                            var breadcrumb = {
                                type: 'group',
                                id: name,
                                left: pos.leftCur + pos.leftPlus,
                                top: pos.top + 3,
                                children: [{
                                    type: 'polyline',
                                    left: -90,
                                    top: -5,
                                    shape: {
                                        points: line
                                    },
                                    style: {
                                        stroke: '#fff',
                                        key: name
                                        // lineWidth: 2,
                                    },
                                    onclick: function() {
                                        var name = this.style.key;
                                        handleEvents.resetOption(chart, option, name);
                                    }
                                }, {
                                    type: 'text',
                                    left: -68,
                                    top: 'middle',
                                    style: {
                                        text: name,
                                        textAlign: 'center',
                                        fill: style.textColor,
                                        font: style.font
                                    },
                                    onclick: function() {
                                        var name = this.style.text;
                                        handleEvents.resetOption(chart, option, name);
                                    }
                                }, {
                                    type: 'text',
                                    left: -68,
                                    top: 10,
                                    style: {

                                        name: name,
                                        text: cityToPinyin[name] ? cityToPinyin[name].toUpperCase() : '',
                                        textAlign: 'center',
                                        fill: style.textColor,
                                        font: '12px "Microsoft YaHei", sans-serif',
                                    },
                                    onclick: function() {
                                        // console.log(this.style);
                                        var name = this.style.name;
                                        handleEvents.resetOption(chart, option, name);
                                    }
                                }]
                            }

                            pos.leftCur += pos.leftPlus;

                            return breadcrumb;
                        },

                        // 设置effectscatter
                        initSeriesData: function(data) {
                            var temp = [];
                            for (var i = 0; i < data.length; i++) {
                                var geoCoord = geoCoordMap[data[i].name];
                                if (geoCoord) {
                                    temp.push({
                                        name: data[i].name,
                                        value: geoCoord.concat(data[i].value),
                                        degree:data[i].degree,
                                        /*company:data[i].company,
                                        position:data[i].position,
                                        region:data[i].region*/
                                    });
                                }
                            };
                            return temp;
                        },
                        zoomAnimation: function() {
                            var count = null;
                            var zoom = function(per) {
                                if (!count) count = per;
                                count = count + per;
                                // console.log(per,count);
                                chart.setOption({
                                    geo: {
                                        zoom: count
                                    }
                                });
                                if (count < 1) window.requestAnimationFrame(function() {
                                    zoom(0.2);
                                });
                            };
                            window.requestAnimationFrame(function() {
                                zoom(0.2);
                            });
                        }
                    };

                    var option = {
                        backgroundColor: opt.bgColor,
                        tooltip: {
                            show: true,
                            trigger:'item',
                            alwaysShowContent:false,
                            backgroundColor:'rgba(50,50,50,0.7)',
                            hideDelay:100,
                            triggerOn:'mousemove',
                            enterable:true,
                            position:['60%','70%'],
                            formatter:function(params, ticket, callback){
                                return '城市：'+params.data.name+'<br/>'+'气温：'+params.data.degree;
                            }
                        },
                        graphic: [{
                            type: 'group',
                            left: pos.left,
                            top: pos.top - 4,
                            children: [{
                                type: 'line',
                                left: 0,
                                top: -20,
                                shape: {
                                    x1: 0,
                                    y1: 0,
                                    x2: 60,
                                    y2: 0
                                },
                                style: {
                                    stroke: style.lineColor,
                                }
                            }, {
                                type: 'line',
                                left: 0,
                                top: 20,
                                shape: {
                                    x1: 0,
                                    y1: 0,
                                    x2: 60,
                                    y2: 0
                                },
                                style: {
                                    stroke: style.lineColor,
                                }
                            }]
                        },
                            {
                                id: name[idx],
                                type: 'group',
                                left: pos.left + 2,
                                top: pos.top,
                                children: [{
                                    type: 'polyline',
                                    left: 90,
                                    top: -12,
                                    shape: {
                                        points: line
                                    },
                                    style: {
                                        stroke: 'transparent',
                                        key: name[0]
                                    },
                                    onclick: function() {
                                        var name = this.style.key;
                                        handleEvents.resetOption(chart, option, name);
                                    }
                                }, {
                                    type: 'text',
                                    left: 0,
                                    top: 'middle',
                                    style: {
                                        text: '中国',
                                        textAlign: 'center',
                                        fill: style.textColor,
                                        font: style.font
                                    },
                                    onclick: function() {
                                        handleEvents.resetOption(chart, option, '中国');
                                    }
                                }, {
                                    type: 'text',
                                    left: 0,
                                    top: 10,
                                    style: {
                                        text: 'China',
                                        textAlign: 'center',
                                        fill: style.textColor,
                                        font: '12px "Microsoft YaHei", sans-serif',
                                    },
                                    onclick: function() {
                                        handleEvents.resetOption(chart, option, '中国');
                                    }
                                }]
                            }],
                        geo: {
                            map: opt.mapName,
                            roam: true,
                            zoom: 1,
                            label: {
                                normal: {
                                    show: true,
                                    textStyle: {
                                        color: '#fff'
                                    }
                                },
                                emphasis: {
                                    textStyle: {
                                        color: '#fff'
                                    }
                                }
                            },
                            itemStyle: {
                                normal: {
                                    borderColor: 'rgba(147, 235, 248, 1)',
                                    borderWidth: 1,
                                    areaColor: {
                                        type: 'radial',
                                        x: 0.5,
                                        y: 0.5,
                                        r: 0.8,
                                        colorStops: [{
                                            offset: 0,
                                            color: 'rgba(147, 235, 248, 0)' // 0% 处的颜色
                                        }, {
                                            offset: 1,
                                            color: 'rgba(147, 235, 248, .2)' // 100% 处的颜色
                                        }],
                                        globalCoord: false // 缺省为 false
                                    },
                                    shadowColor: 'rgba(128, 217, 248, 1)',
                                    // shadowColor: 'rgba(255, 255, 255, 1)',
                                    shadowOffsetX: -2,
                                    shadowOffsetY: 2,
                                    shadowBlur: 10
                                },
                                emphasis: {
                                    areaColor: '#389BB7',
                                    borderWidth: 0
                                }
                            },
                            regions: opt.activeArea.map(function(item) {
                                if (typeof item !== 'string') {
                                    return {
                                        name: item.name,
                                        itemStyle: {
                                            normal: {
                                                areaColor: item.areaColor || '#389BB7'
                                            }
                                        },
                                        label: {
                                            normal: {
                                                show: item.showLabel,
                                                textStyle: {
                                                    color: '#fff'
                                                }
                                            }
                                        }
                                    }
                                } else {
                                    return {
                                        name: item,
                                        itemStyle: {
                                            normal: {
                                                borderColor: '#91e6ff',
                                                areaColor: '#389BB7'
                                            }
                                        }
                                    }
                                }
                            })
                        },
                        series: [{
                            type: 'effectScatter',
                            coordinateSystem: 'geo',
                            showEffectOn: 'render',
                            rippleEffect: {
                                period:15,
                                scale: 4,
                                brushType: 'fill'
                            },
                            hoverAnimation: true,
                            itemStyle: {
                                normal: {
                                    color: '#FFC848',
                                    shadowBlur: 10,
                                    shadowColor: '#333'
                                }
                            },
                            data: handleEvents.initSeriesData(opt.data)
                        }]
                    };

                    chart.setOption(option);
                    // 添加事件
                    chart.on('click', function(params) {
                        var _self = this;
                        if (opt.goDown && params.name !== name[idx]) {
                            if (cityMap[params.name]) {
                                var url = cityMap[params.name];
                                $.get(url, function(response) {
                                    //console.log(response);
                                    curGeoJson = response;
                                    echarts.registerMap(params.name, response);
                                    handleEvents.resetOption(_self, option, params.name);
                                });
                            }
                        }
                    });

                    chart.setMap = function(mapName) {
                        var _self = this;
                        if (mapName.indexOf('市') < 0) mapName = mapName + '市';
                        var citySource = cityMap[mapName];
                        if (citySource) {
                            var url = './map/' + citySource + '.json';
                            $.get(url, function(response) {
                                // console.log(response);
                                curGeoJson = response;
                                echarts.registerMap(mapName, response);
                                handleEvents.resetOption(_self, option, mapName);
                            });
                        }

                    };

                    return chart;
                };

                $.getJSON(zhongguo,  async function(geoJson) {
                    await setTimeout(()=>{
                        echarts.registerMap('中国', geoJson);
                        var myChart = echarts.extendsMap('top', {
                            bgColor: '#154e90', // 画布背景色
                            mapName: '中国', // 地图名
                            text:'',
                            goDown: true, // 是否下钻
                            // 下钻回调
                            callback: function(name, option, instance) {
                                //console.log(name, option, instance);
                            },
                            // 数据展示
                            data: finalData
                        });
                    },3000)

                })


            }
        },
        mounted(){
            // this.renderChart()
        }

    }

</script>

<style lang="scss">
    *{
        margin: 0;
        padding: 0;
    }
    #top{
        display: flex;
        justify-content: center;
    }
    .andyContain{background: #f6f2f6;}
    .b{
        width:calc(100% - 20px);display: flex;justify-content: center;padding-top: 20px;
        &>div{
            flex-grow: 1;
            min-width: 24%;
            min-height: 400px;
        }
    }
</style>