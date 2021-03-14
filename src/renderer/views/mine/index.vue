<!--  -->
<template>
  <div class="app-container">
    <div id="gnav" ref="gnav">
			<!-- <input type="text" id="mine-total" /> -->
      <input v-model="mine_total" id="mine-total" />

      <!-- <el-radio v-model="level" label="0" @click="select_lev(0)" >初级</el-radio>
      <el-radio v-model="level" label="1" @click="select_lev(1)" >中级</el-radio>
      <el-radio v-model="level" label="2" @click="select_lev(2)" >高级</el-radio> -->

      <el-radio-group v-model="level">
        <el-radio :label="0">初级</el-radio>
        <el-radio :label="1">中级</el-radio>
        <el-radio :label="2">高级</el-radio>
      </el-radio-group>
      
      <el-button type="primary" size="mini" @click="select_lev()">重新开始</el-button>

			<!-- <input type="text" id="timer" /> -->
      <input v-model="mine_timer" id="timer" />
		</div>

    <canvas id="myCanvas"></canvas>
  </div>
</template>

<script>
let canv;
let ctx;
let levels= [
    [9, 9, 10],
    [16, 16, 40],
    [16, 30, 99],
  ];
let g = levels[0];
let g_color = {//预设游戏块颜色
    block: '#369',
    open: '#ddd',
    mine: '#69f',
    bomb: '#f66',
    highlight: '#69d',
  };
let gblock = {//布局,游戏块尺寸:宽度,圆角,外边距
    width: 50,
    radius: 6,
    margin: 2,
  };

