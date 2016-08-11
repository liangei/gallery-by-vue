<template>
  <header class="header">
    <a href="" class="logo"><img  src="./assets/logo.png"></a>
    <h1 class="title" v-text="msg"></h1>
  </header>
  <section class="stage" id="stage" v-el:stage>
    <section class="img-sec">
       <figure  v-for="figure in figures" class="img-fig" v-bind:class="{ 'is-inverse' : isInverse[$index] }" v-on:click.stop="handleClick($index)" v-bind:style="styleObject[$index]">
          <img :src="figure.src" alt="{{figure.title}}">
          <figcaption>
            <h3 class="img-tit">{{figure.title}}</h3>
            <div class="img-back">
              {{figure.desc}}
            </div>
          </figcaption>

       </figure>
    </section>
    <nav class="contorller-nav">
      <span v-for="figure in figures"></span>
    </nav>
  </section>
</template>
<script>
let imagesData = require('./data/imagesData.json')
export default {
  data () {
    return {
      msg: '图片画廊',
      figures: imagesData, //图片信息数组
      constant : {   //初始化取值范围
        conterPos: { //中心
          top:0,
          left:0
        },
        hPosRange: { //水平方向取值范围
          leftSecX:[0, 0],
          rightSecX:[0, 0],
          y:[0, 0]
        },
        vPosRange: { //垂直方向取值范围
          topY:[0, 0],
          x:[0, 0]  
        }
      },
      imgsArrangeArr:[],
      styleObject: [],
      isInverse:[]
    }
  },
  methods : {
    handleClick (i) {
      if (!this.imgsArrangeArr[i].isCenter) {
        this.rearrange(i);
      }else{
        this.inverse(i)
      }
    },
    /*
     *翻转图片
     * @param i 输入当前被执行inverse操作对应的图片信息数组index
     * @return 返回一个布尔值
     */
    inverse (i) {
        this.isInverse.$set(i, !this.isInverse[i] )
    },
    /*
     *重新排布所有图片
     *@param i 指定居中哪个图片
     */
    rearrange (i)  {
      let imgsArrangeArr = this.imgsArrangeArr,
          styleObject = this.styleObject,
          constant = this.constant,
          conterPos = constant.conterPos,
          hPosRange = constant.hPosRange,
          vPosRange = constant.vPosRange,
          leftSecX = hPosRange.leftSecX,
          rightSecX = hPosRange.rightSecX,
          hPosRangeY= hPosRange.y,
          vPosRangeY = vPosRange.topY,
          vPosRangeX = vPosRange.x,

          imgsArrangeTopArr = [],
          topNum = Math.floor( Math.random() * 2 ),//获取0-1到数值
          topImgSpliceIndex = 0,//标志顶部图片是图片数组哪个位置取出来的
          imgsArrangeCenterArr = imgsArrangeArr.splice(i, 1);//中间图片数组

          //排布居中centerIndex的图片
          imgsArrangeCenterArr[0] = {
            pos:conterPos,
            rotate:0,
            isCenter:true
          };


          //取出顶部图片位置信息
          topImgSpliceIndex = Math.ceil( Math.random() * ( imgsArrangeArr.length - topNum ));
          imgsArrangeTopArr = imgsArrangeArr.splice(topImgSpliceIndex, topNum);

          //排布顶部图片
          imgsArrangeTopArr.forEach((v, i)=>{
            imgsArrangeTopArr[i] = {
              pos: {
                top: this.getRangeRandom(vPosRangeY[0], vPosRangeY[1] ),
                left: this.getRangeRandom(vPosRangeX[0], vPosRangeX[1] )
              },
              rotate: this.get30RangeRandom(),
              isCenter:false
            }
          });

          //排布左右区域图片
          for (let i = 0, j = imgsArrangeArr.length, k = j / 2; i < j; i++) {
              let hPosRangeLORX = null;

               //前半图片在左，后半图片在右
              if (i < k) {
                hPosRangeLORX = leftSecX;
              }else{
                hPosRangeLORX = rightSecX;
              };

              imgsArrangeArr[i] = {
                pos:{
                  top: this.getRangeRandom(hPosRangeY[0],hPosRangeY[1] ),
                  left: this.getRangeRandom(hPosRangeLORX[0], hPosRangeLORX[1])
                },
                rotate: this.get30RangeRandom(),
                isCenter:false  
              };
          };

          //重新组合图片数组，把之前抽出的插回
          if(imgsArrangeTopArr && imgsArrangeTopArr[0]){
            imgsArrangeArr.splice(topImgSpliceIndex, 0 ,imgsArrangeTopArr[0] )
          }
          imgsArrangeArr.splice(i, 0, imgsArrangeCenterArr[0]);

          imgsArrangeArr.forEach((v,i)=>{
            let styleOBJ = {
               top:  v.pos.top + 'px',
               left: v.pos.left+ 'px',
            }
            if (v.rotate) {
              styleOBJ['transform'] = 'rotate('+ v.rotate +'deg)';
            }
            if (v.isCenter) {
               styleOBJ.zIndex = 11;
            }
            styleObject.$set(i, styleOBJ) 
          })
        },
        //获取区域的随机数值
        getRangeRandom (min, max) {
          return Math.ceil( Math.random() * (max - min) + min) 
        },
        //获取正负30度数值
        get30RangeRandom () {
          return  ((Math.random() > 0.5 ? '':'-') + Math.ceil( Math.random() * 30 ))
        }
  },
  created () {
       let images = this.figures;
       for (let i = 0; i < images.length; i++) {
            images[i].src = "static/images/"+images[i].fileName;
            if (!this.imgsArrangeArr[i]) {
              this.imgsArrangeArr[i] = {
                pos:{
                  top:0,
                  left:0
                },
                rotate:0,
                isCenter:false
              }
            };
            if (!this.isInverse[i]) {
               this.isInverse.push(false)
            }
       }
  },
  ready () {
     //获取舞台大小
     let stageDOM = this.$els.stage,
         stageW = stageDOM.scrollWidth,
         stageH = stageDOM.scrollHeight,
         halfStageW = Math.ceil(stageW / 2),
         halfStageH = Math.ceil(stageH /2);

       
     //获取图片大小
      let imgFigueDOM = stageDOM.children[0].children[0],
          imgW = imgFigueDOM.scrollWidth,
          imgH = imgFigueDOM.scrollHeight,
          halfImgW = Math.ceil(imgW / 2),
          halfImgH = Math.ceil(imgH / 2);


     //设置图片中心位置
     this.constant.conterPos = {
          top: halfStageH - halfImgH ,
          left: halfStageW - halfImgW 
     } 

     //设置左右区域取值范围
     this.constant.hPosRange.leftSecX[0] = -halfImgW;
     this.constant.hPosRange.leftSecX[1] = halfStageW - halfImgW / 2 * 7;
     this.constant.hPosRange.rightSecX[0] = halfStageW + halfImgW * 2;
     this.constant.hPosRange.rightSecX[1] =  stageW - halfImgW;
     this.constant.hPosRange.y[0] = -halfImgH;
     this.constant.hPosRange.y[1] = stageH - halfImgH;

     //设置上区域取值范围
     this.constant.vPosRange.topY[0] = -halfImgH;
     this.constant.vPosRange.topY[1] = halfStageH - halfImgH / 2 * 7;
     this.constant.vPosRange.x[0] = halfStageW - imgW;
     this.constant.vPosRange.x[1] = halfStageW ;   

     this.rearrange(0);
       
  }
}
</script>

<style lang="scss">
  @import 'style/app.scss'
</style>
