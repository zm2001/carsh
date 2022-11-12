<template>
  <div class="home"  ref="test">
    <!-- 遮罩 防止移入挡板 坐标抖动 -->
    <div class="container-top" @mousemove="move($event)" @click="start"   ></div>
    <div class="container"  ref="container">
      <!-- 砖块 -->
      <div class="bricks" v-for="item in bricks" ref="brick">
      </div>

      <!-- 碰撞球 -->
      <div class="ball" v-for="item in balls" ref="ball"></div>
      <!-- 挡板 -->
      <div class="rebound" ref="rebound"></div>
    </div>
    <div class="control">

    </div>
  </div>
</template>

<script>

  export default {
    name: 'Home',
    data() {
      return {
        // 砖块的行数
        coulmns: 10,

        // 小球个数
        balls:[{x:250,y:250,speedX:0,speedY:5}],
        // 动画开始时间
        stratTime:undefined,
        // 小球移动动画id 用于停止动画
        animaId:undefined,

        // 是否开始游戏 
        isStart:false,

        timer:null,

      };
    },

    computed: {
      // 计算砖块个数
      bricks() {
        return this.coulmns *18
      }
    },

    mounted() {
      // 将所有砖块设置为绝对定位
      for(let i=0;i<this.$refs.brick.length;i++){
        this.$refs.brick[i].style.top = this.$refs.brick[i].offsetTop+'px'
        this.$refs.brick[i].style.left =this.$refs.brick[i].offsetLeft+'px'
      }
      for(let i=0;i<this.$refs.brick.length;i++){
        this.$refs.brick[i].style.position='absolute'
      }
      
    },
    methods: {
      // 移动挡板
      move(e){
        // 获取鼠标坐标 e.layerX
        // 设置偏移量
        this.$refs.rebound.style.left=e.layerX-this.$refs.rebound.offsetWidth/2+'px'
        // 移动到边界
        if(e.layerX<this.$refs.rebound.offsetWidth/2){
          this.$refs.rebound.style.left=0
        }
        if(e.layerX> this.$refs.container.offsetWidth-this.$refs.rebound.offsetWidth/2){
          this.$refs.rebound.style.left=this.$refs.container.offsetWidth-this.$refs.rebound.offsetWidth-3+'px'
        }
      },

      // 点击开始
      start(){
        console.log()
        if(!this.isStart){
          this.isStart=true
          this.ballMove()
        }else{
          window.cancelAnimationFrame(this.animaId);
          this.isStart=!this.isStart
          // this.balls.push({
          //   x:250,
          //   y:250,
          //   speedX:3,
          //   speedY:4
          // })
        }
      },

      
       // 小球运动
       ballMove(){
        
        // window.requestAnimationFrame()需要先得到id 用于下面小球触底取消动画
        this.animaId = window.requestAnimationFrame(this.ballMove)
        for(let i=0;i<this.balls.length;i++){


          // 小球位置 
          this.$refs.ball[i].style.left = this.$refs.ball[i].offsetLeft+this.balls[i].speedX+'px'
          this.$refs.ball[i].style.top = this.$refs.ball[i].offsetTop+this.balls[i].speedY+'px'
          
          // 边缘检测 碰到顶部
          if(this.$refs.ball[i].offsetTop<0){
            this.balls[i].speedY*=-1
            return
          }// 碰到左右边
          else if(this.$refs.ball[i].offsetLeft<0||this.$refs.ball[i].offsetLeft>this.$refs.container.clientWidth-this.$refs.ball[i].clientWidth){
            this.balls[i].speedX*=-1
            return
          }// 触底取消动画
          else if(this.$refs.ball[i].offsetTop>=700){
            // window.cancelAnimationFrame(this.animaId)
            this.balls[i].speedY*=-1
          }

          // 进行挡板和小球的碰撞检测
          this.$refs.ball[i]
          // console.log(this.$refs.ball[i].offsetLeft)
          this.konck(this.$refs.rebound,this.$refs.ball[i],i,'reb')

          // 检测小球与砖块的碰撞
          for(let j=0;j<this.$refs.brick.length;j++){
            let type = this.konck(this.$refs.brick[j],this.$refs.ball[i],i,'bri')
            if(type=='tb'){
              // this.balls[i].speedY*=-1
              this.$refs.brick[j].style.display='none'
              return
            }else if(type=='lr'){
              // this.balls[i].speedX*=-1
              this.$refs.brick[j].style.display='none'
            }
          }
        }
      },
      

      // 检测是否碰撞砖块
      konck(node1,node2,i,type){
        let l1 = node1.offsetLeft
        let r1 = node1.offsetLeft + node1.offsetWidth
        let t1 = node1.offsetTop
        let b1 = node1.offsetTop + node1.offsetHeight
        let x1 = l1+node1.clientWidth/2
        let y1 = t1+node1.clientHeight/2
        
        let l2 = node2.offsetLeft
        let r2 = node2.offsetLeft + node2.offsetWidth
        let t2 = node2.offsetTop
        let b2 = node2.offsetTop + node2.offsetHeight
        let x2 = l2+node2.clientWidth/2
        let y2 = t2+node2.clientHeight/2
        // 运动物体合速度
        let speed =Math.sqrt(this.balls[i].speedX*this.balls[i].speedX+this.balls[i].speedY*this.balls[i].speedY)
       
        // l2 > r1 || r2 < l1 || t2 > b1 || b2 < t1 判断俩个物体是否碰撞
        if(l2 > r1 || r2 < l1 || t2 > b1 || b2 < t1){
          return
        }else{
          // 撞到挡板
          if(type=='reb'){
            this.balls[i].speedX = (x2-x1)/node1.clientWidth/2*speed*3
            this.balls[i].speedY = -Math.sqrt(speed*speed-this.balls[i].speedX*this.balls[i].speedX)
            return
          }
          if(y2>b1||y2<t1){
            this.balls[i].speedY*=-1
            return 'tb'
            return
          }else if(x2>r1 || x2<l1){
            this.balls[i].speedX*=-1
            return 'lr'
            return
          }
        }
      },
    },
    watch:{
    }
    
  }
</script>


<style>
.home {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100vw;
  min-height: 100vh;
}
.container-top{
  width: 557px;
  height: 100vh;
  z-index: 1;
  position: absolute;
}

.container {
  width: 403px;
  height: 700px;
  border: 2px solid rgba(145, 146, 146, 0.918);
  display: flex;
  flex-wrap: wrap;
  align-content: flex-start;
  justify-content: center;
  position: relative;
  background-color: black;
}

.bricks {
  width: 20px;
  height: 20px;
  margin: 1px;
  background-color: cornflowerblue;
}

.rebound {
  width: 100px;
  height: 10px;
  border-radius: 5px;
  background: repeating-linear-gradient(to left, #03a9f4, #71b2ee,#03a9f4);
  background-size: 300%;
  border: 1px solid rgba(145, 146, 146, 0.918);
  position: absolute;
  bottom: 40px;
  left: calc(50% - 50px);
  animation: reb 6s  infinite;
}
@keyframes reb {
  100% {
    background-position: -300% 0;
  }
}
.ball{
  width: 12px;
  height: 12px;
  background-color: coral;
  border-radius: 10px;
  position: absolute;
  bottom: 252px;
  left: calc(50% - 5px);
}
.control{
  width: 150px;
  height: 704px;
  /* position: absolute; */
  background-color: #03a9f4;
  left:900px;
}
</style>