export default {
  data () {
    return {
			//预设游戏等级0
			level: 0,
			// g: levels[this.level],//当前游戏等级信息
			g_arr: [],//游戏块id列表
			g_info: {},//每个块的游戏信息
      mine_total: g[2],
      mine_timer: 0,
			mine_arr: [],//当前游戏雷块id列表
			count: 0,//已标记雷块统计
			over: false,//游戏是否结束
			win: false,//游戏是否获胜
			XY: '',//构造xy
			mine: ['💣', '🚩', '❔', '💥'],//预设雷块标记符号
			gamestart: 0, //游戏是否开始
			ttimer: 0, //游戏计时器
			durtime: 0, //游戏耗时记录
    };
  },

  mounted() {
    this.initCanvas()
    this.init();
  },

  methods: {
    initCanvas() {      
      
    },
    init() {
      console.log("初始化canvas")
      canv = document.getElementById('myCanvas');
      ctx = canv.getContext('2d');
      //初始化布局及游戏信息
			//------重置游戏基础信息------
      this.g_arr = [];
      this.mine_arr = [];
      this.count = 0;
      this.over = false;
      this.win = false;
      this.gamestart = 0;
      this.durtime = 0;
      clearInterval(this.ttimer);//清除定时器
      g = levels[this.level];//获取游戏等级,重置游戏画布及相关游戏数据
      this.$refs.gnav.style.width = g[1] * gblock.width + 'px';
      this.mine_total = g[2];
      this.mine_timer = 0;
      let h = g[0] * gblock.width;
      let w = g[1] * gblock.width;
      canv.height = h;
      canv.width = w;
      ctx.clearRect(0, 0, w, h);//清除画布
      //按行列输出游戏块
      for (let i = 0; i < g[0]; i++) {
        for (let j = 0; j < g[1]; j++) {
          let xy = j + '-' + i;//根据坐标构造游戏块id
          this.g_arr.push(xy);//g_arr记录游戏块id
          this.g_info[xy] = {//对每个游戏块, 预设游戏信息: 
            mark: 0,//mark:数字标记0-8或雷标记-1;
            open: 0,//open:游戏块打开状态:0未打开/1已打开/-1标记雷块/-2疑似雷块
          };
          this.drawBlock(xy, g_color.block);//绘制: 块,颜色
        }
      }
      //随机布雷
      this.setMine();
      // showInfo();
      canv.addEventListener('click', this.openBlock),
      canv.addEventListener('contextmenu', this.markMine),
      canv.addEventListener('mousedown', this.highLight);
      canv.addEventListener('mouseup', this.supGame); 
    },

    select_lev(lv) {//选择游戏等级
      console.log(123, lv)
      this.level = lv || this.level;
      this.init();
    },

    //绘制游戏块: 圆角矩形
    drawBlock(xy, c) {
      let [x, y] = xy.split('-').map(n => n * gblock.width);//解析id,并构造坐标
      let w = gblock.width - gblock.margin;
      let r = gblock.radius;
      ctx.clearRect(x, y, gblock.width, gblock.width); 
      ctx.save();
      ctx.beginPath();
      ctx.moveTo(x, y + gblock.radius);
      ctx.arcTo(x, y + w, x + w, y + w, r);
      ctx.arcTo(x + w, y + w, x + w, y, r);
      ctx.arcTo(x + w, y, x, y, r);
      ctx.arcTo(x, y, x, y + w, r);
      ctx.closePath();
      ctx.fillStyle = c;
      ctx.fill();
      ctx.restore();
    },

    //随机布雷: 生成雷块列表mine_arr,更新游戏块信息g_info:标记为雷或计算数字
    //对游戏块随机打乱,提取定量雷块
    setMine() {
      this.mine_arr = this.g_arr.sort(() => Math.random() - 0.5).slice(0, g[2]);
      this.mine_arr.forEach(xy => {
        this.g_info[xy].mark = -1;//将游戏块标记为雷-1
        this.getAround(xy).forEach(n => {//获取当前雷块周边8块: 计算数字
          if (this.g_info[n].mark != -1) this.g_info[n].mark++;//每布一个雷,对于周边非雷块数字+1
        });
      });
    },

    //获取当前游戏块的周边有效块
    getAround(xy) {
      let [x, y] = xy.split('-').map(n => n * 1);
      let around = [];
      //左中右,上中下
      for (let i = -1; i <= 1; i++) {
        for (let j = -1; j <= 1; j++) {
          //构造游戏块id
          let id = `${x + i}-${y + j}`;
          //判断id是否有效:在游戏块数组g_arr中包含, 并排除自身块;
          if (this.g_arr.includes(id) && id != xy) around.push(id);
        }
      }
      return around;
    },

    //在游戏块上标注文本: 数字或雷块标记
    markText(xy, text) {
      let [x, y] = xy.split('-').map(n => n * gblock.width);
      ctx.save();
      ctx.fillStyle = '#111';
      ctx.font = '20px arial';
      ctx.textAlign = 'center';
      ctx.textBaseline = 'middle';
      ctx.fillText(text, x + gblock.width / 2, y + gblock.width / 2);
      ctx.restore();
    },
    //辅助显示布雷情况信息, 显示数字和雷块标记
    showInfo() {
      this.g_arr.forEach(xy => {
        if (this.g_info[xy].mark == -1) {
          this.drawBlock(xy, g_color.mine);
        } else {
          //显示数字
          this.drawBlock(xy, g_color.block);
          this.markText(xy, this.g_info[xy].mark);
        }
      });
    },

    highLight(ev) {//右击非雷块,辅助: 高亮周边
      if (this.over) return;
      //获取正确坐标
      let x = ~~(ev.offsetX / gblock.width);
      let y = ~~(ev.offsetY / gblock.width);
      let xy = x + '-' + y;
      if (this.g_info[xy].open == 1) this.getAround(xy).forEach(n => {
        if (this.g_info[n].open == 0) {
          this.drawBlock(n, g_color.highlight);
        }
      });
    },

    startTimer() {//游戏开始计时
      this.ttimer = setInterval(() => {
        this.durtime++;
        this.mine_timer = (this.durtime / 10).toFixed(1);
      }, 100);
    },

    //右击非雷块,辅助: 鼠标按下高亮,鼠标松开取消高亮并标注确定的游戏块(打开或标记雷)
    supGame(ev) {
      if (this.over) return;
      //获取正确坐标
      let x = ~~(ev.offsetX / gblock.width);
      let y = ~~(ev.offsetY / gblock.width);
      let xy = x + '-' + y;
      if (this.g_info[xy].open == 1) {
        let around = this.getAround(xy);//获取当前游戏块周边
        let mark = this.g_info[xy].mark;
        let marked_mine = 0;//已标记雷块数量
        let unopen = 0;//未打开块数量
        around.forEach(n => {//统计周边游戏块信息: 未打开块数量和已标记雷数量
          if (this.g_info[n].open == 0 || this.g_info[n].open == -2) unopen++;
          if (this.g_info[n].open == -1) marked_mine++;
        });
        around.forEach(n => {//遍历周边块,
          if (this.g_info[n].open == 0) {
            this.drawBlock(n, g_color.block);//取消高亮
            //辅助扫雷
            if (mark == marked_mine) {//如果当前数字等于已经标记的雷块:雷已经全部排出, 其他为安全块
              this.g_info[n].open = 1;//安全块,自动打开
              this.drawBlock(n, g_color.open);
              this.markText(n, this.g_info[n].mark);
              if (this.g_info[n].mark == 0) this.openZero(n);//如果是0块, 递归清零(0块说明周边没有雷)
              if (this.g_info[n].mark == -1) {//在安全块中遇到雷(说明标记了错误雷块)
                this.drawBlock(n, g_color.bomb);
                this.markText(n, this.mine[0]);
                this.markText(n, this.mine[3]);
                this.checkOver(true);//游戏结束
              }
            } else if (unopen == mark - marked_mine) {//如果剩余的块都是雷, 则直接标注雷
              this.g_info[n].open = -1;
              this.drawBlock(n, g_color.mine);
              this.markText(n, this.mine[1]);
              this.count++;
              this.mine_total = g[2] - this.count;
              if (this.count == g[2]) this.checkOver();//标记雷之后, 判断数量, 是否完成扫雷
            }
          }
        });
      }
    },

    openBlock(ev) {//左键单击,打开游戏块
      if (this.over) return;
      if (this.gamestart == 0) {//打开第一个块,游戏开始标记
        this.gamestart = 1;
        this.startTimer();
      }
      //获取正确坐标
      let x = ~~(ev.offsetX / gblock.width);
      let y = ~~(ev.offsetY / gblock.width);
      let xy = x + '-' + y;
      if (this.g_info[xy].open == 0) {//仅对未打开的游戏块有效
        this.g_info[xy].open = 1;//常规标注,标记打开状态
        this.drawBlock(xy, g_color.open);
        this.markText(xy, this.g_info[xy].mark);
        if (this.g_info[xy].mark == 0) {//遇到0块, 递归清零
          this.openZero(xy);
        } else if (this.g_info[xy].mark == -1) {//点爆雷块, 游戏结束
          this.drawBlock(xy, g_color.bomb);
          this.markText(xy, this.mine[0]);
          this.markText(xy, this.mine[3]);
          this.checkOver(true);
        }
      }
    },

    openZero(xy) {//递归清零,遇到0块说明周边安全,可以全部打开
      this.getAround(xy).forEach(n => {
        if (this.g_info[n].open == 0) {
          this.g_info[n].open = 1;
          this.drawBlock(n, g_color.open);
          this.markText(n, this.g_info[n].mark);
          if (this.g_info[n].mark == 0) this.openZero(n);
        }
      });
    },

    checkOver(bomb) {//判断游戏结束,
      this.over = true;
      clearInterval(this.ttimer);
      //判断是否获胜:所有标注的雷块open-1是否和对应的mark-1一致.
      //bomb:左键点爆雷,或辅助扫雷点爆,游戏直接失败结束
      this.win = bomb ? false : this.mine_arr.every(xy => this.g_info[xy].mark == this.g_info[xy].open);
      
      setTimeout(() => {//延迟弹窗,确定:重玩
        if(this.win){
          this.$msgbox(`恭喜胜利!\n耗时：${(this.durtime/10).toFixed(1)}秒` , '恭喜！！', {
            confirmButtonText: '确定',
          })
        } else {
          this.$msgbox(`您已失败，请重新开始` , '失败', {
            confirmButtonText: '确定',
          })
        }

      //   let restart = confirm(this.win ? 
      //   this.$msgbox(`恭喜胜利!\n耗时：${(this.durtime/10).toFixed(1)}秒` , '恭喜！！', {
      //     confirmButtonText: '确定',
      //   })
      //   : 
      //   this.$message({
      //     showClose: true,
      //     message: '错了哦，这是一条错误消息',
      //     type: 'error'
      //   })
      // );
        
      // if (this.win) 
      this.init();
      }, 100);
    },

    //右键标注雷块
    markMine(ev) {
      //禁用右键的浏览器默认菜单:阻止默认动作
      ev.preventDefault();
      if (this.over) return;
      if (this.gamestart == 0) {
        this.gamestart = 1;
        this.startTimer();
      }
      //获取正确坐标
      let x = ~~(ev.offsetX / gblock.width);
      let y = ~~(ev.offsetY / gblock.width);
      let xy = x + '-' + y;
      if (this.g_info[xy].open == 0) {//如果是未打开块, 标注雷-1
        this.g_info[xy].open = -1;
        this.drawBlock(xy, g_color.mine);
        this.markText(xy, this.mine[1]);
        this.count++;
        this.mine_total = g[2] - this.count;
        if (this.count == g[2]) this.checkOver();
      } else if (this.g_info[xy].open == -1) {//如果已标注雷-1, 则标注为疑似雷-2
        this.g_info[xy].open = -2;
        this.drawBlock(xy, g_color.mine);
        this.markText(xy, this.mine[2]);
        this.count--;
        this.mine_total = g[2] - this.count;
      } else if (this.g_info[xy].open == -2) {//如果标注疑似雷-2, 则恢复未打开状态0
        this.g_info[xy].open = 0;
        this.drawBlock(xy, g_color.block);
      }
    }
  }
}
</script>

<style rel='stylesheet/scss' lang='scss' scoped>
  #gnav {
    height: 30px;
    text-align: center;
  }

  #mine-total {
    width: 30px;
    text-align: center;
  }

  #timer {
    width: 60px;
    text-align: center;
  }

</style>