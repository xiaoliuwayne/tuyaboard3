<template>
  <el-container>
    <el-header class="h-pad h-strong">在线涂鸦工具</el-header>
      <el-main id="main-work">
        <el-row :gutter="20" >
          <!--工具功能区-->
          <el-col :span="6"><div class="grid-content bg-purple base-col">
            <!--画笔颜色-->
              <h5 class="h-pad">颜色设置</h5>
              <span class="canvas-color">
                <el-tooltip class="item" effect="dark" content="画笔颜色"  placement="top">
              <el-color-picker
                v-model="preColor"
                show-alpha
                :predefine="predefineColors">
              </el-color-picker>
            </el-tooltip>
              </span>
              <span class="canvas-color">
               <el-tooltip class="item" effect="dark" content="背景颜色"  placement="top">
             <el-color-picker
               v-model="bgColor">
             </el-color-picker>
             </el-tooltip>
              </span>
            </div>
          </el-col>
          <!--画笔大小-->
          <el-col :span="6">
            <div class="grid-content bg-purple base-col">
              <h5 class="h-pad">画笔大小</h5>
              <el-radio-group v-model="sizeRadio" class="canvas-brush">
                <el-radio :label="1" ><i class="el-icon-edit" style="font-size: 12px;"></i></el-radio>
                <el-radio :label="3" ><i class="el-icon-edit" style="font-size: 14px;"></i></el-radio>
                <el-radio :label="5" ><i class="el-icon-edit" style="font-size: 16px;"></i></el-radio>
                <el-radio :label="7" ><i class="el-icon-edit" style="font-size: 18px;"></i></el-radio>
              </el-radio-group>
            </div>
          </el-col>
          <!--操作-->
          <el-col :span="6"><div class="grid-content bg-purple base-col base-col">
              <h5 class="h-pad">基本操作</h5>
            <div class="canvas-operate">
              <el-tooltip placement="top" class="item" content="上一步" effect="dark">
                <el-button  icon="el-icon-caret-left" @click="back()"></el-button>
              </el-tooltip>
              <el-tooltip class="item" effect="dark" content="下一步"  placement="top">
                <el-button  icon="el-icon-caret-right" @click="next()"></el-button>
              </el-tooltip>
              <el-tooltip class="item" effect="dark" content="清除"  placement="top">
                <el-button  icon="el-icon-delete" @click="clean()"></el-button>
              </el-tooltip>
              <!--<el-tooltip class="item" effect="dark" content="插入图片"  placement="top">-->
                <!--<el-button  icon="el-icon-picture" disabled></el-button>-->
              <!--</el-tooltip>-->
              <el-tooltip class="item" effect="dark" content="分享"  placement="top">
                  <el-button icon="el-icon-share" disabled></el-button>
              </el-tooltip>
            </div>
            </div>
          </el-col>
          <!--图片操作-->
          <el-col :span="6"><div class="grid-content bg-purple base-col">
              <h5 class="h-pad">图片操作</h5>
              <el-button  icon="el-icon-view" @click="save()">预览</el-button>
              <el-button  icon="el-icon-download" @click="downloadImg()">下载</el-button>
            </div>
          </el-col>
        </el-row>
        <!--工作区-->
        <div @mousemove="beginPath($event)" class="work-div" >
          <!--:width="canvasWidth+'px'" :height="canvasHeight+'px'"-->
          <canvas  id="myCanvas"
                   class="canvas-div"
                   :width="canvasWidth+'px'" :height="canvasHeight+'px'"
                   :style="'border:1px solid #000000;background: '+bgColor"
                   @mouseup="canvasUp($event)"
                   @mousedown="canvasDown($event)"
                   @mousemove="canvasMove($event)"
                   @mouseout="canvasOut($event)"
                   @touchstart="canvasDown($event)"
                   @touchend="canvasUp($event)"
                   @touchmove="canvasMove($event)">
          </canvas>
          <el-container class="show-image" :style="'width: '+showAreaWidth+'px;height: '+showAreaHeight+'px;'">
          <!--<el-container class="show-image" style="width: 50%;height: 50%;">-->
            <el-header class="h-pad" style="color: black">图片选择区</el-header>
            <el-main style="display: inline-flex">
              <div class="small-img-div"><img src="../../static/imgs/AAA.jpg" alt="加载失败" class="img-size"></div>
              <div class="small-img-div"><img src="../../static/imgs/AAA.jpg" alt="加载失败" class="img-size"></div>
              <div class="small-img-div"><img src="../../static/imgs/BBB.gif" alt="加载失败" class="img-size"></div>
              <!--<div class="small-img-div"><img src="../../static/imgs/BBB.gif" alt="加载失败" class="img-size"></div>-->
            </el-main>
          </el-container>
          <el-container  class="pre-view" :style="'width: '+showAreaWidth+'px;height: '+showAreaHeight+'px;'">
          <!--<el-container  class="pre-view" style="width: 40%;height: 50%;">-->
            <el-header class="h-pad" style="color: black">图片预览区</el-header>
            <el-main style="display: inline-flex">
              <div class="small-img-div"  v-show="imgUrl.length" v-for="src in imgUrl">
                  <span  @click="removeImg(src)"> <span style="color: red">X</span>
                    <img :src="src" class="img-size">
                  </span>
                </div>
            </el-main>
          </el-container>
        </div>
      </el-main>
  </el-container>
