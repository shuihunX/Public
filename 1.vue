<template>
  <div id="container" width="100%">
    <canvas id="grideCanvas" width="1920" height="600"> </canvas>
    <div id="msg1" :style="{ top: top1 + 'px', left: left1 + 'px' }" v-show="at">
      <p>构筑物名称：{{ caseInfo.name }}</p>
      <p>当前施工：{{ caseInfo.construction }}</p>
      <p>运架设备：{{ caseInfo.equipment }}</p>
      <p>架梁进度：{{ caseInfo.progress }}</p>
      <p>进度评价：{{ caseInfo.rate }}</p>
      <p>架梁总数：{{ caseInfo.workNum }}</p>

      <p>开始里程：{{ caseInfo.startMile }}</p>
      <p>结束里程：{{ caseInfo.endMile }}</p>
      <p>计划开始时间：{{ caseInfo.planStarttime }}</p>
      <p>实际开始时间：{{ caseInfo.actualStarttime }}</p>
      <p>计划完成时间：{{ caseInfo.planfinishtime }}</p>
      <p>实际完成时间：{{ caseInfo.actualfinishtime }}</p>
    </div>
    <div id="msg2" :style="{ top: top2 + 'px', left: left2 + 'px' }" v-show="at2">
      <p>构筑物名称：{{ caseInfo.name }}</p>
      <p>当前施工：{{ caseInfo.construction }}</p>
      <p>运架设备：{{ caseInfo.equipment }}</p>
      <p>架梁进度：{{ caseInfo.progress }}</p>
      <p>进度评价：{{ caseInfo.rate }}</p>
      <p>架梁总数：{{ caseInfo.workNum }}</p>
      <p>开始里程：{{ caseInfo.startMile }}</p>
      <p>结束里程：{{ caseInfo.endMile }}</p>
      <p>计划开始时间：{{ caseInfo.planStarttime }}</p>
      <p>实际开始时间：{{ caseInfo.actualStarttime }}</p>
      <p>计划完成时间：{{ caseInfo.planfinishtime }}</p>
      <p>实际完成时间：{{ caseInfo.actualfinishtime }}</p>
    </div>
    <div id="equipmentName1" :style="{ top: top0 + 'px', left: left0 + 'px' }" v-show="dialogbig">
      {{ equipmentName }}
    </div>
    <div id="equipmentName2" :style="{ top: top0 + 'px', left: left0 + 'px' }" v-show="dialogsmall">
      {{ equipmentName }}
    </div>
  </div>
</template>

<script>
import html2canvas from "html2canvas" // 导出图片
import debounce from "lodash/debounce" // 防抖

