<!--  -->
<template>
  <div class="app-container">
		<div class="layout">
			<div class="alert">
        <el-alert
          title="基于canvas的五子棋"
					description="人机大战，人先走，执黑子"
          type="info"
					effect="dark"
					show-icon>
        </el-alert>
      </div>
			<div class="gnav">
				<el-button type="primary" @click="start()" round >重新开始</el-button>
				<el-button type="success" @click="surrender()" round >认输</el-button>
			</div>
				<canvas id="chess" width="450px" height="450px"></canvas>
		</div>
  </div>
</template>

<script>
let chess
let context
export default {
  data() {
		return {
			me: true,
			chessBoard: [],
			over: false,
			//  AI 赢法的数组
			wins: [],
			// 赢法的统计数组
			myWin: [],  //  我方
			computerWin: [],  //计算机方
			count: 0, // 赢法总类的索引
		}
	},

  components: {},

  mounted() {
		this.init();
	},

  methods: {
		init() {
			chess = document.getElementById("chess");
			context = chess.getContext('2d');
			chess.addEventListener('click', this.setPiece)
			this.over = false
			this.me = true
			this.count = 0
			// 初始化棋盘
			for(var i = 0 ; i < 15 ;i++){
				this.chessBoard[i] = [];
				for(var j = 0 ; j < 15 ; j++){
					this.chessBoard[i][j] = 0 ;
				}
			}

			for(var i = 0 ; i < 15 ; i++){
				this.wins[i] = [];
				for(var j = 0 ; j < 15 ; j++) {
					this.wins[i][j] = [];
				}
			}

			// 初始化所有的横线
			for(var i = 0 ; i < 15 ; i++){
				for(var j = 0 ; j < 11 ; j++){
					for(var k = 0 ; k < 5 ; k++){
						this.wins[i][j+k][this.count] = true ;
					}
					this.count++ ;
				}
			}

			// 统计所有的竖线
			for(var i = 0 ; i < 15 ; i++){
				for(var j = 0 ; j < 11 ; j++){
					for(var k = 0 ; k < 5 ; k++){
						this.wins[j+k][i][this.count] = true ;
					}
					this.count++ ;
				}
			}

			// 统计所有的斜线
			for(var i = 0 ; i < 11 ; i++){
				for(var j = 0 ; j < 11 ; j++){
					for(var k = 0 ; k < 5 ; k++){
						this.wins[i+k][j+k][this.count] = true ;
					}
					this.count++ ;
				}
			}


			// 统计所有的反斜线
			for(var i = 0 ; i < 11 ; i++){
				for(var j = 14 ; j > 3 ; j--){
					for(var k = 0 ; k < 5 ; k++){
						this.wins[i+k][j-k][this.count] = true ;
					}
					this.count++ ;
				}
			}

			console.log("count:" + this.count);

			for(var i = 0 ; i < this.count ; i++){
				this.myWin[i]  = 0 ; 
				this.computerWin[i] = 0 ;
			}

			context.strokeStyle = "grey";

			var logo = new Image();
			logo.src = require("./background/back.jpg");
			logo.onload = () => {
				context.drawImage(logo , 0 , 0 , 450 , 450);
				// 画线
				for(var i = 0 ; i < 15 ; i++){
					context.moveTo(15 + i * 30 , 15);
					context.lineTo(15 + i * 30 , 435) ;
					context.stroke();
					context.moveTo(15 , 15 + i * 30);
					context.lineTo(435 , 15 + i * 30);
					context.stroke();
				}
			}
		},

		surrender() {
			setTimeout(() => {//延迟弹窗,确定:重玩
					this.$msgbox(`你已认输！计算机获胜！` , '认输！！', {
						confirmButtonText: '确定',
					})
			}, 100);
			this.over = true
		},

		start() {
			this.init()
		},
		setPiece(e) {
			if(this.over)
				return ;
			if(!this.me)
				return ;
			var x = e.offsetX ; 
			var y = e.offsetY ;
			var i = Math.floor(x/30);
			var j = Math.floor(y/30);
			if(this.chessBoard[i][j] == 0){
				this.oneStep(i, j, this.me);
				this.chessBoard[i][j] = 1 ; 
				for(var k = 0 ; k < this.count ; k++){
					if(this.wins[i][j][k]){
						this.myWin[k]++ ;
						this.computerWin[k] = 6;  // 表示计算机在第k种赢法种类上不可能赢
						if(this.myWin[k] == 5){
							setTimeout(() => {//延迟弹窗,确定:重玩
								this.$msgbox(`你赢了!` , '获胜！！', {
									confirmButtonText: '确定',
								})
						}, 100);
							this.over = true
						}
					}
				}
				if(!this.over){
					this.me = !this.me ;
					this.computerAI();
				}
			}
		},

		oneStep(i , j, me){
			context.beginPath();
			context.arc(15 + i*30,15 + j*30,13,0 ,2 * Math.PI);
			context.closePath();
			var gradient = context.createRadialGradient(15 + i*30 + 2, 15 + j*30 - 2, 13, 15 + i*30 + 2, 15 + j*30 - 2, 0);
			if(me){
				gradient.addColorStop(0,"#0A0A0A");
				gradient.addColorStop(1,"#636766");
			} else {
				gradient.addColorStop(0,"#D1D1D1");
				gradient.addColorStop(1,"#F9F9F9");
			}

			context.fillStyle = gradient;
			context.fill();
		},

		computerAI(){
			var myScore = [] ;
			var computerScore = [];
			var max = 0 ; // 保存最高的分数
			var u = 0 , v = 0 ; //保存最高分点的坐标
			for(var i = 0 ; i < 15 ; i++){
				myScore[i] = [];
				computerScore[i] = [];
				for(var j = 0 ; j < 15 ; j++){
					myScore[i][j] = 0 ; 
					computerScore[i][j] = 0 ;
				}
			}

			for(var i = 0 ; i < 15 ; i++){
				for(var j = 0 ; j < 15 ; j++){
					if(this.chessBoard[i][j] == 0){
						for(var k = 0 ; k < this.count ;k++){
							if(this.wins[i][j][k] ){
								if(this.myWin[k] == 1){
									myScore[i][j] += 200 ;}
								else if(this.myWin[k] == 2){
									myScore[i][j] += 400;
								}
								else if(this.myWin[k] == 3){
									myScore[i][j] += 2000;
								}
								else if(this.myWin[k] == 4){
									myScore[i][j] += 10000;
								}

								if(this.computerWin[k] == 1){
									computerScore[i][j] += 220;
								}
								else if(this.computerWin[k] == 2){
									computerScore[i][j] +=420 ;
								}
								else if(this.computerWin[k] == 3){
									computerScore[i][j] +=2100
								}
								else if(this.computerWin[k] == 4){
									computerScore[i][j] +=20000;
								}
							}
						}
						if(myScore[i][j] > max){
							max = myScore[i][j];
							u = i ; 
							v = j ;
						}
						else if(myScore[i][j] == max){
							if(computerScore[i][j] > computerScore[u][v]){
								u = i ; 
								v = j ;
							}
						}

						if(computerScore[i][j] > max){
							max = computerScore[i][j];
							u = i ; 
							v = j ;
						} else if (computerScore[i][j] == max){
							if(myScore[i][j] > myScore[u][v]){
								u = i ; 
								v = j ;
							}
						}
					}
				}
			}
			this.oneStep(u,v,false);
			this.chessBoard[u][v] = 2 ;

			for(var k = 0 ; k < this.count ; k++){
				if(this.wins[u][v][k]){
					this.computerWin[k]++ ;
					this.myWin[k] = 6;  // 表示计算机在第k种赢法种类上不可能赢
					if(this.computerWin[k] == 5){
						// window.alert("计算机赢了");
						setTimeout(() => {//延迟弹窗,确定:重玩
								this.$msgbox(`计算机获胜！` , '失败！！', {
									confirmButtonText: '确定',
								})
							// this.init();
						}, 100);
						this.over = true
					}
				}
			}
			if(!this.over){
				this.me = !this.me ;
			}
		}
	}
}

</script>
<style rel='stylesheet/scss' lang='scss' scoped>
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
	.alert {
    width: 550px;
  }
	.gnav {
    padding-top: 20px;
    height: 10px;
    text-align: center;
  }
	canvas {
		display: block;
		margin:50px auto;
		box-shadow: -2px -2px 2px #EFEFEF , 5px 5px 5px #B9B9B9;
	}
</style>