</template>

<script>
export default {
  data () {
    return {
      bgColor:'#ffffff',
      preColor: '#000000',
      predefineColors: [
        '#000000',
        '#ff4500',
        '#ff8c00',
        '#ffd700',
        '#90ee90',
        '#00ced1',
        '#1e90ff',
        '#c71585',
        'rgba(255, 69, 0, 0.68)',
        'rgb(255, 120, 0)',
        'hsv(51, 100, 98)',
        'hsva(120, 40, 94, 0.5)',
        'hsl(181, 100%, 37%)',
        'hsla(209, 100%, 56%, 0.73)',
        '#c7158577'
      ],
      sizeRadio: 1,
      input: '',
      context: {},
      imgUrl:[],
      //画布宽，高
      canvasWidth: 0,
      canvasHeight:0,
      showAreaWidth:0,
      showAreaHeight:0,
      preDrawArray:[],   // 存储当前表面状态数组-上一步
      middleArray:[],    // 中间数组
      nextDrawArray:[],  // 存储当前表面状态数组-下一步
      canvasMoveUse: false,
      //画笔配置
      config: {
        lineWidth: 1.2,
        lineColor: '#000000',
        shadowBlur: 1
      }
    }
  },
  created(){
    window.SCREENWIDTH = window.screen.availWidth;
    // window.SCREENWIDTH = document.getElementById('main-work').offsetWidth;
    window.SCREENHEIGHT = window.screen.availHeight;
    // window.SCREENHEIGHT = document.getElementById('main-work').offsetHeight;

  },
  mounted(){
    const canvas = document.querySelector('#myCanvas');
    // var c=document.getElementById("myCanvas");
    this.context = canvas.getContext("2d");
    this.init();
    this.setCanvasStyle();
  },
  watch:{
    preColor(curVal,oldVal){
      // this.config.lineColor = this.preColor;
      this.setColor(curVal);
    },
    input(){
      if(! this.input){ //没有自定义大小
        this.config.lineWidth = this.sizeRadio;
        return
      }
      if(!isNaN(this.input) && (parseFloat(this.input) < 10)){
        this.setSize(this.input);
        this.sizeRadio = ''
      }else{
        alert('请输入1-9之间的数值');
        this.setSize(this.sizeRadio);
        this.input = '';
      }
    },

    sizeRadio(curVal,oldVal){
      console.log('curVal: ',curVal,'oldVal: ',oldVal);
      //如果没有自定义画笔粗细
      this.setSize(curVal);
    }
  },
  methods:{
    setSize(num){
      // alert(num);
      this.input = '';
      this.config.lineWidth = num;
    },
    isPc () {
      const userAgentInfo = navigator.userAgent;
      const Agents = ['Android', 'iPhone', 'SymbianOS', 'Windows Phone', 'iPad', 'iPod'];
      let flag = true;
      for (let v = 0; v < Agents.length; v++) {
        if (userAgentInfo.indexOf(Agents[v]) > 0) {
          flag = false;
          break
        }
      }
      return flag
    },

    back(){
      if (this.preDrawArray.length) {
        const popData = this.preDrawArray.pop();
        const midData = this.middleArray[this.preDrawArray.length + 1];
        this.nextDrawArray.push(midData);
        this.context.putImageData(popData, 0, 0);
      }
    },
    next(){
      if (this.nextDrawArray.length) {
        const popData = this.nextDrawArray.pop();
        const midData = this.middleArray[this.middleArray.length - this.nextDrawArray.length - 2];
        this.preDrawArray.push(midData);
        this.context.putImageData(popData, 0, 0);
      }
    },

   init(){
      this.canvasWidth = window.SCREENWIDTH * 0.47+15;
      this.canvasHeight = window.SCREENHEIGHT * 0.56;
      this.showAreaWidth =  this.canvasWidth *0.95;
      this.showAreaHeight = this.canvasHeight * 0.5;
     const preData = this.context.getImageData(0,0,this.canvasWidth,this.canvasHeight);
     // const preData = this.context.getImageData(0,0,this.context.canvas.width,this.context.canvas.height);
     this.middleArray.push(preData)
   },
    canvasOut(e){
      this.canvasMoveUse = false;
    },
    canvasDown(e){
      this.canvasMoveUse = true;
      // client是基于整个页面的坐标
      // offset是canvas距离顶部以及左边的距离
      const canvasX = e.clientX - e.target.parentNode.offsetLeft;
      const canvasY = e.clientY - e.target.parentNode.offsetTop;
      this.setCanvasStyle();  //画笔设置
      //清除子路经
      this.context.beginPath();
      this.context.moveTo(canvasX,canvasY); //把路径移动到画布中的指定点，不创建线条
      // 当前绘图表面状态
      const preData = this.context.getImageData(0,0,this.canvasWidth,this.canvasHeight);
      // 当前绘图表面进栈
      this.preDrawArray.push(preData)
    },
    canvasMove(e){
      if(this.canvasMoveUse){
        const t = e.target;
        let canvasX;
        let canvasY;
        if(this.isPc()){
          canvasX = e.clientX - t.parentNode.offsetLeft;
          canvasY = e.clientY - t.parentNode.offsetTop;
        }else {
          canvasX = e.changedTouches[0].clientX - t.parentNode.offsetLeft;
          canvasY = e.changedTouches[0].clientY - t.parentNode.offsetTop;
        }
        // debugger;
        this.context.lineTo(canvasX,canvasY); //添加一个新点，然后在画布中创建从该点到最后指定点的线条
        this.context.stroke();
      }
    },
    canvasUp(e){
      const preData = this.context.getImageData(0,0,this.canvasWidth,this.canvasHeight);
      if(!this.nextDrawArray.length){
        //当前绘图表面进栈
        this.middleArray.push(preData);
      }else{
        this.middleArray = [];
        this.middleArray = this.middleArray.concat(this.preDrawArray);
        this.middleArray.push(preData);
        this.nextDrawArray = [];
      }
      this.canvasMoveUse = false
    },

    setColor(color){
      this.config.lineColor = color
    },
    setCanvasStyle(){
      this.context.lineWidth = this.config.lineWidth;
      this.context.shadowBlur = this.config.shadowBlur ; //设置或返回用于阴影的模糊级别
      this.context.shadowColor = this.config.lineColor;
      this.context.strokeStyle = this.config.lineColor;  //设置或返回用于笔触的颜色、渐变或模式
    },
    beginPath(e){
      const canvas = document.querySelector('#myCanvas');
      if(e.target !== canvas){
        this.context.beginPath(); //起始一条路径，或重置当前路径
      }
    },
    clean(){
      this.context.clearRect(0,0,this.canvasWidth,this.canvasHeight);
      this.bgColor = '#FFFFFF';
    },
    save(){
      const canvas = document.querySelector('#myCanvas');
      const src = canvas.toDataURL('image/png');
      // console.log('src',src);
      this.imgUrl.push(src);
      if (!this.isPc()) {
        // window.open(`data:text/plain,${src}`)
        window.location.href = src
      }

    },
    imgType(ty){
      let type = ty.toLowerCase().replace(/jpg/i,'jpeg');
      var r = type.match(/png|jpeg|bmp|gif/)[0];
      return 'image/'+ r;
    },
    downloadImg(){
      let type = 'png';
      let imgSrc = document.querySelector('#myCanvas').toDataURL("image/png");
      let imgData = imgSrc.replace(this.imgType(type),'image/octet-stream');
      let fileName = 'image' + '.'+type;
      // this.saveFile(imgData,fileName)
      let saveLink = document.createElement('a');
      saveLink.href = imgData;
      saveLink.download = fileName;
      let event = document.createEvent('MouseEvents');
      event.initEvent("click",true,false);
      saveLink.dispatchEvent(event);
    },

    removeImg (src) {
      this.imgUrl = this.imgUrl.filter(item => item !== src)
    },
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

  #main-work{
    min-height: 500px;
    width: 100%;
    height: 100%;
  }

  .canvas-div{
    float: left;
    /*padding-right: 20px;*/
  }

  .img-size{
    width: 100%;
    height: 100%;
  }
  .work-div{
    background: #ffffff;
    width: 100%;
    height: 30%;

  }
 .small-img-div{
   border: 1px solid;
   background: #f3f3f3;
   width: calc(100% / 4);
   height: calc(100% / 1.3);
   margin: 0 10px;
 }
  .show-image{
    /*background: #77bc4f;*/
    float: left;
    /*border:  1px solid;*/
    /*margin-left: 24px;*/
  }
  .pre-view{
    /*background: #77bc4f;*/
    /*border:  1px solid;*/
    /*float: right;*/
  }
  .base-col{
    color: white;
    text-align: center;
    font-size: 14px;
  }

  .h-pad{
    padding: 15px 0 0 0;
    text-align: center;
    color: white;
    /*width: 100%;*/
  }
  .h-strong{
    background: #77bc4f;
    font-size: 20px;
    font-weight:bold;
    margin: 0 20px;
  }
  .canvas-color{
    margin-left: 10px;
    margin-right: 10px;
}
  .canvas-brush {
    margin: 10px 0;
    cursor: pointer;
    font-size: 16px;
  }

  .canvas-operate{
    margin-bottom: 10px;
    bottom: 10px;
  }

  .canvas-save{
    margin: 0 10px;
  }

  .drawImage{
    width: 100px;
    height: 30px;
    font-size: 12px;
    line-height: 30px;
  }
  .img-box {
    flex: 1;
    padding-left: 10px;
    width: 50px;
    height: 50px;
  }
  .img-box .img-item {
    position: relative;
    display: inline-block;
  }
  .img-box .img-item .fa {
    position: absolute;
    cursor: pointer;
    right: 1px;
    top: -1px;
    font-size: 12px;
    font-weight: bold;
    display: none;
    color: #ccc;
  }

  .el-row {
    margin-bottom: 20px;
     /*&:last-child {*/
     /*margin-bottom: 0;*/
   /*}*/
  }
  .el-col {
    border-radius: 4px;
  }
  .bg-purple-dark {
    background:  #77bc4f;
  }
  .bg-purple {
    background:  #77bc4f;
  }
  .bg-purple-light {
    background:  #77bc4f;
  }
  .grid-content {
    border-radius: 4px;
    min-height: 100px;
  }
  .row-bg {
    padding: 10px 0;
    background-color:  #77bc4f;
  }

</style>