// import html2canvas from '@/Excel/html2canvas.js'
import jsPDF from "jspdf" // 导出pdf
export default {
  name: "Gride",
  props: {
    checked: {
      type: Boolean,
      required: true,
      default: () => {},
    },
    checked2: {
      type: Boolean,
      required: true,
      default: () => {},
    },
    size: {
      type: Boolean,
    },
    bigorsmall: {
      type: Boolean,
    },
    isexport: {
      type: Boolean,
    },
    canvasdata: {
      type: Array,
    },
  },
  data() {
    return {
      startX: 0,
      startY: 0,
      left: 0,
      top: 0,
      msg: null,
      at: false,
      at2: false,
      left0: null,
      top0: null,
      left1: null,
      top1: null, //大里程
      left2: null, //小里程
      top2: null,
      endX: null,
      endY: null,
      dialogbig: false,
      dialogsmall: false,
      dialog: false,
      equipmentName: null,
      iconArr: [],
      mouseon: false, //为false表示还未按下过鼠标，为true表示已经按过鼠标,
      totalWidth: null,
      totalHeight: null,
      upYearHeight: 0, //最上面年份高度
      middleYearHeight: 0, //中间年份高度
      lastYearHeight: 0, //最后一个年份高度,
      quarternum: 0,
      selected: false,
      legendWidth: 190,
      caseInfo: {
        name: "",
        construction: "",
        equipment: "",
        progress: "",
        rate: "",
        workNum: "",
        startMile: "",
        endMile: "",
        planStarttime: "",
        actualStarttime: "",
        planfinishtime: "",
        actualfinishtime: "",
      },
      yearData: [],
      canvasData: [
        {
          ylname: "1000t运梁车",
          startno: "DK",
          sjks: "2022-04-16",
          site_code: "20985",
          percent: "100.0%",
          sectionName: "JBSG-1标中铁一局",
          lcLocation: 124318.41,
          jhStart: "2021-10-15",
          jhjlnum: "2",
          sjwc: "2022-04-16",
          end: "127868.731",
          jskm: "141628.79",
          jhend: "2023-01-06",
          sjStart: "2021-06-28",
          endno: "DK",
          jqname: "1000t架桥机",
          earlyTime: "2021-06-28",
          gzw: "苏湖特大桥（吴兴桥段）166#-168#",
          end_mound: 168,
          start: "127803.331",
          end_time: "2022-04-18",
          tyDate: "2023-04-08",
          kslc: "DK127803.331",
          sbState: 0,
          jslc: "DK127868.731",
          start_time: "2022-04-16",
          start_mound: 166,
          jhDate: "2022-04-18",
          kskm: "109189.956",
          planid: "1557281506877296641",
          location: "大里程",
          zt: "正常",
          jd: null,
        },
      ],
      titleName: null,
      canvasInfo: {
        offset: { x: 0, y: 0 },
        scale: 1,
        scaleStep: 0.1,
        maxScale: 2,
        minScale: 0.5,
      },
      realCanvasPosition: {
        x: 0,
        y: 0,
      },
      image: null,
      src: require("@/assets/jia.png"),
    }
  },
  watch: {
    // 监听数据变化
    checked: {
      handler: function () {
        var canvas = document.getElementById("grideCanvas") //获取canvas元素
        var ctx = canvas.getContext("2d") //获取画图环境，指明为2d
        let totalwidth = document.getElementById("container").clientWidth //获取画布宽度
        let titleHeight = 40 //标题高度
        canvas.width = totalwidth //设置画布宽度
        canvas.height = this.totalYearHeight + 150 //设置画布高度
        ctx.clearRect(0, 0, canvas.width, canvas.height) //清空画布
        this.draw(ctx, totalwidth, titleHeight, this.yearData, this.startX, this.startY) //绘制
      },
    },
    // 监听是否显示图例
    checked2: {
      handler: function () {
        var canvas = document.getElementById("grideCanvas") //获取canvas元素
        var ctx = canvas.getContext("2d") //获取画图环境，指明为2d
        let totalwidth = document.getElementById("container").clientWidth //获取画布宽度
        let titleHeight = 40 //标题高度
        canvas.width = totalwidth //设置画布宽度
        canvas.height = this.totalYearHeight + 150 //设置画布高度
        if (this.checked2) {
          //判断是否显示图例
          this.legendWidth = 190 //显示图例
        } else {
          this.legendWidth = 10 //不显示图例
        }
        ctx.clearRect(0, 0, canvas.width, canvas.height) //清空画布
        this.draw(ctx, totalwidth, titleHeight, this.yearData, this.startX, this.startY) //绘制
      },
    },
    //监听是否导出
    isexport: {
      handler: function () {
        this.exportCanvas() //导出
      },
    },
    // 监听是否放大缩小
    size: {
      handler: function () {
        var canvas = document.getElementById("grideCanvas")
        var ctx = canvas.getContext("2d")
        let totalwidth = document.getElementById("container").clientWidth
        let titleHeight = 40
        canvas.width = totalwidth
        if (this.bigorsmall) {
          this.getLarger(canvas, this.canvasInfo, ctx, totalwidth, titleHeight, this.totalYearHeight, true)
        } else {
          this.getLarger(canvas, this.canvasInfo, ctx, totalwidth, titleHeight, this.totalYearHeight, false)
        }
      },
      deep: true,
      immediate: false,
    },
  },
  mounted() {
    this.canvasData = this.canvasdata //获取数据
    var canvas = document.getElementById("grideCanvas") //获取canvas元素
    var ctx = canvas.getContext("2d") //获取画图环境，指明为2d
    // canvas.style.border = '1px solid #000000'
    let totalwidth = document.getElementById("container").clientWidth //获取画布宽度
    let titleHeight = 40 //标题高度
    //年份数据
    let yearData = []
    console.log(yearData, "yeardata")
    if (this.canvasData && this.canvasData.length > 0) {
      this.titleName = this.canvasData[0].sectionName
      //计算年份的取值范围
      let earliestTime = this.canvasData[0].earlyTime.split("-")[0]
      let lastTime = this.canvasData[0].tyDate.split("-")[0]
      console.log(earliestTime, lastTime, "year")
      if (earliestTime != lastTime) {
        for (var e = 0; e < lastTime - earliestTime + 1; e++) {
          let year = (Number(lastTime) - e).toFixed(0)
          yearData.push(year)
        }
      } else {
        yearData.push(earliestTime)
      }
      this.yearData = yearData
      canvas.width = totalwidth
      // canvas.height = 150 + yearData.length *180
      //计算最上面年份季度个数
      let lastyearMonth = this.canvasData[0].tyDate.split("-")[1]
      let quarternum = Math.ceil(lastyearMonth / 3)
      this.quarternum = quarternum
      let upYearHeight = quarternum * 45 //最上面年份高度
      let middleYearHeight = 0 //中间年份高度
      if (yearData.length > 2) {
        middleYearHeight = (yearData.length - 2) * 180
      }
      if (this.checked2) {
        this.legendWidth = 190
      } else {
        this.legendWidth = 10
      }
      //第一个年份（图中最后一个年份）
      let firstYearquarter = Number(this.canvasData[0].earlyTime.split("-")[1])
      let firstquarternum = Math.ceil((12 - firstYearquarter + 1) / 3)
      let lastYearHeight = firstquarternum * 45
      this.upYearHeight = upYearHeight
      this.middleYearHeight = middleYearHeight
      this.lastYearHeight = lastYearHeight
      let totalYearHeight = upYearHeight + middleYearHeight + lastYearHeight
      this.totalYearHeight = totalYearHeight

      canvas.height = 150 + this.totalYearHeight
      this.totalWidth = totalwidth
      this.totalHeight = 150 + this.totalYearHeight
      ctx.strokeStyle = "#000"
      ctx.lineWidth = 1
      // const tempCanvas = document.createElement('canvas'); //新建一个canvas作为缓存canvas
      // const tempCtx = tempCanvas.getContext('2d');
      // tempCanvas.width = totalwidth;
      // tempCanvas.height = this.totalHeight; // 设置宽高
      //  // 开始绘制
      // tempCtx.drawImage(bg,0,0); // 背景
      this.draw(ctx, totalwidth, titleHeight, yearData, 0, 0)
    }

    // //缩放参数
    const canvasInfo = {
      offset: { x: 0, y: 0 },
      scale: 1,
      scaleStep: 0.1,
      maxScale: 2,
      minScale: 0.5,
    }
    const getCanvasPosition = (e) => {
      return {
        x: e.offsetX,
        y: e.offsetY,
      }
    }

    canvas.addEventListener("wheel", (e) => {
      e.preventDefault()
      const canvasPosition = getCanvasPosition(e)
      console.log(canvasPosition, "canvasPosition")
      console.log(this.canvasInfo.offset.x, "this.canvasInfo.offset.x")
      const realCanvasPosition = {
        x: canvasPosition.x - this.canvasInfo.offset.x,
        y: canvasPosition.y - this.canvasInfo.offset.y,
      }
      this.realCanvasPosition = realCanvasPosition
      const { scaleStep } = canvasInfo
      const deltaX = (realCanvasPosition.x / this.canvasInfo.scale) * scaleStep
      const deltaY = (realCanvasPosition.y / this.canvasInfo.scale) * scaleStep
      console.log(deltaX, "deltaX")
      ctx.clearRect(0, 0, canvas.width, canvas.height)
      let canvaswidth = canvas.width
      canvas.width = canvaswidth //如果不重新设置，再次绘制会出现偏移现象
      if (e.wheelDelta > 0) {
        console.log("up")
        this.canvasInfo.offset.x -= deltaX
        this.canvasInfo.offset.y -= deltaY
        this.canvasInfo.scale += scaleStep
      } else {
        console.log("down")
        this.canvasInfo.offset.x += deltaX
        this.canvasInfo.offset.y += deltaY
        this.canvasInfo.scale -= scaleStep
      }
      console.log(this.canvasInfo.offset.x, "xxxxxxxxwheel")
      ctx.setTransform(
        this.canvasInfo.scale,
        0,
        0,
        this.canvasInfo.scale,
        this.canvasInfo.offset.x,
        this.canvasInfo.offset.y
      )
      this.draw(ctx, totalwidth, titleHeight, yearData, this.startX, this.startY)
    })
    let pathXY2 = [] //当前鼠标实时移动位移数组
    var time = 0 //鼠标移动次数
    let lastMovetotal = [] //所有移动次数的位移之和
    canvas.onmousedown = (args) => {
      let _this = this
      var evt = args || event
      var StartX = evt.clientX
      var StartY = evt.clientY
      time++
      console.log("mousedown")
      let lastMove =
        sessionStorage.getItem("lastMove") != null ? JSON.parse(sessionStorage.getItem("lastMove")) : [{ x: 0, y: 0 }]
      console.log(lastMove, "lastMove")
      lastMovetotal.push(lastMove[0])
      let x = 0,
        y = 0
      console.log(lastMovetotal, "lastMovetotal")
      if (window.sessionStorage.getItem("lastMove") != null) {
        for (var j = 0; j < time; j++) {
          x += lastMovetotal[j].x
          y += lastMovetotal[j].y
        }
      }

      console.log("down")
      document.onmousemove = (ev) => {
        var iEvent = ev || event
        // 鼠标移动的距离
        let left = iEvent.clientX - StartX
        let top = iEvent.clientY - StartY
        pathXY2.push({
          x: left,
          y: top,
        })
        let result = pathXY2.slice(-1)
        window.sessionStorage.setItem("lastMove", JSON.stringify(result))
        console.log(result, time, "result====time")
        this.endX = iEvent.clientX
        this.endY = iEvent.clientY
        // 鼠标移动的距离 + 最开始的top 或者 left 就是最终的位置
        // canvas.style.left = left + initleft + 'px';
        // canvas.style.top = top + inittop +'px';
        ctx.clearRect(0, 0, canvas.width, canvas.height)
        let canvaswidth = canvas.width
        canvas.width = canvaswidth
        console.log(this.endX - this.startX, this.endY - this.startY, "位移---")
        let time2 = pathXY2.length
        // _this.draw(ctx,totalwidth,titleHeight,yearData,left,top)
        //根据鼠标移动的频率，重新绘制图形，只要鼠标位移改变，就会重新画图，
        // 偏移量为所有位移之和（当前实时偏移量加上鼠标多次拖动的位移之和）
        _this.startX = parseInt(x + pathXY2[time2 - 1].x)
        _this.startY = parseInt(y + pathXY2[time2 - 1].y)
        ctx.setTransform(
          this.canvasInfo.scale,
          0,
          0,
          this.canvasInfo.scale,
          this.canvasInfo.offset.x,
          this.canvasInfo.offset.y
        )
        _this.draw(
          ctx,
          totalwidth,
          titleHeight,
          yearData,
          parseInt(x + pathXY2[time2 - 1].x),
          parseInt(y + pathXY2[time2 - 1].y)
        )
      }
      document.onmouseup = function () {
        document.onmousedown = null
        document.onmousemove = null
        console.log("up")
      }
      return false
    }

    const canvasMouseMove = debounce((e) => {
      e.preventDefault()
      let _this = this
      var evt = e || event
      var x = evt.clientX - canvas.getBoundingClientRect().left
      var y = evt.clientY - canvas.getBoundingClientRect().top
      //  let oy = (y == undefined ? 0 : y);//纵向偏移
      //  if((oy & 1) == 0){
      // draw();
      _this.drawrollLine(ctx, titleHeight, _this.startX, _this.startY, totalwidth, x, y)
      //  }//基本不闪了
    }, 50)

    canvas.addEventListener("mousemove", canvasMouseMove)
  },
  methods: {
    draw(ctx, totalwidth, titleHeight, yearData, startX, startY) {
      if (this.canvasData && this.canvasData.length > 0) {
        ctx.save()
        ctx.beginPath()
        ctx.fillStyle = "#3A3F5D"
        ctx.font = "22px Arial"
        let titleLength = this.titleName.length * 28
        ctx.fillText(this.titleName + "架梁进度形象图", totalwidth / 2 - titleLength / 2 + startX, 19 + startY)
        ctx.fill()
        //计算最上面年份季度个数
        let lastyearMonth = this.canvasData[0].tyDate.split("-")[1]
        let quarternum = Math.ceil(lastyearMonth / 3)
        let upYearHeight = quarternum * 45 //最上面年份高度
        let middleYearHeight = 0 //中间年份高度
        if (this.checked2) {
          this.legendWidth = 190
        } else {
          this.legendWidth = 10
        }
        if (yearData.length > 2) {
          middleYearHeight = (yearData.length - 2) * 180
        }
        //第一个年份（图中最后一个年份）
        let firstYear = yearData[yearData.length - 1]
        let firstYearquarter = Number(this.canvasData[0].earlyTime.split("-")[1])
        let firstquarternum = Math.ceil((12 - firstYearquarter + 1) / 3)
        let lastYearHeight = firstquarternum * 45
        this.upYearHeight = upYearHeight
        this.middleYearHeight = middleYearHeight
        this.lastYearHeight = lastYearHeight
        let totalYearHeight = upYearHeight + middleYearHeight + lastYearHeight
        this.totalYearHeight = totalYearHeight
        //顶线
        ctx.beginPath()
        ctx.moveTo(startX, titleHeight + startY + 0.5)
        ctx.lineTo(totalwidth + startX, titleHeight + startY + 0.5)
        ctx.stroke()
        //左线
        ctx.beginPath()
        ctx.moveTo(0.5 + startX, titleHeight + startY)
        ctx.lineTo(0.5 + startX, totalYearHeight + titleHeight + 100 + startY)
        ctx.stroke()
        //梁场线 高30
        ctx.beginPath()
        ctx.moveTo(1 + startX, totalYearHeight + titleHeight + 5 + startY + 0.5)
        ctx.lineTo(totalwidth + startX - this.legendWidth, totalYearHeight + titleHeight + 5 + startY + 0.5)
        console.log(totalwidth + startX - this.legendWidth, "梁场线---")
        ctx.strokeStyle = "blue"
        ctx.stroke()
        //施工进度
        ctx.beginPath()
        ctx.moveTo(25 + startX + 0.5, titleHeight + startY)
        ctx.lineTo(25 + startX + 0.5, totalYearHeight + titleHeight + startY)
        //年份右线
        // ctx.beginPath()
        ctx.moveTo(80.5 + startX, titleHeight + startY)
        ctx.lineTo(80.5 + startX, totalYearHeight + titleHeight + startY)
        ctx.strokeStyle = "#000"
        ctx.stroke()
        let titleH = totalYearHeight / 2 + titleHeight
        ctx.font = "12px Arial"
        ctx.beginPath()
        this.writeTextOnCanvas(ctx, 10 + startX, titleH + startY, "度进工施")
        //年份底线,高度100
        let start = 25 //最大年份的起点x坐标
        let end = totalwidth - this.legendWidth //所有年份的终点X坐标
        // let end = totalwidth-190//所有年份的终点X坐标
        ctx.beginPath()
        //最后一个年份的底线(第一条年份底线)
        let lastYear = yearData[0]
        this.writeTextOnCanvas(ctx, 60 + startX, (quarternum * 45) / 2 - 9 + startY + titleHeight, lastYear)
        ctx.moveTo(start + startX, quarternum * 15 * 3 + titleHeight + startY + 0.5)
        ctx.lineTo(end + startX, quarternum * 15 * 3 + titleHeight + startY + 0.5)
        ctx.stroke()

        //季度分割线
        //起始点
        let quarterstart = 30 + 50
        let quarterend = 30 + 50 + 80
        //最上季度分割线
        ctx.beginPath()
        if (quarternum > 1) {
          //画季度分割线
          for (var q1 = 0; q1 < quarternum + 1; q1++) {
            ctx.moveTo(quarterstart + startX, (quarternum - q1) * 45 + titleHeight + startY + 0.5)
            ctx.lineTo(quarterend + startX, (quarternum - q1) * 45 + titleHeight + startY + 0.5)
            ctx.stroke()
            if (q1 == 1) {
              ctx.fillText("第一季度", 100 + startX, (quarternum + 1 - q1) * 45 + titleHeight + startY - 15)
              ctx.fill()
            } else if (q1 == 2) {
              ctx.fillText("第二季度", 100 + startX, (quarternum + 1 - q1) * 45 + titleHeight + startY - 15)
              ctx.fill()
            } else if (q1 == 3) {
              ctx.fillText("第三季度", 100 + startX, (quarternum + 1 - q1) * 45 + titleHeight + startY - 15)
              ctx.fill()
            } else if (q1 == 4) {
              ctx.fillText("第四季度", 100 + startX, (quarternum + 1 - q1) * 45 + titleHeight + startY - 15)
              ctx.fill()
            }
          }
        } else {
          //无需画线，年份即是分割线
          ctx.fillText("第一季度", 100 + startX, 45 + titleHeight + startY - 15)
          ctx.fill()
        }
        // //第一个年份（图中最后一个年份）
        // let firstYear = yearData[yearData.length-1]
        // let firstYearquarter = this.canvasData[0].earlyTime.split('-')[1]
        // let firstquarternum =  Math.ceil(firstYearquarter/3)
        // let lastYearHeight = firstquarternum*45
        console.log(lastYearHeight, "lastYearHeight")
        ctx.beginPath()
        //如果第一个年份有4个季度，正常遍历画线，否则，单独画线
        let yearLength = firstquarternum === 4 ? yearData.length : yearData.length - 1
        let yearH = upYearHeight + titleHeight - 10
        for (let m = 1; m < yearLength; m++) {
          //年份文字排版(
          this.writeTextOnCanvas(ctx, 60 + startX, yearH + startY + (180 * m) / 2, yearData[m])
          ctx.moveTo(start + startX, upYearHeight + 180 * m + titleHeight + startY + 0.5)
          ctx.lineTo(end + startX, upYearHeight + 180 * m + titleHeight + startY + 0.5)
          ctx.stroke()
        }

        if (firstquarternum && firstquarternum < 4) {
          ctx.beginPath()
          this.writeTextOnCanvas(
            ctx,
            60 + startX,
            yearH + 180 * (yearData.length - 2) + lastYearHeight / 2 + startY,
            firstYear
          )
          ctx.moveTo(
            start + startX,
            upYearHeight + (yearLength - 1) * 180 + lastYearHeight + titleHeight + startY + 0.5
          )
          ctx.lineTo(end + startX, upYearHeight + (yearLength - 1) * 180 + lastYearHeight + titleHeight + startY + 0.5)
          ctx.stroke()
          // console.log(upYearHeight+yearLength*15*3 + lastYearHeight + titleHeight+startY +0.5,'lastzuobiao ====')
        }
        //最早年份分割线（图中最后一个年份）

        if (firstquarternum > 1) {
          //画季度分割线
          for (var q2 = 0; q2 < firstquarternum + 1; q2++) {
            ctx.moveTo(
              quarterstart + startX,
              upYearHeight + middleYearHeight + (firstquarternum - q2) * 45 + titleHeight + startY + 0.5
            )
            ctx.lineTo(
              quarterend + startX,
              upYearHeight + middleYearHeight + (firstquarternum - q2) * 45 + titleHeight + startY + 0.5
            )
            ctx.stroke()
            if (q2 == 1) {
              ctx.fillText(
                "第四季度",
                100 + startX,
                upYearHeight + middleYearHeight + q2 * 45 + titleHeight + startY - 15
              )
              ctx.fill()
            } else if (q2 == 2) {
              ctx.fillText(
                "第三季度",
                100 + startX,
                upYearHeight + middleYearHeight + q2 * 45 + titleHeight + startY - 15
              )
              ctx.fill()
            } else if (q2 == 3) {
              ctx.fillText(
                "第二季度",
                100 + startX,
                upYearHeight + middleYearHeight + q2 * 45 + titleHeight + startY - 15
              )
              ctx.fill()
            } else if (q2 == 4) {
              ctx.fillText(
                "第一季度",
                100 + startX,
                upYearHeight + middleYearHeight + q2 * 45 + titleHeight + startY - 15
              )
              ctx.fill()
            }
          }
        } else {
          //无需画线，年份即是分割线
          ctx.fillText("第四季度", 100 + startX, 45 + titleHeight + startY - 15)
          ctx.fill()
        }
        //中间年份季度分割线
        ctx.beginPath()
        for (let n = 1; n <= (yearData.length - 2) * 4; n++) {
          if (n % 4 != 0) {
            ctx.moveTo(quarterstart + startX, upYearHeight + 45 * n + titleHeight + startY + 0.5)
            ctx.lineTo(quarterend + startX, upYearHeight + 45 * n + titleHeight + startY + 0.5)
            ctx.stroke()
          }
          if (n % 4 == 1) {
            ctx.fillText("第四季度", 100 + startX, upYearHeight + 45 * n + titleHeight + startY - 15)
            ctx.fill()
          } else if (n % 4 == 2) {
            ctx.fillText("第三季度", 100 + startX, upYearHeight + 45 * n + titleHeight + startY - 15)
            ctx.fill()
          } else if (n % 4 == 3) {
            ctx.fillText("第二季度", 100 + startX, upYearHeight + 45 * n + titleHeight + startY - 15)
            ctx.fill()
          } else if (n % 4 == 0) {
            ctx.fillText("第一季度", 100 + startX, upYearHeight + 45 * n + titleHeight + startY - 15)
            ctx.fill()
          }
        }
        //季度右线
        ctx.beginPath()
        ctx.moveTo(159.5 + startX, titleHeight + startY)
        ctx.lineTo(159.5 + startX, totalYearHeight + titleHeight + startY)
        ctx.stroke()
        //12个月份
        let monthwidth = 40
        //月份起点x坐标
        // let monthStart = 160 + monthwidth
        let monthY = titleHeight + startY
        //最上面年份对应月份线
        for (var c = 0; c < quarternum * 3; c++) {
          //月份线
          ctx.save()
          ctx.beginPath()
          ctx.setLineDash([0, 0])
          ctx.moveTo(160 + startX, 15 * c + monthY + 0.5)
          ctx.lineTo(160 + monthwidth + startX, 15 * c + monthY + 0.5)
          // console.log(15*c + monthY+0.5,'upline----')
          ctx.stroke()
          ctx.restore()
          //月份文字
          this.fillMonthText(ctx, quarternum, c, startX, titleHeight, startY, 0, 0)
          //月份分割指示线
          ctx.save()
          ctx.beginPath()
          ctx.setLineDash([3, 3])
          let dashWidth = Math.floor((totalwidth - this.legendWidth - 200 - 12 * 9) / 9)
          for (var line = 0; line < 9; line++) {
            ctx.save()
            ctx.beginPath()
            ctx.setLineDash([3, 3])
            if (c % 12 != 0) {
              this.MonthdashLine(c, ctx, line, startX, startY, titleHeight, 0, 0, quarternum, dashWidth)
              ctx.moveTo(200 + startX + dashWidth * line + 12 * line, titleHeight + startY + 15 * c + 0.5)
              ctx.lineTo(200 + startX + dashWidth * (line + 1) + 12 * line, titleHeight + startY + 15 * c + 0.5)
              ctx.strokeStyle = "#ccc"
              ctx.stroke()
              ctx.restore()
            }
          }
        }
        //中间年份对应月份线
        if (yearData.length > 2) {
          for (var d = 0; d <= (yearData.length - 2) * 12; d++) {
            //月份线
            ctx.save()
            ctx.beginPath()
            ctx.setLineDash([0, 0])
            ctx.moveTo(160 + startX, upYearHeight + 15 * d + monthY + 0.5)
            ctx.lineTo(160 + monthwidth + startX, upYearHeight + 15 * d + monthY + 0.5)
            ctx.stroke()
            ctx.restore()
            //中间月份文字
            let middlequarter = (yearData.length - 2) * 4 //中间年份的季度总和
            // this.fillMonthText(ctx,middlequarter,c,startX,titleHeight,startY,upYearHeight,0)
            if (d == 1) {
              ctx.fillText(middlequarter * 3 - d + 1, 175 + startX, upYearHeight + 15 * d + titleHeight + startY - 2)
              ctx.fill()
            } else if (d == 2) {
              ctx.fillText(middlequarter * 3 - d + 1, 175 + startX, upYearHeight + 15 * d + titleHeight + startY - 2)
              ctx.fill()
            } else if (d == 3) {
              ctx.fillText(middlequarter * 3 - d + 1, 175 + startX, upYearHeight + 15 * d + titleHeight + startY - 2)
              ctx.fill()
            } else if (d == 4) {
              ctx.fillText(middlequarter * 3 - d + 1, 175 + startX, upYearHeight + 15 * d + titleHeight + startY - 2)
              ctx.fill()
            } else if (d == 5) {
              ctx.fillText(middlequarter * 3 - d + 1, 175 + startX, upYearHeight + 15 * d + titleHeight + startY - 2)
              ctx.fill()
            } else if (d == 6) {
              ctx.fillText(middlequarter * 3 - d + 1, 175 + startX, upYearHeight + 15 * d + titleHeight + startY - 2)
              ctx.fill()
            } else if (d == 7) {
              ctx.fillText(middlequarter * 3 - d + 1, 175 + startX, upYearHeight + 15 * d + titleHeight + startY - 2)
              ctx.fill()
            } else if (d == 8) {
              ctx.fillText(middlequarter * 3 - d + 1, 175 + startX, upYearHeight + 15 * d + titleHeight + startY - 2)
              ctx.fill()
            } else if (d == 9) {
              ctx.fillText(middlequarter * 3 - d + 1, 175 + startX, upYearHeight + 15 * d + titleHeight + startY - 2)
              ctx.fill()
            } else if (d == 10) {
              ctx.fillText(middlequarter * 3 - d + 1, 175 + startX, upYearHeight + 15 * d + titleHeight + startY - 2)
              ctx.fill()
            } else if (d == 11) {
              ctx.fillText(middlequarter * 3 - d + 1, 175 + startX, upYearHeight + 15 * d + titleHeight + startY - 2)
              ctx.fill()
            } else if (d == 12) {
              ctx.fillText(middlequarter * 3 - d + 1, 175 + startX, upYearHeight + 15 * d + titleHeight + startY - 2)
              ctx.fill()
            }

            //月份分割指示线
            ctx.save()
            ctx.beginPath()
            ctx.setLineDash([3, 3])
            let dashWidth = Math.floor((totalwidth - this.legendWidth - 200 - 12 * 9) / 9)
            for (var line2 = 0; line2 < 9; line2++) {
              ctx.save()
              ctx.beginPath()
              ctx.setLineDash([3, 3])
              if (d % 12 != 0) {
                // this.MonthdashLine(d,ctx,line2,startX,startY,titleHeight,upYearHeight,middlequarter,dashWidth)
                //月份指示文字
                ctx.fillStyle = "#ccc"
                ctx.font = "10px Microsoft Yahei"
                ctx.fillText(
                  d + 1,
                  200 + startX + dashWidth * (line2 + 1) + 12 * line2,
                  upYearHeight + titleHeight + startY + 15 * (middlequarter * 3 - d) + 0.5
                )
                ctx.fill()
                ctx.moveTo(
                  200 + startX + dashWidth * line2 + 12 * line2,
                  upYearHeight + titleHeight + startY + 15 * d + 0.5
                )
                ctx.lineTo(
                  200 + startX + dashWidth * (line2 + 1) + 12 * line2,
                  upYearHeight + titleHeight + startY + 15 * d + 0.5
                )
                ctx.strokeStyle = "#ccc"
                ctx.stroke()
                ctx.restore()
              }
            }
          }
        }
        //最下年份对应月份底线firstquarternum
        for (var e = 0; e < firstquarternum * 3; e++) {
          //月份线
          ctx.save()
          ctx.beginPath()
          ctx.setLineDash([0, 0])
          ctx.moveTo(160 + startX, upYearHeight + middleYearHeight + 15 * e + monthY + 0.5)
          ctx.lineTo(160 + monthwidth + startX, upYearHeight + middleYearHeight + 15 * e + monthY + 0.5)
          console.log(15 * c + monthY + 0.5, "upline----")
          ctx.stroke()
          ctx.restore()
          //最下面月份文字
          this.downfillMonthText(ctx, e, startX, titleHeight, startY, upYearHeight, middleYearHeight)
          //月份分割指示线
          ctx.save()
          ctx.beginPath()
          ctx.setLineDash([3, 3])
          let dashWidth = Math.floor((totalwidth - this.legendWidth - 200 - 12 * 9) / 9)
          for (var line3 = 0; line3 < 9; line3++) {
            ctx.save()
            ctx.beginPath()
            ctx.setLineDash([3, 3])
            if (c % 12 != 0) {
              this.downMonthdashLine(
                e,
                ctx,
                line3,
                startX,
                startY,
                titleHeight,
                upYearHeight,
                middleYearHeight,
                firstquarternum,
                dashWidth
              )
              if (e > 0) {
                //顶行不画分割线
                ctx.moveTo(
                  200 + startX + dashWidth * line3 + 12 * line3,
                  upYearHeight + middleYearHeight + titleHeight + startY + 15 * e + 0.5
                )
                ctx.lineTo(
                  200 + startX + dashWidth * (line3 + 1) + 12 * line3,
                  upYearHeight + middleYearHeight + titleHeight + startY + 15 * e + 0.5
                )
                ctx.strokeStyle = "#ccc"
                ctx.stroke()
              }
              ctx.restore()
            }
          }
        }
        //月份右线
        ctx.beginPath()
        ctx.setLineDash([])
        ctx.moveTo(159.5 + monthwidth + startX, titleHeight + startY)
        ctx.lineTo(159.5 + monthwidth + startX, totalYearHeight + 30 + titleHeight + startY)
        ctx.stroke()
        //右侧图表
        ctx.beginPath()
        ctx.moveTo(totalwidth + startX - 0.5, titleHeight + startY)
        ctx.lineTo(totalwidth + startX - 0.5, totalYearHeight + titleHeight + 90 + startY)
        ctx.moveTo(totalwidth + startX - this.legendWidth + 10 - 0.5, titleHeight + startY)
        ctx.lineTo(totalwidth + startX - this.legendWidth + 10 - 0.5, totalYearHeight + titleHeight + 90 + startY)
        ctx.strokeStyle = "#000"
        ctx.stroke()
        ctx.beginPath()
        ctx.moveTo(totalwidth + startX - this.legendWidth - 0.5, titleHeight + startY)
        ctx.lineTo(totalwidth + startX - this.legendWidth - 0.5, totalYearHeight + titleHeight + 98 + startY)
        ctx.stroke()
        //制梁场 统一里程 底线
        ctx.beginPath()
        ctx.moveTo(0.5 + startX, totalYearHeight + titleHeight + 89 + startY + 0.5)
        ctx.lineTo(0.5 + totalwidth + startX, totalYearHeight + titleHeight + 89 + startY + 0.5)
        ctx.strokeStyle = "#000000"
        ctx.stroke()
        ctx.fillText("制梁场", 200 / 2 - 20 + startX, totalYearHeight + titleHeight + 28 + startY)
        ctx.fillText("统一里程", 200 / 2 - 25 + startX, totalYearHeight + titleHeight + 80 + startY)
        //标段布置--箭头
        let leftdollX = 200 + startX
        let leftdollY = totalYearHeight + titleHeight + 20 + startY
        //这只三角箭头长边为20px 夹角为48度
        let topdollY = leftdollY - 20 * Math.sin((24 * Math.PI) / 180)
        let topdollX = 200 + 20 * Math.cos((24 * Math.PI) / 180) + startX
        let bottomDollX = 200 + 20 * Math.cos((24 * Math.PI) / 180) + startX
        let bottomDollY = leftdollY + 20 * Math.sin((24 * Math.PI) / 180)
        //左箭头
        ctx.save()
        ctx.beginPath()
        ctx.strokeStyle = "#49fd49"
        ctx.moveTo(leftdollX, leftdollY)
        ctx.lineTo(topdollX, topdollY)
        ctx.lineTo(bottomDollX, bottomDollY)
        ctx.fillStyle = "#49fd49"
        ctx.fill()
        ctx.closePath()
        ctx.stroke()
        ctx.restore()
        //右箭头
        ctx.save()
        ctx.beginPath()
        ctx.strokeStyle = "#49fd49"
        ctx.moveTo(totalwidth - this.legendWidth - 1 + startX, leftdollY)
        ctx.lineTo(totalwidth - 20 * Math.cos((24 * Math.PI) / 180) - this.legendWidth + startX, topdollY)
        ctx.lineTo(totalwidth - 20 * Math.cos((24 * Math.PI) / 180) - this.legendWidth + startX, bottomDollY)
        ctx.fillStyle = "#49fd49"
        ctx.fill()
        ctx.closePath()
        ctx.stroke()
        ctx.restore()
        //三条实线
        const actualstartTime = this.canvasData[0].sjStart //标段实际开始时间--蓝色实线
        const actualplanTime = this.canvasData[0].jhend //标段计划完成时间--红色实线
        const actualtyTime = this.canvasData[0].tyDate //标段推演完成时间--黄色实线
        // let linetimeY =  actualtyTime.split('-')[0]  - actualstartTime.split('-')[0] //计算多少个年份
        // let linetimeY2 =  actualtyTime.split('-')[0]  - actualplanTime.split('-')[0] //计算多少个年份
        // let linetimeY3 =  actualtyTime.split('-')[0]  - actualtyTime.split('-')[0] //计算多少个年份
        let monthy = parseInt(actualstartTime.split("-")[1])
        let monthy2 = parseInt(actualplanTime.split("-")[1])
        let monthy3 = parseInt(actualtyTime.split("-")[1])
        let datey = parseInt(actualstartTime.split("-")[2])
        let datey2 = parseInt(actualplanTime.split("-")[2])
        let datey3 = parseInt(actualtyTime.split("-")[2])
        let actualstartYear = Number(actualstartTime.split("-")[0])
        let actualplanYear = Number(actualplanTime.split("-")[0])
        let actualtyYear = Number(actualtyTime.split("-")[0])
        let yeardate = actualstartYear
        let lastyear = Number(lastYear)
        let downYear = Number(firstYear)
        let lineY1 = this.markLine(yeardate, lastyear, downYear, monthy, datey, startY, titleHeight)
        let lineY2 = this.markLine(actualplanYear, lastyear, downYear, monthy2, datey2, startY, titleHeight)
        let lineY3 = this.markLine(actualtyYear, lastyear, downYear, monthy3, datey3, startY, titleHeight)
        // if(yeardate === lastYear){//最上面
        //   return  lineY = (this.quarternum*3 - monthy + (31-datey)/31)*15 +startY + titleHeight

        // }else if(yeardate < lastYear && yeardate > downYear){//中间
        //     let middleheight = ((lastYear - yeardate - 1)+12 - monthy + (31-datey)/31)*15
        //    return lineY = upYearHeight + middleheight + startY + titleHeight

        // }else if(yeardate === downYear){//最下面
        //     let downheight = (12 - monthy + (31-datey)/31)*15
        //     return lineY = upYearHeight + middleheight + downheight + startY + titleHeight

        // }

        // let lineY1 = linetimeY*180 +titleHeight + startY + (12-monthy )*15 + 15*((30- datey)/30)+0.5
        // let lineY2 = linetimeY2*180 +titleHeight + startY + (12-monthy2 )*15+ 15*((30- datey2)/30)+0.5
        // let lineY3 = parseInt(linetimeY3*180 +titleHeight + startY + (12-monthy3 )*15+ 15*((30- datey3)/30))+0.5
        console.log(lineY1, lineY2, lineY3, "liney2,liney3")
        ctx.save()
        ctx.beginPath()
        // ctx.lineWidth = 1
        ctx.strokeStyle = "#108de9"
        ctx.moveTo(200 + startX, lineY1)
        ctx.lineTo(totalwidth - this.legendWidth + startX, lineY1)
        ctx.stroke()
        ctx.beginPath()
        ctx.strokeStyle = "#ec808d"
        ctx.moveTo(200 + startX, lineY2)
        ctx.lineTo(totalwidth - this.legendWidth + startX, lineY2)
        ctx.stroke()
        ctx.beginPath()
        ctx.strokeStyle = "#f59a23"
        ctx.moveTo(200 + startX, lineY3)
        ctx.lineTo(totalwidth - this.legendWidth + startX, lineY3)
        ctx.stroke()
        ctx.restore()

        //底部刻度
        //标段起始及终点里程
        // let startmileage = Math.trunc(109031/1000)
        // let endmileage = Math.trunc(141066/1000)
        let startmileage = this.canvasData[0].kskm == 0 ? 0 : Math.trunc(this.canvasData[0].kskm / 1000)
        console.log(startmileage, "startmileage")
        let endmileage = Math.trunc(this.canvasData[0].jskm / 1000)
        ctx.beginPath()
        ctx.moveTo(159.5 + monthwidth + startX, totalYearHeight + titleHeight + 40 + startY + 0.5)
        ctx.lineTo(159.5 + monthwidth + startX, totalYearHeight + titleHeight + 98 + startY + 0.5)
        ctx.stroke()
        //Math.trunc() 舍弃小数取整
        // console.log(startmileage,'起始点')
        let calibration = this.find(startmileage) //起点整刻度
        //刻度数量计算 5为刻度之间相差5KD里程
        let calibrationNum = Math.trunc((endmileage - calibration) / 5) + 1
        console.log(calibrationNum, "calibrationNum")
        //整刻度起始坐标点
        let calibrationwidth = (totalwidth - this.legendWidth - 160 - 30) / calibrationNum
        let calibraStarX = 200 + 30 + 0.5 + startX
        let calibraStarY = totalYearHeight + titleHeight + 52 + startY + 0.5
        ctx.beginPath()
        this.writeTextOnCanvas(ctx, 213 + startX, calibraStarY, "DK" + startmileage + "")
        for (var y = 0; y < calibrationNum; y++) {
          this.writeTextOnCanvas(
            ctx,
            calibraStarX + 14 + Math.trunc(calibrationwidth * y),
            calibraStarY,
            "DK" + (calibration + 5 * y) + ""
          )
          ctx.moveTo(calibraStarX + Math.trunc(calibrationwidth * y), calibraStarY)
          ctx.lineTo(calibraStarX + Math.trunc(calibrationwidth * y), calibraStarY + 48)
          ctx.stroke()
        }

        //梁场中线
        let middleLineX = parseInt((totalwidth - 200 - this.legendWidth) / 2 + 200) + startX
        ctx.save()
        ctx.beginPath()
        ctx.setLineDash([3, 3])
        ctx.moveTo(middleLineX + 0.5, titleHeight + startY)
        ctx.lineTo(middleLineX + 0.5, titleHeight + totalYearHeight + 34 + startY)
        ctx.strokeStyle = "#ff07ff"
        ctx.stroke()
        ctx.restore()
        //制梁场 中间绿色线段
        ctx.beginPath()
        ctx.lineWidth = 2
        ctx.moveTo(middleLineX - 10, titleHeight + totalYearHeight + 20 + startY)
        ctx.lineTo(middleLineX + 10, titleHeight + totalYearHeight + 20 + startY)
        ctx.stroke()
        ctx.beginPath()
        ctx.moveTo(middleLineX - 10, titleHeight + totalYearHeight + 16 + startY)
        ctx.lineTo(middleLineX - 10, titleHeight + totalYearHeight + 24 + startY)
        ctx.stroke()
        ctx.beginPath()
        ctx.moveTo(middleLineX + 10, titleHeight + totalYearHeight + 16 + startY)
        ctx.lineTo(middleLineX + 10, titleHeight + totalYearHeight + 24 + startY)
        ctx.stroke()
        ctx.save()
        ctx.beginPath()
        ctx.strokeStyle = "#49fd49"
        ctx.moveTo(200 + startX, titleHeight + totalYearHeight + 20 + startY)
        ctx.lineTo(middleLineX - 15, titleHeight + totalYearHeight + 20 + startY)
        ctx.stroke()
        //箭头中间绿色线
        ctx.beginPath()
        ctx.moveTo(middleLineX + 15, titleHeight + totalYearHeight + 20 + startY)
        ctx.lineTo(totalwidth - this.legendWidth - 1 - 10 + startX, titleHeight + totalYearHeight + 20 + startY)
        ctx.stroke()
        ctx.restore()
        //右侧指示说明图例
        if (this.checked2) {
          //start
          //横线部分
          let lineX1 = totalwidth - 180 + startX
          let lineX2 = totalwidth + startX
          let lineY = null
          ctx.save()
          for (var l = 0; l < 10; l++) {
            lineY = totalYearHeight + titleHeight + startY + 49 - 40 * l + 0.5
            ctx.beginPath()
            ctx.lineWidth = 1
            ctx.font = "12px 新宋体 常规, 新宋体, sans-serif"
            ctx.fillStyle = "#000"
            if (l == 0) {
              ctx.fillText(
                "架桥机",
                totalwidth - 60 + startX,
                totalYearHeight + titleHeight + startY + 89 - 40 * l - 12
              )
            }
            if (l == 1) {
              ctx.fillText(
                "架梁完工",
                totalwidth - 70 + startX,
                totalYearHeight + titleHeight + startY + 89 - 40 * l - 12
              )
            }
            if (l == 2) {
              ctx.fillText(
                "架梁滞后",
                totalwidth - 70 + startX,
                totalYearHeight + titleHeight + startY + 89 - 40 * l - 12
              )
            }
            if (l == 3) {
              ctx.fillText(
                "架梁正常",
                totalwidth - 70 + startX,
                totalYearHeight + titleHeight + startY + 89 - 40 * l - 12
              )
            }
            if (l == 4) {
              ctx.fillText(
                "架梁工程",
                totalwidth - 70 + startX,
                totalYearHeight + titleHeight + startY + 89 - 40 * l - 12
              )
            }
            if (l == 5) {
              ctx.fillText(
                "推演完成时间",
                totalwidth - 80 + startX,
                totalYearHeight + titleHeight + startY + 89 - 40 * l - 12
              )
            }
            if (l == 6) {
              ctx.fillText(
                "计划完成时间",
                totalwidth - 80 + startX,
                totalYearHeight + titleHeight + startY + 89 - 40 * l - 12
              )
            }
            if (l == 7) {
              ctx.fillText(
                "实际开始时间",
                totalwidth - 80 + startX,
                totalYearHeight + titleHeight + startY + 89 - 40 * l - 12
              )
            }
            if (l == 8) {
              ctx.fillText(
                "制梁场",
                totalwidth - 60 + startX,
                totalYearHeight + titleHeight + startY + 89 - 40 * l - 12
              )
            }
            if (l == 9) {
              ctx.fillText(
                "标段布置",
                totalwidth - 70 + startX,
                totalYearHeight + titleHeight + startY + 89 - 40 * l - 12
              )
            }
            ctx.moveTo(lineX1, lineY)
            ctx.lineTo(lineX2, lineY)
            ctx.stroke()
          }
          //竖线
          ctx.beginPath()
          ctx.moveTo(totalwidth - 90 + startX + 0.5, totalYearHeight + titleHeight + startY + 49 - 40 * 9 + 0.5)
          ctx.lineTo(totalwidth - 90 + startX + 0.5, totalYearHeight + titleHeight + startY + 89 - 40 * 0 + 0.5)
          ctx.stroke()
          //最上面横线
          ctx.beginPath()
          ctx.moveTo(totalwidth - 180 + startX, totalYearHeight + titleHeight + startY + 89 - 35 - 40 * 10 + 0.5)
          ctx.lineTo(totalwidth + startX, totalYearHeight + titleHeight + startY + 89 - 35 - 40 * 10 + 0.5)
          ctx.stroke()
          // ctx.font = '14px Microsoft Yahei'
          ctx.save()
          ctx.beginPath()
          ctx.font = "12px 新宋体 常规, 新宋体, sans-serif"
          ctx.fillStyle = "#000"
          ctx.fillText(
            "图例",
            totalwidth - 100 + startX,
            totalYearHeight + titleHeight + startY + 89 - 35 - 40 * 10 + 20
          )
          //左三角箭头
          let leftarrowx1 = totalwidth - 160 + startX
          let leftarrowy1 = totalYearHeight + titleHeight + startY - 40 * 9 - 20 + 89
          let middlearrowx2 = totalwidth - 160 + startX + 6
          let middlearrowy2 = leftarrowy1 - 6
          let rightarrowx3 = middlearrowx2
          let rightarrowy3 = leftarrowy1 + 6
          ctx.strokeStyle = "#49fd49"
          ctx.moveTo(leftarrowx1, leftarrowy1)
          ctx.lineTo(middlearrowx2, middlearrowy2)
          ctx.lineTo(rightarrowx3, rightarrowy3)
          ctx.fillStyle = "#49fd49"
          ctx.fill()
          ctx.closePath()
          ctx.stroke()
          //中间线
          ctx.beginPath()
          ctx.moveTo(middlearrowx2, leftarrowy1 + 0.5)
          ctx.lineTo(middlearrowx2 + 40, leftarrowy1 + 0.5)
          ctx.stroke()
          //右三角箭头
          ctx.beginPath()
          ctx.strokeStyle = "#49fd49"
          ctx.moveTo(middlearrowx2 + 40, middlearrowy2)
          ctx.lineTo(middlearrowx2 + 40 + 6, leftarrowy1)
          ctx.lineTo(middlearrowx2 + 40, rightarrowy3)
          ctx.fillStyle = "#49fd49"
          ctx.fill()
          ctx.closePath()
          ctx.stroke()
          ctx.restore()
          //梁场线段
          ctx.beginPath()
          ctx.strokeStyle = "#000000"
          ctx.lineWidth = 1
          ctx.moveTo(leftarrowx1 + 0.5, leftarrowy1 + 40 - 5)
          ctx.lineTo(leftarrowx1 + 0.5, leftarrowy1 + 40 + 5)
          ctx.stroke()
          ctx.moveTo(leftarrowx1 + 52 + 0.5, leftarrowy1 + 40 - 5)
          ctx.lineTo(leftarrowx1 + 52 + 0.5, leftarrowy1 + 40 + 5)
          ctx.stroke()
          ctx.beginPath()
          ctx.lineWidth = 2
          ctx.moveTo(leftarrowx1, leftarrowy1 + 40)
          ctx.lineTo(leftarrowx1 + 40 + 12, leftarrowy1 + 40)
          ctx.stroke()
          //蓝色线
          ctx.beginPath()
          ctx.lineWidth = 1
          ctx.strokeStyle = "#108de9"
          ctx.moveTo(leftarrowx1, leftarrowy1 + 40 * 2 + 0.5)
          ctx.lineTo(leftarrowx1 + 52, leftarrowy1 + 40 * 2 + 0.5)
          ctx.stroke()
          //橙色线
          ctx.beginPath()
          ctx.strokeStyle = "#f59a23"
          ctx.moveTo(leftarrowx1, leftarrowy1 + 40 * 3 + 0.5)
          ctx.lineTo(leftarrowx1 + 52, leftarrowy1 + 40 * 3 + 0.5)
          ctx.stroke()
          //红色线 #ec808d
          ctx.beginPath()
          ctx.strokeStyle = "#ec808d"
          ctx.moveTo(leftarrowx1, leftarrowy1 + 40 * 4 + 0.5)
          ctx.lineTo(leftarrowx1 + 52, leftarrowy1 + 40 * 4 + 0.5)
          ctx.stroke()
          ctx.restore()
          //架梁工程
          for (var mall = 0; mall < 5; mall++) {
            let x = leftarrowx1 + 2 + mall * 12
            let y = leftarrowy1 + 40 * 5 + 0.5
            ctx.save()
            ctx.beginPath()
            ctx.fillStyle = "#000"
            ctx.arc(x, y, 2, (0 * Math.PI) / 180, (360 * Math.PI) / 180, true)
            ctx.closePath()
            ctx.fill()
            ctx.stroke()
          }
          ctx.restore()
          ctx.beginPath()
          ctx.strokeStyle = "#ccc"
          ctx.lineWidth = 3
          ctx.moveTo(leftarrowx1, leftarrowy1 + 40 * 5 + 0.5)
          ctx.lineTo(leftarrowx1 + 52, leftarrowy1 + 40 * 5 + 0.5)
          ctx.stroke()
          ctx.restore()
          //架梁正常
          for (var vv = 0; vv < 5; vv++) {
            let x = leftarrowx1 + 2 + vv * 12
            let y = leftarrowy1 + 40 * 6 + 0.5
            ctx.save()
            ctx.beginPath()
            ctx.fillStyle = "#000"
            ctx.arc(x, y, 2, (0 * Math.PI) / 180, (360 * Math.PI) / 180, true)
            ctx.closePath()
            ctx.fill()
            ctx.stroke()
          }
          ctx.restore()
          ctx.beginPath()
          ctx.strokeStyle = "#ccc"
          ctx.lineWidth = 3
          ctx.moveTo(leftarrowx1, leftarrowy1 + 40 * 6 + 0.5)
          ctx.lineTo(leftarrowx1 + 52, leftarrowy1 + 40 * 6 + 0.5)
          ctx.stroke()
          ctx.beginPath()
          ctx.strokeStyle = "#49fd49"
          ctx.moveTo(leftarrowx1, leftarrowy1 + 40 * 6 + 0.5)
          ctx.lineTo(leftarrowx1 + 27, leftarrowy1 + 40 * 6 + 0.5)
          ctx.stroke()
          ctx.restore()
          //架梁之后
          for (var mal = 0; mal < 5; mal++) {
            let x = leftarrowx1 + 2 + mal * 12
            let y = leftarrowy1 + 40 * 7 + 0.5
            ctx.save()
            ctx.beginPath()
            ctx.fillStyle = "#000"
            ctx.arc(x, y, 2, (0 * Math.PI) / 180, (360 * Math.PI) / 180, true)
            ctx.closePath()
            ctx.fill()
            ctx.stroke()
          }
          ctx.restore()
          ctx.beginPath()
          ctx.strokeStyle = "#ccc"
          ctx.lineWidth = 3
          ctx.moveTo(leftarrowx1, leftarrowy1 + 40 * 7 + 0.5)
          ctx.lineTo(leftarrowx1 + 52, leftarrowy1 + 40 * 7 + 0.5)
          ctx.stroke()
          ctx.beginPath()
          ctx.strokeStyle = "orange"
          ctx.moveTo(leftarrowx1, leftarrowy1 + 40 * 7 + 0.5)
          ctx.lineTo(leftarrowx1 + 27, leftarrowy1 + 40 * 7 + 0.5)
          ctx.stroke()
          ctx.restore()
          //架梁完工
          for (var ll = 0; ll < 5; ll++) {
            let x = leftarrowx1 + 2 + ll * 12
            let y = leftarrowy1 + 40 * 8 + 0.5
            ctx.save()
            ctx.beginPath()
            ctx.fillStyle = "#000"
            ctx.arc(x, y, 2, (0 * Math.PI) / 180, (360 * Math.PI) / 180, true)
            ctx.closePath()
            ctx.fill()
            ctx.stroke()
          }
          ctx.restore()
          ctx.beginPath()
          ctx.strokeStyle = "#49fd49"
          ctx.lineWidth = 3
          ctx.moveTo(leftarrowx1, leftarrowy1 + 40 * 8 + 0.5)
          ctx.lineTo(leftarrowx1 + 52, leftarrowy1 + 40 * 8 + 0.5)
          ctx.stroke()
          ctx.restore()
          //架字
          this.circleText(ctx, totalwidth - 133 + startX, leftarrowy1 + 40 * 9 + 1)
          //图例结束
        }

        //折线部分
        let sectionstartMile = this.canvasData[0].kskm //标段开始里程
        let sectionendMile = this.canvasData[0].jskm //标段结束里程
        let color = [] //底线颜色，正常为灰色，滞后为橙色
        this.iconFontPlace(
          ctx,
          quarternum,
          titleHeight,
          startX,
          startY,
          upYearHeight,
          middleYearHeight,
          sectionstartMile,
          sectionendMile,
          totalwidth,
          color,
          true
        )
        this.iconFontPlace(
          ctx,
          quarternum,
          titleHeight,
          startX,
          startY,
          upYearHeight,
          middleYearHeight,
          sectionstartMile,
          sectionendMile,
          totalwidth,
          color,
          false
        )
      }
    },
    //月份文字填充
    fillMonthText(ctx, quarternum, c, startX, titleHeight, startY, upYearHeight, middleYearHeight) {
      if (c % 12 == 1) {
        ctx.fillText(
          quarternum * 3 - c,
          175 + startX,
          upYearHeight + middleYearHeight + 15 * (c + 1) + titleHeight + startY - 2
        )
        ctx.fill()
      } else if (c % 12 == 2) {
        ctx.fillText(
          quarternum * 3 - c,
          175 + startX,
          upYearHeight + middleYearHeight + 15 * (c + 1) + titleHeight + startY - 2
        )
        ctx.fill()
      } else if (c % 12 == 3) {
        ctx.fillText(
          quarternum * 3 - c,
          175 + startX,
          upYearHeight + middleYearHeight + 15 * (c + 1) + titleHeight + startY - 2
        )
        ctx.fill()
      } else if (c % 12 == 4) {
        ctx.fillText(
          quarternum * 3 - c,
          175 + startX,
          upYearHeight + middleYearHeight + 15 * (c + 1) + titleHeight + startY - 2
        )
        ctx.fill()
      } else if (c % 12 == 5) {
        ctx.fillText(
          quarternum * 3 - c,
          175 + startX,
          upYearHeight + middleYearHeight + 15 * (c + 1) + titleHeight + startY - 2
        )
        ctx.fill()
      } else if (c % 12 == 6) {
        ctx.fillText(
          quarternum * 3 - c,
          175 + startX,
          upYearHeight + middleYearHeight + 15 * (c + 1) + titleHeight + startY - 2
        )
        ctx.fill()
      } else if (c % 12 == 7) {
        ctx.fillText(
          quarternum * 3 - c,
          175 + startX,
          upYearHeight + middleYearHeight + 15 * (c + 1) + titleHeight + startY - 2
        )
        ctx.fill()
      } else if (c % 12 == 8) {
        ctx.fillText(
          quarternum * 3 - c,
          175 + startX,
          upYearHeight + middleYearHeight + 15 * (c + 1) + titleHeight + startY - 2
        )
        ctx.fill()
      } else if (c % 12 == 9) {
        ctx.fillText(
          quarternum * 3 - c,
          175 + startX,
          upYearHeight + middleYearHeight + 15 * (c + 1) + titleHeight + startY - 2
        )
        ctx.fill()
      } else if (c % 12 == 10) {
        ctx.fillText(
          quarternum * 3 - c,
          175 + startX,
          upYearHeight + middleYearHeight + 15 * (c + 1) + titleHeight + startY - 2
        )
        ctx.fill()
      } else if (c % 12 == 11) {
        ctx.fillText(
          quarternum * 3 - c,
          175 + startX,
          upYearHeight + middleYearHeight + 15 * (c + 1) + titleHeight + startY - 2
        )
        ctx.fill()
      } else if (c % 12 == 0) {
        ctx.fillText(
          quarternum * 3 - c,
          175 + startX,
          upYearHeight + middleYearHeight + 15 * (c + 1) + titleHeight + startY - 2
        )
        ctx.fill()
      }
    },
    downfillMonthText(ctx, c, startX, titleHeight, startY, upYearHeight, middleYearHeight) {
      if (c % 12 == 1) {
        ctx.fillText(12 - c, 175 + startX, upYearHeight + middleYearHeight + 15 * (c + 1) + titleHeight + startY - 2)
        ctx.fill()
      } else if (c % 12 == 2) {
        ctx.fillText(12 - c, 175 + startX, upYearHeight + middleYearHeight + 15 * (c + 1) + titleHeight + startY - 2)
        ctx.fill()
      } else if (c % 12 == 3) {
        ctx.fillText(12 - c, 175 + startX, upYearHeight + middleYearHeight + 15 * (c + 1) + titleHeight + startY - 2)
        ctx.fill()
      } else if (c % 12 == 4) {
        ctx.fillText(12 - c, 175 + startX, upYearHeight + middleYearHeight + 15 * (c + 1) + titleHeight + startY - 2)
        ctx.fill()
      } else if (c % 12 == 5) {
        ctx.fillText(12 - c, 175 + startX, upYearHeight + middleYearHeight + 15 * (c + 1) + titleHeight + startY - 2)
        ctx.fill()
      } else if (c % 12 == 6) {
        ctx.fillText(12 - c, 175 + startX, upYearHeight + middleYearHeight + 15 * (c + 1) + titleHeight + startY - 2)
        ctx.fill()
      } else if (c % 12 == 7) {
        ctx.fillText(12 - c, 175 + startX, upYearHeight + middleYearHeight + 15 * (c + 1) + titleHeight + startY - 2)
        ctx.fill()
      } else if (c % 12 == 8) {
        ctx.fillText(12 - c, 175 + startX, upYearHeight + middleYearHeight + 15 * (c + 1) + titleHeight + startY - 2)
        ctx.fill()
      } else if (c % 12 == 9) {
        ctx.fillText(12 - c, 175 + startX, upYearHeight + middleYearHeight + 15 * (c + 1) + titleHeight + startY - 2)
        ctx.fill()
      } else if (c % 12 == 10) {
        ctx.fillText(12 - c, 175 + startX, upYearHeight + middleYearHeight + 15 * (c + 1) + titleHeight + startY - 2)
        ctx.fill()
      } else if (c % 12 == 11) {
        ctx.fillText(12 - c, 175 + startX, upYearHeight + middleYearHeight + 15 * (c + 1) + titleHeight + startY - 2)
        ctx.fill()
      } else if (c % 12 == 0) {
        ctx.fillText(12 - c, 175 + startX, upYearHeight + middleYearHeight + 15 * (c + 1) + titleHeight + startY - 2)
        ctx.fill()
      }
    },
    //最下面月份分割指示线
    MonthdashLine(c, ctx, line, startX, startY, titleHeight, upYearHeight, middleYearHeight, quarternum, dashWidth) {
      if (c > 0 && c < quarternum * 3) {
        ctx.fillStyle = "#ccc"
        ctx.font = "10px Microsoft Yahei"
        ctx.fillText(
          c + 1,
          200 + startX + dashWidth * (line + 1) + 12 * line,
          upYearHeight + middleYearHeight + titleHeight + startY + 15 * (quarternum * 3 - c) + 0.5
        )
        ctx.fill()
      }
    },
    downMonthdashLine(
      c,
      ctx,
      line,
      startX,
      startY,
      titleHeight,
      upYearHeight,
      middleYearHeight,
      quarternum,
      dashWidth
    ) {
      if (c > 0 && c < quarternum * 3) {
        ctx.fillStyle = "#ccc"
        ctx.font = "10px Microsoft Yahei"
        ctx.fillText(
          12 - c + 1,
          200 + startX + dashWidth * (line + 1) + 12 * line,
          upYearHeight + middleYearHeight + titleHeight + startY + 15 * c + 0.5
        )
        ctx.fill()
      }
    },
    //计算三条实线Y坐标
    markLine(year, lastYear, downYear, month, date, startY, titleHeight) {
      if (year === lastYear) {
        //最上面
        return (this.quarternum * 3 - month + (31 - date) / 31) * 15 + startY + titleHeight
      } else if (year < lastYear && year > downYear) {
        //中间
        let middleheight = (lastYear - year - 1 + 12 - month + (31 - date) / 31) * 15
        return this.upYearHeight + middleheight + startY + titleHeight
      } else if (year === downYear) {
        //最下面
        let downheight = (12 - month + (31 - date) / 31) * 15
        return this.upYearHeight + this.middleYearHeight + downheight + startY + titleHeight
      }
    },
    toPdfAndDownload(totalWidth, totalHeight) {
      // let uuid = this.uuid()
      // 截图区域是testCanvas
      html2canvas(document.getElementById("grideCanvas"), {
        background: "#ffffff",
        dpi: 350, //导出pdf清晰度
        // scale:window.devicePixelRatio * 2, // 提升画质质量，但是会增加文件大小,
        scale: 2,
        width: totalWidth,
        height: totalHeight,
        useCORS: true, // 如果截图的内容里有图片,可能会有跨域的情况,加上这个参数,解决文件跨域问题
        allowTaint: true, //允许跨域（图片跨域相关）
        // taintTest: true,              //是否在渲染前测试图片
      }).then(function (canvas) {
        // let contentWidth = canvas.width;
        // let contentHeight = canvas.height;
        // // let imgWidth = 595.28;
        // let imgWidth = 841.89;
        // // let imgHeight = 592.28 / contentWidth * contentHeight;
        // let imgHeight = 841.89 / contentWidth * contentHeight;
        // let pageData = canvas.toDataURL('image/jpeg', 1.0);
        // let pdf = new jsPDF('', 'pt', 'a3');
        // pdf.addImage(pageData, 'JPEG', 0, 0, imgWidth, imgHeight)
        // pdf.save( '**************.pdf');
        console.log(totalWidth, totalHeight, "width,height")
        let contentWidth = canvas.width
        let contentHeight = canvas.height
        let pageHeight = (contentWidth / 841.89) * 841.89
        let leftHeight = contentHeight
        let position = 0
        let imgWidth = 841.89
        let imgHeight = (841.89 / contentWidth) * contentHeight
        let pageData = canvas.toDataURL("image/jpeg", 1.0)
        let PDF = new jsPDF("", "pt", "a3")
        if (leftHeight < pageHeight) {
          PDF.addImage(pageData, "JPEG", 0, 0, imgWidth, imgHeight)
        } else {
          while (leftHeight > 0) {
            PDF.addImage(pageData, "JPEG", 0, position, imgWidth, imgHeight)
            leftHeight -= pageHeight
            position -= 841.89
            if (leftHeight > 0) {
              PDF.addPage()
            }
          }
        }
        PDF.save("******" + ".pdf")
        // let newImg = new Image()
        // newImg.setAttribute("crossOrigin", 'Anonymous')
        // newImg.src = canvas.toDataURL("image/png")
        // let a = document.createElement('a')
        // document.body.appendChild(a)
        // a.href = newImg.src
        // a.download = '一个小例子'
        // a.click()
        // document.body.removeChild(a)
      })
    },
    exportCanvas() {
      let canvas = document.getElementById("grideCanvas")
      const clientWidth = canvas.offsetWidth
      const clientHeight = canvas.offsetHeight
      const kh = [clientWidth, clientHeight]
      html2canvas(canvas, { useCORS: true, logging: true }).then((canvas) => {
        const dataURL = canvas.toDataURL("image/png")
        this.download(dataURL, kh)
      })
    },
    getUrlBase64(url, kh) {
      return new Promise((resolve) => {
        let canvas = document.createElement("canvas")
        const ctx = canvas.getContext("2d")
        const img = new Image()
        img.crossOrigin = "Anonymous"
        img.src = url
        img.onload = () => {
          canvas.height = kh[1]
          canvas.width = kh[0]
          ctx.drawImage(img, 0, 0, kh[0], kh[1])
          const dataURL = canvas.toDataURL("image/png")
          canvas = null
          resolve(dataURL)
        }
      })
    },
    download(imgUrl, kh) {
      this.getUrlBase64(imgUrl, kh).then((base64) => {
        const link = document.createElement("a")
        link.href = base64
        link.download = this.titleName + "架梁进度形象图.png"
        link.click()
      })
    },
    //刻度计算
    find(num) {
      for (let i = num; ; i++) {
        if (i % 5 === 0) {
          return i
        }
      }
    },
    // 竖排文字处理
    writeTextOnCanvas(ctx, x, y, name) {
      // name; // 文本内容
      // x,y // 文字开始的坐标
      let letterSpacing = 2 // 设置字间距
      // console.log(this.getReverse(name),'取反')
      if (name && name.length > 0) {
        for (let i = 0; i < name.length; i++) {
          const str = this.getReverse(name)
            .slice(i, i + 1)
            .toString()
          // const str =name.slice(i,i+1).toString();
          if (str.match(/[A-Za-z0-9]/)) {
            // 非汉字 旋转
            ctx.save()
            ctx.translate(x, y)
            // ctx.rotate(Math.PI/180*90);
            ctx.rotate((-Math.PI / 180) * 90)
            ctx.textBaseline = "bottom"
            ctx.fillText(str, 0, 0)
            ctx.restore()
            y += ctx.measureText(str).width + letterSpacing // 计算文字宽度
          } else if (str.match(/[\u4E00-\u9FA5]/)) {
            ctx.save()
            ctx.textBaseline = "top"
            ctx.fillText(str, x, y)
            ctx.restore()
            y += ctx.measureText(str).width + letterSpacing // 计算文字宽度
          }
        }
      }
    },
    getReverse(str) {
      let newStr = str.split("").reverse().join("")
      return newStr
    },
    //计算直角三角形斜边长度
    findHypotenuse(base, perpendicular) {
      const bSquare = base ** 2
      const pSquare = perpendicular ** 2
      const sum = bSquare + pSquare
      const hypotenuse = Math.sqrt(sum)
      return hypotenuse
    },
    //圆圈带字
    circleText(ctx, x, y) {
      var image = new Image()
      image.src = this.src
      ctx.save()
      ctx.beginPath()
      ctx.strokeStyle = "#33ccff"
      ctx.lineWidth = 1
      ctx.fillStyle = "rgba(255,255,255,0.9)"
      ctx.arc(x, y, 10, 0, 2 * Math.PI, true)
      ctx.fill()
      // ctx.stroke()
      // ctx.fillStyle = '#333'
      // ctx.font = '12px Microsoft Yahei'
      // ctx.fillText(font,x-6,y+5)
      ctx.beginPath()
      if (image.complete) {
        ctx.drawImage(image, x - 10, y - 10, 20, 20)
      } else {
        image.onload = function () {
          ctx.drawImage(image, x - 10, y - 10, 20, 20)
        }
      }
      ctx.restore()
    },

    //折线
    drawrollLine(ctx, titleHeight, startX, startY, totalwidth, x, y) {
      if (this.canvasData.length > 0) {
        let sectionstartMile = this.canvasData[0].kskm //标段开始里程
        let sectionendMile = this.canvasData[0].jskm //标段结束里程
        // let color =[]  //底线颜色，正常为灰色，滞后为橙色
        for (var v = 0; v < this.canvasData.length; v++) {
          let ifStart = this.canvasData[v].sjks
          let progress = this.canvasData[v].zt //进度 正常/滞后
          let color = []
          let progressplan = this.canvasData[v].percent.split("%")[0] / 100
          //里程方向
          let mileDirection = this.canvasData[v].location
          //是否开始施工 为null表示未开始
          let planStart = this.canvasData[v].start_time //计划开始时间
          let planEnd = this.canvasData[v].end_time //计划结束时间
          let actualStart = this.canvasData[v].sjks //实际开始时间
          // let actualEnd = this.canvasData[v].sjwc       //实际结束时间
          let planStartMile = this.canvasData[v].start //该计划开始里程
          let planEndMile = this.canvasData[v].end //该计划结束里程
          let lastTime = this.canvasData[0].tyDate.split("-")[0] //标段推演完成时间，即最晚时间
          //纵坐标时间获取
          let planUsingtime1 = mileDirection == "小里程" ? planEnd : planStart //该计划的计划时间第一个点 Y1
          let planUsingtime2 = mileDirection == "小里程" ? planStart : planEnd //该计划的计划时间第二个点 Y2
          //计算折线Y坐标
          //计划灰色部分折线Y坐标
          let monthy = Number(planUsingtime1.split("-")[1]) //第一个点多少个月
          let monthy2 = Number(planUsingtime2.split("-")[1]) //第二个点多少个月
          let datey = Number(planUsingtime1.split("-")[2]) //多少天
          let datey2 = Number(planUsingtime2.split("-")[2]) //第二个点多少天
          let totalMonths = 0 //当前年份区段所占月份总数
          let totalDates = 0 //当前年份区段所占天数总数
          let timeY_plan = null //第一个点纵坐标
          let totalMonths2 = 0
          let totalDates2 = 0
          let timeY_plan2 = null
          let earilyYear = this.canvasData[0].earlyTime.split("-")[0]
          // console.log(upYearHeight,middleYearHeight,'upheight,middleheight======')
          if (Number(lastTime) === Number(planUsingtime1.split("-")[0])) {
            //年份在最上面
            totalMonths = this.quarternum * 3 - monthy
            totalDates = 31 - datey
            timeY_plan = titleHeight + startY + (totalMonths + totalDates / 31) * 15 //总的月份乘以单月高度15
          } else if (
            Number(planUsingtime1.split("-")[0]) < Number(lastTime) &&
            Number(planUsingtime1.split("-")[0]) > Number(earilyYear)
          ) {
            //处于中间年份计算坐标
            totalDates = 31 - datey
            totalMonths =
              (Number(lastTime) - Number(planUsingtime1.split("-")[0]) - 1) * 12 +
              12 -
              Number(planUsingtime1.split("-")[1])
            timeY_plan = titleHeight + startY + this.upYearHeight + (totalMonths + totalDates / 31) * 15
          } else if (Number(planUsingtime1.split("-")[0]) === Number(earilyYear)) {
            //位于最下面的年份
            totalDates = 31 - datey
            totalMonths = 12 - monthy
            timeY_plan =
              this.upYearHeight + this.middleYearHeight + titleHeight + startY + (totalMonths + totalDates / 31) * 15 //总的月份乘以单月高度15
          }

          if (Number(lastTime) === Number(planUsingtime2.split("-")[0])) {
            //年份在最上面
            totalMonths2 = this.quarternum * 3 - monthy2
            totalDates2 = 31 - datey2
            timeY_plan2 = titleHeight + startY + (totalMonths2 + totalDates2 / 31) * 15 //总的月份乘以单月高度15
          } else if (
            Number(planUsingtime2.split("-")[0]) < Number(lastTime) &&
            Number(planUsingtime2.split("-")[0]) > Number(earilyYear)
          ) {
            //处于中间年份计算坐标
            totalDates2 = 31 - datey2
            totalMonths2 =
              (Number(lastTime) - Number(planUsingtime2.split("-")[0]) - 1) * 12 +
              12 -
              Number(planUsingtime2.split("-")[1])
            timeY_plan2 = titleHeight + startY + this.upYearHeight + (totalMonths2 + totalDates2 / 31) * 15
          } else if (Number(planUsingtime2.split("-")[0]) === Number(earilyYear)) {
            //位于最下面的年份
            totalDates2 = 31 - datey2
            totalMonths2 = 12 - monthy2
            timeY_plan2 =
              this.upYearHeight + this.middleYearHeight + titleHeight + startY + (totalMonths2 + totalDates2 / 31) * 15 //总的月份乘以单月高度15
          }
          //计算折线X坐标
          let startPlanX = null //折线第一个X坐标
          let startPlanX2 = null //折线第二个X坐标
          if (mileDirection == "小里程") {
            //小里程方向，结束里程小，开始里程大
            startPlanX =
              ((planEndMile - sectionstartMile) / (sectionendMile - sectionstartMile)) *
                (totalwidth - 200 - this.legendWidth) +
              200 +
              startX
            startPlanX2 =
              ((planStartMile - sectionstartMile) / (sectionendMile - sectionstartMile)) *
                (totalwidth - 200 - this.legendWidth) +
              200 +
              startX
          } else {
            //大里程方向
            startPlanX =
              ((planStartMile - sectionstartMile) / (sectionendMile - sectionstartMile)) *
                (totalwidth - 200 - this.legendWidth) +
              200 +
              startX
            startPlanX2 =
              ((planEndMile - sectionstartMile) / (sectionendMile - sectionstartMile)) *
                (totalwidth - 200 - this.legendWidth) +
              200 +
              startX
          }
          this.caseInfo.name = this.canvasData[v].gzw
          this.caseInfo.construction = this.canvasData[v].jd
          this.caseInfo.equipment = this.canvasData[v].ylname
          this.caseInfo.progress = this.canvasData[v].percent
          this.caseInfo.rate = this.canvasData[v].zt
          this.caseInfo.workNum = this.canvasData[v].jhjlnum
          this.caseInfo.startMile = this.canvasData[v].kslc
          this.caseInfo.endMile = this.canvasData[v].jslc
          this.caseInfo.planStarttime = this.canvasData[v].start_time
          this.caseInfo.actualStarttime = actualStart
          this.caseInfo.planfinishtime = this.canvasData[v].end_time
          this.caseInfo.actualfinishtime = this.canvasData[v].sjwc

          //灰色折线
          ctx.save()
          ctx.beginPath()
          ctx.lineWidth = 1
          ctx.strokeStyle = "red"
          if (mileDirection == "大里程") {
            ctx.moveTo(parseInt(startPlanX) - 4.5, parseInt(timeY_plan))
            ctx.lineTo(parseInt(startPlanX2) - 4.5, parseInt(timeY_plan2))
            ctx.lineTo(parseInt(startPlanX2) + 4.5, parseInt(timeY_plan2) + 3)
            ctx.lineTo(parseInt(startPlanX) + 4.5, parseInt(timeY_plan) + 3)
            ctx.closePath()
            ctx.restore()
            this.at = ctx.isPointInPath(x, y)
            if (this.at == true) {
              this.left1 = x + startX
              this.top1 = y + startY + 30

              return false
            }
          } else {
            ctx.moveTo(parseInt(startPlanX) + 4.5, parseInt(timeY_plan))
            ctx.lineTo(parseInt(startPlanX2) + 4.5, parseInt(timeY_plan2))
            ctx.lineTo(parseInt(startPlanX2) - 4.5, parseInt(timeY_plan2) + 3)
            ctx.lineTo(parseInt(startPlanX) - 4.5, parseInt(timeY_plan) + 3)
            ctx.closePath()
            // ctx.stroke()
            ctx.restore()

            this.at2 = ctx.isPointInPath(x, y)
            if (this.at2 == true) {
              this.left2 = x + startX - 100
              this.top2 = y + startY + 30
              return false
            }
          }
          //绿色折线坐标
          let timeY_actual = null
          let timeY_actual2 = null
          let actualMileX1 = null
          let actualMileX2 = null
          //判断是否已开始施工
          if (ifStart != null) {
            //已施工
            if (progress == "正常") {
              color.push("#ccc")
            } else {
              color.push("orange")
            }
            if (mileDirection == "大里程") {
              actualMileX1 = startPlanX
              actualMileX2 = (startPlanX2 - startPlanX) * progressplan + startPlanX
              timeY_actual = timeY_plan
              timeY_actual2 = timeY_plan - (timeY_plan - timeY_plan2) * progressplan
            } else {
              actualMileX1 = startPlanX2 - (startPlanX2 - startPlanX) * progressplan
              actualMileX2 = startPlanX2
              timeY_actual = timeY_plan2 - (timeY_plan2 - timeY_plan) * progressplan
              timeY_actual2 = timeY_plan2
            }
          } else {
            //未施工
            color.push("#ccc")
          }
          // 架梁设备图标
          if (ifStart != null) {
            // 架梁设备名称
            let equipname = this.canvasData[v].jqname
            let namewidth = equipname.length * 10
            if (this.checked && this.canvasData[v].sbState == 1) {
              this.equipmentName = equipname
              let flag = false
              ctx.save()
              ctx.beginPath()
              ctx.lineWidth = 1
              if (mileDirection == "大里程") {
                let x2 = parseInt(actualMileX2) + 6 + 0.5
                let y2 = parseInt(timeY_actual2) - 6
                let xx2 = Math.abs(x - x2)
                let yy2 = Math.abs(y - y2)
                let l = this.findHypotenuse(xx2, yy2)
                console.log(l, "lllllllllllll")
                if (l < 10) {
                  ctx.save()
                  ctx.beginPath()
                  ctx.fillStyle = "#fff"
                  ctx.fillRect(parseInt(actualMileX2) + 4 + 0.5, parseInt(timeY_actual2) - 42 + 0.5, namewidth, 22)
                  ctx.strokeStyle = "#333"
                  ctx.strokeRect(parseInt(actualMileX2) + 4 + 0.5, parseInt(timeY_actual2) - 42 + 0.5, namewidth, 22)
                  ctx.fillStyle = "#333"
                  ctx.fillText(equipname, parseInt(actualMileX2) + 10 + 0.5, parseInt(timeY_actual2) - 27 + 0.5)
                  ctx.restore()
                  return false
                } else {
                  if (!flag) {
                    ctx.clearRect(0, 0, totalwidth + 1, 150 + this.totalYearHeight + 1)
                    this.draw(ctx, totalwidth, titleHeight, this.yearData, this.startX, this.startY)
                    flag = true
                    console.log("outda-----")
                  }
                }
              } else {
                // this.circleText(ctx,parseInt(actualMileX1)+0.5-6,parseInt(timeY_actual)-6,'架')
                let x1 = parseInt(actualMileX1) + 6 + 0.5
                let y1 = parseInt(timeY_actual) - 6
                let xx1 = Math.abs(x - x1)
                let yy1 = Math.abs(y - y1)
                let L1 = this.findHypotenuse(xx1, yy1)
                console.log(L1, "lllllLLLLLLLLL999llllllll")
                if (L1 < 10) {
                  ctx.save()
                  ctx.beginPath()
                  ctx.fillStyle = "#fff"
                  ctx.fillRect(parseInt(actualMileX1) - 10 + 0.5, parseInt(timeY_actual) - 42 + 0.5, namewidth, 22)
                  ctx.strokeStyle = "#333"
                  ctx.strokeRect(parseInt(actualMileX1) - 10 + 0.5, parseInt(timeY_actual) - 42 + 0.5, namewidth, 22)
                  ctx.fillStyle = "#333"
                  ctx.fillText(equipname, parseInt(actualMileX1) - 6 + 0.5, parseInt(timeY_actual) - 27 + 0.5)
                  ctx.restore()
                  return false
                } else {
                  if (!flag) {
                    ctx.clearRect(0, 0, totalwidth + 1, 150 + this.totalYearHeight + 1)
                    this.draw(ctx, totalwidth, titleHeight, this.yearData, this.startX, this.startY)
                    flag = true
                    console.log("out1-----")
                  }
                }
              }
            }
          }
        }
      }
    },
    //放大缩小
    getLarger(canvas, canvasInfo, ctx, totalwidth, titleHeight, yearHeight, bigorsmall) {
      console.log(yearHeight, "yearHeight---")
      const { scaleStep } = canvasInfo
      this.$set(this.realCanvasPosition, "x", parseInt(canvas.width / 2))
      this.$set(this.realCanvasPosition, "y", parseInt((titleHeight + 80 + this.startY + yearHeight) / 2))
      let defaultscale = 1
      const deltaX = (this.realCanvasPosition.x / defaultscale) * scaleStep
      const deltaY = (this.realCanvasPosition.y / defaultscale) * scaleStep
      ctx.clearRect(0, 0, canvas.width, canvas.height)
      let canvaswidth = canvas.width
      canvas.width = canvaswidth
      if (bigorsmall) {
        //放大
        canvasInfo.offset.x -= deltaX
        canvasInfo.offset.y -= deltaY
        canvasInfo.scale += scaleStep
      } else {
        //缩小
        canvasInfo.offset.x += deltaX
        canvasInfo.offset.y += deltaY
        canvasInfo.scale -= scaleStep
      }
      ctx.setTransform(canvasInfo.scale, 0, 0, canvasInfo.scale, canvasInfo.offset.x, canvasInfo.offset.y)
      this.draw(ctx, totalwidth, titleHeight, this.yearData, this.startX, this.startY)
    },

    //折线部分坐标及运架设备图标字体计算
    iconFontPlace(
      ctx,
      quarternum,
      titleHeight,
      startX,
      startY,
      upYearHeight,
      middleYearHeight,
      sectionstartMile,
      sectionendMile,
      totalwidth,
      color,
      is
    ) {
      for (var v = 0; v < this.canvasData.length; v++) {
        //里程方向
        let mileDirection = this.canvasData[v].location
        //是否开始施工 为null表示未开始
        let ifStart = this.canvasData[v].sjks
        let planStart = this.canvasData[v].start_time //计划开始时间
        let planEnd = this.canvasData[v].end_time //计划结束时间
        // let actualStart = this.canvasData[v].sjks       //实际开始时间
        // let actualEnd = this.canvasData[v].sjwc       //实际结束时间
        let planStartMile = this.canvasData[v].start //该计划开始里程
        let planEndMile = this.canvasData[v].end //该计划结束里程
        let lastTime = this.canvasData[0].tyDate.split("-")[0] //标段推演完成时间，即最晚时间
        let progress = this.canvasData[v].zt //进度 正常/滞后
        let progressplan = this.canvasData[v].percent.split("%")[0] / 100
        //纵坐标时间获取
        let planUsingtime1 = mileDirection == "小里程" ? planEnd : planStart //该计划的计划时间第一个点 Y1
        let planUsingtime2 = mileDirection == "小里程" ? planStart : planEnd //该计划的计划时间第二个点 Y2
        //计算折线Y坐标
        //计划灰色部分折线Y坐标
        let monthy = Number(planUsingtime1.split("-")[1]) //第一个点多少个月
        let monthy2 = Number(planUsingtime2.split("-")[1]) //第二个点多少个月
        let datey = Number(planUsingtime1.split("-")[2]) //多少天
        let datey2 = Number(planUsingtime2.split("-")[2]) //第二个点多少天
        let totalMonths = 0 //当前年份区段所占月份总数
        let totalDates = 0 //当前年份区段所占天数总数
        let timeY_plan = null //第一个点纵坐标
        let totalMonths2 = 0
        let totalDates2 = 0
        let timeY_plan2 = null
        let earilyYear = this.canvasData[0].earlyTime.split("-")[0]
        if (Number(lastTime) === Number(planUsingtime1.split("-")[0])) {
          //年份在最上面
          totalMonths = quarternum * 3 - monthy
          totalDates = 31 - datey
          timeY_plan = titleHeight + startY + (totalMonths + totalDates / 31) * 15 //总的月份乘以单月高度15
        } else if (
          Number(planUsingtime1.split("-")[0]) < Number(lastTime) &&
          Number(planUsingtime1.split("-")[0]) > Number(earilyYear)
        ) {
          //处于中间年份计算坐标
          totalDates = 31 - datey
          totalMonths =
            (Number(lastTime) - Number(planUsingtime1.split("-")[0]) - 1) * 12 +
            12 -
            Number(planUsingtime1.split("-")[1])
          timeY_plan = titleHeight + startY + upYearHeight + (totalMonths + totalDates / 31) * 15
        } else if (Number(planUsingtime1.split("-")[0]) === Number(earilyYear)) {
          //位于最下面的年份
          totalDates = 31 - datey
          totalMonths = 12 - monthy
          timeY_plan = upYearHeight + middleYearHeight + titleHeight + startY + (totalMonths + totalDates / 31) * 15 //总的月份乘以单月高度15
        }
        if (Number(lastTime) === Number(planUsingtime2.split("-")[0])) {
          //年份在最上面
          totalMonths2 = quarternum * 3 - monthy2
          totalDates2 = 31 - datey2
          timeY_plan2 = titleHeight + startY + (totalMonths2 + totalDates2 / 31) * 15 //总的月份乘以单月高度15
        } else if (
          Number(planUsingtime2.split("-")[0]) < Number(lastTime) &&
          Number(planUsingtime2.split("-")[0]) > Number(earilyYear)
        ) {
          //处于中间年份计算坐标
          totalDates2 = 31 - datey2
          totalMonths2 =
            (Number(lastTime) - Number(planUsingtime2.split("-")[0]) - 1) * 12 +
            12 -
            Number(planUsingtime2.split("-")[1])
          timeY_plan2 = titleHeight + startY + upYearHeight + (totalMonths2 + totalDates2 / 31) * 15
        } else if (Number(planUsingtime2.split("-")[0]) === Number(earilyYear)) {
          //位于最下面的年份
          totalDates2 = 31 - datey2
          totalMonths2 = 12 - monthy2
          timeY_plan2 = upYearHeight + middleYearHeight + titleHeight + startY + (totalMonths2 + totalDates2 / 31) * 15 //总的月份乘以单月高度15
          console.log(totalMonths2, "totalmaonths2-----")
        }
        //计算折线X坐标
        let startPlanX = null //折线第一个X坐标
        let startPlanX2 = null //折线第二个X坐标
        if (mileDirection == "小里程") {
          //小里程方向，结束里程小，开始里程大
          startPlanX =
            ((planEndMile - sectionstartMile) / (sectionendMile - sectionstartMile)) *
              (totalwidth - 200 - this.legendWidth) +
            200 +
            startX
          startPlanX2 =
            ((planStartMile - sectionstartMile) / (sectionendMile - sectionstartMile)) *
              (totalwidth - 200 - this.legendWidth) +
            200 +
            startX
        } else {
          //大里程方向
          startPlanX =
            ((planStartMile - sectionstartMile) / (sectionendMile - sectionstartMile)) *
              (totalwidth - 200 - this.legendWidth) +
            200 +
            startX
          startPlanX2 =
            ((planEndMile - sectionstartMile) / (sectionendMile - sectionstartMile)) *
              (totalwidth - 200 - this.legendWidth) +
            200 +
            startX
        }
        //绿色折线坐标
        let timeY_actual = null
        let timeY_actual2 = null
        let actualMileX1 = null
        let actualMileX2 = null
        //判断是否已开始施工
        if (ifStart != null) {
          //已施工
          if (progress == "正常") {
            color.push("#ccc")
          } else {
            color.push("orange")
          }
          if (mileDirection == "大里程") {
            actualMileX1 = startPlanX
            actualMileX2 = (startPlanX2 - startPlanX) * progressplan + startPlanX
            timeY_actual = timeY_plan
            timeY_actual2 = timeY_plan - (timeY_plan - timeY_plan2) * progressplan
          } else {
            actualMileX1 = startPlanX2 - (startPlanX2 - startPlanX) * progressplan
            actualMileX2 = startPlanX2
            timeY_actual = timeY_plan2 - (timeY_plan2 - timeY_plan) * progressplan
            timeY_actual2 = timeY_plan2
          }
        } else {
          //未施工
          color.push("#ccc")
        }
        if (is) {
          //为true画折线 ，为false画架梁图标，为了防止放在一个循环里图标被折线遮挡，图标放后面画
          //小圆点
          let lineLong = this.findHypotenuse(startPlanX2 - startPlanX, timeY_plan2 - timeY_plan) //计算斜边长度
          // console.log(this.findHypotenuse(6,8),'长度')
          let smallLine = 20 //  小圆点之间线段长度设为20
          let circleNum = parseInt(lineLong / smallLine) //小圆点个数
          let circleX = null
          let circleY = null
          // if(circleNum < 1){
          //     // ctx.save()
          //     ctx.beginPath()
          //     //  circleX =  (startPlanX2 - startPlanX)/2 + startPlanX
          //     //  circleY = timeY_plan - (timeY_plan - timeY_plan2)/2
          //     if(mileDirection == '大里程'){
          //         circleX =   startPlanX
          //         circleY = timeY_plan
          //     }else{
          //         circleX =   startPlanX2
          //         circleY = timeY_plan2
          //     }
          //     ctx.fillStyle ='#999'
          //     ctx.arc(parseInt(circleX)+0.5,parseInt(circleY)-0.5,1,0*Math.PI/180,360*Math.PI/180,true)
          //     ctx.closePath()
          //     ctx.fill()
          //     ctx.stroke()

          // }
          if (circleNum >= 1) {
            for (var u = 0; u < circleNum + 1; u++) {
              ctx.save()
              ctx.beginPath()
              if (mileDirection == "大里程") {
                circleX = ((startPlanX2 - startPlanX) * smallLine * u) / lineLong + startPlanX
                circleY = timeY_plan - (smallLine * u * (timeY_plan - timeY_plan2)) / lineLong
              } else {
                circleX = startPlanX2 - (20 * u * (startPlanX2 - startPlanX)) / lineLong
                circleY = timeY_plan2 - (20 * u * (timeY_plan2 - timeY_plan)) / lineLong
              }
              ctx.fillStyle = "#ccc"
              ctx.arc(
                parseInt(circleX) + 0.5,
                parseInt(circleY) - 0.5,
                1.5,
                (0 * Math.PI) / 180,
                (360 * Math.PI) / 180,
                true
              )
              ctx.closePath()
              ctx.fill()
              ctx.stroke()
              ctx.restore()
            }
          }

          //灰色折线
          ctx.save()
          ctx.beginPath()
          ctx.moveTo(parseInt(startPlanX) + 0.5, parseInt(timeY_plan))
          ctx.lineTo(parseInt(startPlanX2) + 0.5, parseInt(timeY_plan2))
          ctx.lineWidth = 1
          ctx.strokeStyle = color[v]
          ctx.stroke()
          ctx.restore()
          //绿色折线
          if (ifStart != null) {
            ctx.save()
            ctx.beginPath()
            ctx.moveTo(parseInt(actualMileX1) + 0.5, parseInt(timeY_actual))
            ctx.lineTo(parseInt(actualMileX2) + 0.5, parseInt(timeY_actual2))
            ctx.lineWidth = 2
            ctx.strokeStyle = "#49fd49"
            ctx.stroke()
            ctx.restore()
          }
        } else {
          //架梁设备图标
          if (ifStart != null) {
            ctx.save()
            ctx.beginPath()
            // ctx.moveTo(parseInt(actualMileX1)+0.5,parseInt(timeY_actual))
            // ctx.lineTo(parseInt(actualMileX2)+0.5,parseInt(timeY_actual2))
            // ctx.lineWidth = 2
            // ctx.strokeStyle = '#49fd49'
            // ctx.stroke()
            // ctx.restore()
            //架梁设备名称
            // let equipname = this.canvasData[v].jqname
            // let namewidth = equipname.length *10
            if (this.checked && this.canvasData[v].sbState == 1) {
              ctx.save()
              ctx.beginPath()
              ctx.lineWidth = 1
              // let equipname = this.canvasData[v].jqname
              if (mileDirection == "大里程") {
                this.circleText(ctx, parseInt(actualMileX2) + 6 + 0.5, parseInt(timeY_actual2) - 6)
              } else {
                this.circleText(ctx, parseInt(actualMileX1) + 0.5 - 6, parseInt(timeY_actual) - 6)
              }

              ctx.restore()
            }
          }
        }
      }
    },
  },
}
</script>

<style lang="scss" scoped>
#container {
  cursor: pointer;
  position: relative;
  margin-top: 18px;
  #msg1,
  #msg2 {
    position: absolute;
    padding: 10px;
    background-color: rgba(0, 0, 0, 0.6);
    color: #fff;
    font-size: 14px;
    left: 300px;
    top: 100px;
  }
  #equipmentName1,
  #equipmentName2 {
    position: absolute;
    padding: 3px 5px;
    font-size: 12px;
    background-color: #fff;
    border: 1px solid #333;
    z-index: 99999;
  }
  .circle {
    // position:relative;
    .iconxy {
      position: absolute;
      width: 26px;
      height: 26px;
      border-radius: 100%;
      background-color: red;
    }
    span {
      position: absolute;
      background-color: green;
    }
  }
}
</style>
