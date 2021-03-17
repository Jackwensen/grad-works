<!--  -->
<template>
  <div class="app-container">
    <div class="layout">
      <div class="alert">
        <el-alert
          title="基于dom的贪吃蛇"
          description="由方向键控制上下左右"
          type="info"
          effect="dark"
          show-icon>
        </el-alert>
      </div>
      <div id="snake">
        <div class="gnav">
          <div class="score">
            得分： {{ position.length - 4 }}
          </div>
          <el-button type="primary" @click="start()" round >开始</el-button>
          <el-button type="success" @click="stop()" round >暂停</el-button>
        </div>

        <table>
          <tr v-for="(col,y) in cols" :key="y">
            <td v-for="(row,x) in rows" :key="x" :class="{ 'snake':body(x,y), 'food':showfood(x,y), 'snake-head': isHead(x,y) }"></td>
          </tr>
        </table>
	    </div>
    </div>
  </div>
</template>

<script>
let timer=''//定时器

export default {
  data:()=> ({
    speed: 400,
    rows:'',//横框
    cols:'',//竖框
    position:[[0,0],[1,0],[2,0],[3,0]],//蛇的初始位置
    direction:1,//方向
    food:[]//食物的位置
  }),
  watch: {
    'position': 'checkSpeed'
  },

  mounted() {
    this.keyboard()
    this.background() 
    this.creatfood()
  },

  methods: {
    checkSpeed () {
      switch (this.position.length) {
        case 10:
          this.changeSpeed(300)
          break
        case 15:
          this.changeSpeed(200)
          break
        case 20:
          this.changeSpeed(100)
          break
        default:
          break
      }
    },
    changeSpeed (speed) {
      clearInterval(timer)
      this.speed = speed
      this.start()
    },

    background(){//生成横框和竖框的函数
      this.rows = Array(30)
      this.cols = Array(30)
    },
    keyboard(){//键盘事件
    let _this = this;
      document.onkeydown = event => {
        if(event.keyCode === 37){
          _this.change(-1)
        }else if(event.keyCode === 38){
          _this.change(-2)
        }else if(event.keyCode === 39){
          _this.change(1)
        }else if(event.keyCode === 40){
          _this.change(2)
        }
      }
    },
    change(dir){//改变方向
      if(Math.abs(dir)===Math.abs(this.direction)){//如果方向相同或者相反，不做任何操作
        return
      }else{
        this.direction=dir//否则改变方向
      }
    },

    //创造食物
    creatfood(){
      // 试图解决食物刷新在蛇身上
      // var foodX = Math.floor(Math.random()*30)
      // var foodY = Math.floor(Math.random()*30)
      // for (var i = 0 ; i < this.position.length; i++){
      //   if (this.position[i][0] === foodX && this.position[i][1] === foodY){
      //     foodX = Math.floor(Math.random()*30)
      //     foodY = Math.floor(Math.random()*30)
      //   }
      // }
      this.food[0] = Math.floor(Math.random()*30)
      this.food[1] = Math.floor(Math.random()*30)
    },

    //显示食物
    showfood(x,y){
      if(this.food[0] === x && this.food[1] === y){
        return true
      }
    },

    //显示蛇头
    isHead (x,y) {
      let head = this.position[this.position.length - 1]
      return head[0] === x && head[1] === y
    },

    //显示身体
    body(x,y){
      for(var i = 0; i < this.position.length; i++){
        if(this.position[i][0]===x && this.position[i][1]===y){
          return true;
        }
      }
    },

    //开始按钮
    start(){
      timer = setInterval(() => this.autorun(), this.speed)
    },

    stop(){
      clearInterval(timer)
    },	
    
    autorun(){					
      //目前方向
      let direction = this.direction
      let headX, headY
      //复制蛇头的X坐标
      headX = this.position[this.position.length - 1][0]
      //复制蛇头的Y坐标
      headY = this.position[this.position.length - 1][1]

      //如果方向是在左右跑动
      if(direction === 1 || direction === -1){
        //往右跑X坐标+1，往左跑X坐标-1 	
        direction > 0 ? headX++ : headX--									
      } else {
        //如果方向是在上下跑动，Y坐标做对应处理
        direction > 0 ? headY++ : headY--
      }
      //此时蛇头的下一个坐标位置就是[headX,headY]，接下来就可以判断是否结束游戏，如果结束了，蛇头就没必要添加了

      //当蛇头下一个位置出了边界或者这个位置是符合身体函数（即蛇头撞上了身体）
      if ( headX < 0 || headX > 29 || headY < 0 || headY > 29 || this.body(headX,headY) ){
        // alert('Game Over')
        setTimeout(() => {//延迟弹窗,确定:重玩
          this.$msgbox(`(⊙o⊙)！Game over！` , '失败！！', {
            confirmButtonText: '确定',
          })
          // this.init();
        }, 100);
        clearInterval(timer)
        this.position = [[0,0],[1,0],[2,0],[3,0]]
        this.creatfood()
        this.direction = 1
      } else {//如果蛇头下一个位置是符合规则的						
        this.position.push( [headX,headY] )//将下一个位置添加进数组，头部长一节

        //如果下一个头部位置不是食物的位置，即吃食物开始
        if( headX !== this.food[0] || headY !== this.food[1] ){							
            //将尾部去掉，一长一短实现了蛇的走动
            this.position.shift()
        } else {//如果下一个头部位置是食物
          this.creatfood()
        }
      }	
    }
  }
}

</script>

<style lang='scss' scoped>
  .alert {
    width: 750px;
  }
  .layout {
    overflow: auto;
    -webkit-overflow-scrolling: touch;
    display: flex;
    justify-content: center;
    flex-direction: column;
    align-items: center;
    height: 100%;
    overflow: auto;
  }
  .score {
    font-size: 18px;
    float: left;
    color: #776e65;
    font-weight: bold;
    // margin-left: 20px;
    margin-top: 10px;
    width: 100px;
    display: inline-flex;
    align-items: center;
    justify-content: center;
  }
  .gnav {
    padding-top: 10px;
    height: 60px;
    text-align: center;
  }
  td{
    border: 0.5px solid rgb(206, 171, 171);
    width: 22px;
    height: 22px;
  }

  table{
    border-collapse: collapse;
  }
  
  .food{
    background: #f60
  }
  .snake {
  background: #1E90FF;
  }
  .snake-head {
  background: rgb(202, 91, 214);
  }
</style>
