<template>
  <div class="home" @touchmove="rebround($event)" @touchstart="rebround($event)" >
    <canvas class="canvas" @click="start"   ref="cvs"></canvas>
    <div v-if="!isStart" class="tips">点击开始游戏</div></div>
  </div>
</template>

<script>
export default {
  name: 'WorkspaceJsonAbout',

  data() {
    return {
      // 页面宽度
      pageW:document.documentElement.clientWidth,

      cvs:'',
      pen:'',

      // 砖块
      bricks:[],
      // 砖块行数
      row:15,
      cross:23,
      // 砖块宽度
      brickW:12,

      // 球
      ball:[{x:100,y:450,width:6,speedX:4,speedY:-5}],

      // 挡板
      bround:{
        x:110,
        y:500,
        width:80,
      },

      // 道具
      prop:[],

      // 动画id
      animaId:null,

      // 是否开始游戏 
      isStart:false,
    };
  },

  mounted() {
    this.cvs=this.$refs.cvs
    this.pen=this.$refs.cvs.getContext('2d')
    this.cvs.width="300"
    this.cvs.height="550"

    //制造砖块数组
    for(let i=0;i<this.row;i++){
      for(let j=0;j<this.cross;j++){
        this.bricks.push({
          x:j,
          y:i,
        })
      }
    }
    // 绘制砖块 砖块宽度15
    for(let i=0;i<this.bricks.length;i++){
      this.pen.fillStyle='#ffffff'
      this.pen.fillRect(this.bricks[i].x * 13+1, this.bricks[i].y * 13+1, this.brickW, this.brickW)
    }
    //绘制反弹板
    this.pen.fillStyle='#6495ed'
    this.pen.fillRect(this.bround.x, this.bround.y, this.bround.width, 10) 

  },

  methods: {
    // 游戏开始 通过清除画布和重绘来模拟游戏运行状态
    move(){
      if(this.isStart){
        // 清除画布
        this.pen.clearRect(0, 0, 300, 550)
        
        this.animaId = window.requestAnimationFrame(this.move)
        // 绘制砖块 砖块宽度15
        for(let i=0;i<this.bricks.length;i++){
          this.pen.fillStyle='#ffffff'
          this.pen.fillRect(this.bricks[i].x * 13+1, this.bricks[i].y * 13+1, this.brickW, this.brickW)

          // 判断是否与球撞击
          let brick = {}
          brick.x=this.bricks[i].x * 13+1
          brick.y=this.bricks[i].y * 13+1
          brick.width=this.brickW
          brick.height=this.brickW

          for(let j=0;j<this.ball.length;j++){
            if(this.konck(brick,this.ball[j],'bricks',j)){
              // 随机产生道具
              if(Math.random()>0.8+this.ball.length/300+this.prop.length/50){
                this.prop.push({x:this.bricks[i].x*13+10,y:this.bricks[i].y*13+3,width:7})
              }
              // 消除砖块
              this.bricks.splice(i,1)
            }
          }
        }

        //绘制反弹板
        this.pen.fillStyle='#6495ed'
        this.pen.fillRect(this.bround.x, this.bround.y, this.bround.width, 10) 

        //绘制小球
        for(let j=0;j<this.ball.length;j++){
          this.pen.beginPath()
          this.pen.fillStyle='#ffffff'
          this.pen.arc(this.ball[j].x, this.ball[j].y, this.ball[j].width, 0, 2 * Math.PI)
          this.pen.fill()

          // 更改小球位置
          this.ball[j].x=this.ball[j].x+this.ball[j].speedX
          this.ball[j].y=this.ball[j].y+this.ball[j].speedY

          // 进行挡板碰撞检测
          this.bround.height=5
          this.konck(this.bround,this.ball[j],'bround',j)

          // 进行边缘碰撞检测
          if(this.ball[j].x<5 || this.ball[j].x+this.ball[j].width>=this.cvs.width){
            this.ball[j].speedX*=-1
          }else if(this.ball[j].y<=this.ball[j].width/2){
            this.ball[j].speedY*=-1
          }else if( this.ball[j].y+this.ball[j].width/2>this.cvs.height||this.ball[j].x<4||this.ball[j].y<0||this.ball[j].x>this.cvs.width){
            this.ball.splice(j,1)
            // this.ball[j].speedY*=-1
          }
        }

         // 绘制道具
        for(let i=this.prop.length-1;i>=0;i--){
          this.pen.beginPath()
          this.pen.fillStyle='#ff8936'
          this.pen.arc(this.prop[i].x, this.prop[i].y, this.prop[i].width, 0, 2 * Math.PI)
          this.pen.fill()
          this.prop[i].y=this.prop[i].y+3

          // 触底消失
          if(this.prop[i].y>=this.cvs.height){
            this.prop.splice(i,1)
          }
          // 接到道具 创建小球
          if(this.konck(this.bround,this.prop[i],'prop')){
            this.prop.splice(i,1)
            this.createBall()
          }
        }
      }
    },

    // 增加小球
    createBall(i){
      for(let i=this.ball.length-1;i>=0;i--){
        this.ball.push({
          x:this.ball[i].x,
          y:this.ball[i].y,
          width:6,
          speedX:-this.ball[i].speedX,
          speedY:this.ball[i].speedY,
        })
        this.ball.push({
          x:this.ball[i].x,
          y:this.ball[i].y,
          width:6,
          speedX:this.ball[i].speedX,
          speedY:-this.ball[i].speedY,
        })
      }
    },

    // 移动挡板
    rebround(e){
      this.bround.x=e.touches[0].clientX-this.pageW/2+this.$refs.cvs.offsetWidth/2-this.bround.width/2
      if(this.bround.x<0){
        this.bround.x=0
      }else if(this.bround.x>this.cvs.width-this.bround.width){
        this.bround.x=this.cvs.width-this.bround.width
      }
      
    },

    // 碰撞检测
    konck(node1,node2,type,i){
      let l1 = node1.x
      let r1 = node1.x+node1.width
      let t1 = node1.y
      let b1 = node1.y+node1.height
      let x1 = node1.x+node1.width/2
      let y1 = node1.y+node1.height/2
      
      let l2 = node2.x-7
      let r2 = node2.x+node2.width
      let t2 = node2.y-7
      let b2 = node2.y+node2.width
      let x2 = node2.x+node2.width/2
      let y2 = node2.y+node2.width/2

      
      if(l2 > r1 || r2 < l1 || t2 > b1 || b2 < t1){
          return false
      }else{
        // 小球撞到挡板
        if(type=='bround'){
          // 运动物体合速度
          let speed = Math.sqrt( node2.speedX * node2.speedX + node2.speedY * node2.speedY)
          this.ball[i].speedX = (x2-x1)/node1.width/2*speed*3
          this.ball[i].speedY = -Math.sqrt(speed*speed - node2.speedX * node2.speedX)
          return
        }else if(type=="bricks"){
          if(y2>=b1||y2<=t1){
            this.ball[i].speedY*=-1
            console.log('上下')
            return true
          }else if(x2>=r1 || x2<=l1){
            this.ball[i].speedX*=-1
            console.log('左右')
            return true
          }
        }else if(type=='prop'){
          console.log('道具')
          return true
        }
        return true
      }
    },
    
    // 点击开始
    start(){
      if(!this.isStart){
        this.isStart=!this.isStart
        this.move()
      }else{
        // this.isStart=!this.isStart
        // this.createBall()
      }
    }
  },

  watch:{
    bricks(val){
      if(this.bricks.length==0){
        // window.cancelAnimationFrame(this.animaId)
      }
    },
  }
};
</script>

<style lang="css" scoped>
.home {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100vw;
  min-width: 300px;
  min-height: 100vh;
  background-color: #abababd6;
  touch-action: none;
}
.canvas{
  width: 300px;
  height: 550px;
  background-color: #000000d6;
}
.tips{
  position: absolute;
  color: aliceblue;
  animation: changetips 4s infinite;
}
@keyframes changetips{
  0%{
    opacity: 0.3;
  }
  50%{
    opacity: 1;
  }
  100%{
    opacity: 0.3;
  }
}
</style>