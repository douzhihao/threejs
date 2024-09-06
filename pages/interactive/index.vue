<template>
	<view class="container">
		<div class="mainUi" v-show="showMainUI">
			<div class="loading" v-show="showLoading" @click="loadingClick">
				<div class="loadingText">加载中... </div>
				<div class="loadingRange">
					<div class="rangeWidth" :style="{ width: loadingIndex + '%' }"></div>
				</div>
			</div>
			<div class="selectRole" v-show="showSelectRole">
				<div class="selectRoleTitle">基础英语教育师范技能虚拟仿真实验</div>
				<div class="selectClick">
					<image src="../../static/ui/select01.png" mode="widthFix" @click="selectRoleClick"></image>
					<image src="../../static/ui/select02.png" mode="widthFix" @click="selectRoleClick"></image>
				</div>
			</div>
			<div class="micCheck" v-show="showMicCheck">
				<div class="startCheck" v-show="showMicCheckIndex == 1">
					<p>本课程需要使用语音完成实验内容 </p>
					<p>点击下方按钮以检测麦克风是否可用</p>
					<div class="imgBtn" @click="micCheckClick(2)">
						<image src="../../static/ui/check01.png" mode="widthFix"></image>
					</div>
				</div>
				<div class="checking" @click="micCheckClick(3)" v-show="showMicCheckIndex == 2">
					<p>检测中...</p>
					<image src="../../static/ui/checking.png" mode="widthFix"></image>
				</div>
				<div class="reCheck" v-show="showMicCheckIndex == 3">
					<p>未检测到麦克风，</p>
					<p>将无法获取录音评分，</p>
					<p>确认继续实验吗？</p>
					<div class="imgBtnBox">
						<image src="../../static/ui/recheck.png" mode="widthFix" @click="micCheckClick(2)"></image>
						<image src="../../static/ui/continue.png" mode="widthFix" @click="micCheckClick(4)"></image>
					</div>
				</div>
				<div class="checkDone" v-show="showMicCheckIndex == 4">
					<p>检测成功！</p>
					<p>立即开启课程体验。</p>
					<div class="imgBtn" @click="micCheckClick(5)">
						<image src="../../static/ui/startLearn.png" mode="widthFix"></image>
					</div>
				</div>
			</div>
			<div class="selectLesson" v-show="showSelectLesson">
				<p>基础英语教育师范技能虚拟仿真实验</p>
				<div class="selectBox">
					<image src="../../static/ui/lesson01.png" mode="widthFix" @click="showMainUIClick"></image>
					<image src="../../static/ui/lesson02.png" mode="widthFix"></image>
					<image src="../../static/ui/lesson03.png" mode="widthFix"></image>
				</div>
			</div>
		</div>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				// 页面数据
				loadingIndex:0,
				showLoading: true,
				showSelectRole: false,
				showMicCheck: false,
				showMicCheckIndex: 1,
				showSelectLesson: false,
				showMainUI: true,
			};
		},
		onLoad() {
			// 页面加载时执行
			let that  = this;
			let timer = setInterval(function(){
				if(that.loadingIndex == 100){
					clearInterval(timer);
					that.showLoading = false;
					that.showSelectRole = true;
				}
				that.loadingIndex = that.loadingIndex+1;
			},20)
		},
		methods: {
			// 页面方法
			loadingClick() {
				this.showLoading = false;
				this.showSelectRole = true;
			},
			selectRoleClick() {
				this.showSelectRole = false;
				this.showMicCheck = true
			},
			micCheckClick(index) {
				if (index == 5) {
					this.showMicCheck = false;
					this.showSelectLesson = true;
				}
				this.showMicCheckIndex = index;
			},
			showMainUIClick(){
				this.showMainUI = false;
				uni.navigateTo({
					url: '/pages/index/index'
				  });
			}
		}
	};
</script>

<style scoped lang="scss">
	.container {
		/* 页面样式 */
		width: 100vw;
		height: 100vh;
		.mainUi {
			width: 100vw;
			height: 100vh;
			overflow: hidden;
			display: flex;
			justify-content: center;
			align-items: center;
			background: url(../../static/ui/main_bg.jpg) no-repeat 100% 100%;
			.loading {
				text-align: center;
				width: 50vw;
				position: absolute;
				bottom: 10vh;
				font-size: 75rpx;
				font-weight: bold;
				color: #FDD718;

				.loadingRange {
					width: 50vw;
					height: 50rpx;
					border: 4rpx solid #FDD718;
					border-radius: 29rpx;
					margin-top: 33rpx;
					overflow: hidden;
					padding: 4rpx;

					.rangeWidth {
						width: 50%;
						height: 48rpx;
						background-color: #FDD718;
						border-radius: 25rpx;
					}
				}
			}

			.selectRole {
				width: 60vw;

				.selectRoleTitle {
					font-family: Source Han Sans;
					font-size: 80rpx;
					font-weight: bold;
					line-height: normal;
					text-align: center;
					letter-spacing: 0em;
					color: #FFFFFF;
					margin-bottom: 5vh;
				}

				.selectClick {
					display: flex;
					justify-content: space-between;
					align-items: center;

					image {
						width: 45%;
					}
				}
			}

			.micCheck {
				width: 1600rpx;
				height: 760rpx;
				border-radius: 20rpx;
				background-color: #434343;
				display: flex;
				justify-content: center;
				align-items: center;

				.startCheck {
					width: 100%;
					display: flex;
					flex-direction: column;
					justify-content: center;
					align-items: center;

					p {
						font-family: Source Han Sans;
						font-size: 84rpx;
						font-weight: bold;
						line-height: normal;
						text-align: center;
						letter-spacing: 0em;
						color: #fff;
					}

					.imgBtn {
						width: 540rpx;
						margin-top: 4vh;

						image {
							width: 100%;
						}
					}
				}

				.checking {
					p {
						font-family: Source Han Sans;
						font-size: 128rpx;
						font-weight: bold;
						line-height: normal;
						text-align: center;
						color: #FFFFFF;
					}

					image {
						margin-top: 2vh;
						width: 1600rpx;
					}
				}

				.reCheck {
					width: 100%;

					p {
						font-family: Source Han Sans;
						font-size: 64rpx;
						font-weight: bold;
						text-align: center;
						color: #fff;
					}

					.imgBtnBox {
						display: flex;
						flex-direction: row;
						justify-content: space-around;
						margin-top: 5vh;

						image {
							width: 500rpx;
						}
					}
				}

				.checkDone {
					width: 100%;
					display: flex;
					flex-direction: column;
					justify-content: center;
					align-items: center;

					p {
						font-family: Source Han Sans;
						font-size: 84rpx;
						font-weight: bold;
						text-align: center;
						color: #FFFFFF;
					}

					.imgBtn {
						width: 540rpx;
						margin-top: 4vh;

						image {
							width: 100%;
						}
					}
				}
			}

			.selectLesson {
				p {
					font-family: Source Han Sans;
					font-size: 116rpx;
					font-weight: bold;
					text-align: center;
					color: #FFFFFF;
				}

				.selectBox {
					display: flex;
					flex-direction: row;
					justify-content: space-around;
					margin-top: 4vh;

					image {
						width: 1000rpx;
					}
				}
			}
		}
	}
</style>