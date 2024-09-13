<template>
	<view class="container">
		<div class="tipsCover" v-show="!showMainUI">
			<div class="cover"></div>
			<image class="rotateTips" src="../../static/ui/rotate.png"></image>
			<text class="textTips">为了更好地体验，请使用横屏浏览</text>
		</div>
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
					<image src="../../static/ui/select01.png" mode="widthFix" @click="selectRoleClick(1)"></image>
					<image src="../../static/ui/select02.png" mode="widthFix" @click="selectRoleClick(2)"></image>
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
				<!-- <div class="checking" @click="micCheckClick(3)" v-show="showMicCheckIndex == 2"> -->
				<div class="checking" v-show="showMicCheckIndex == 2">
					<p>检测中...</p>
					<div class="checkingBox">
						<div class="boxWindow">
							<image src="../../static/ui/checking.png" mode="widthFix"></image>
							<image src="../../static/ui/checking.png" mode="widthFix"></image>
							<image src="../../static/ui/checking.png" mode="widthFix"></image>
							<image src="../../static/ui/checking.png" mode="widthFix"></image>
						</div>
					</div>
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
					<p >检测成功！</p>
					<p >立即开启课程体验。</p>
					<!-- <p @click="recording()">检测成功！</p>
					<p @click="recorded()">立即开启课程体验。</p>
					<p @click="playVoice()">立即开启课程体验。</p> -->
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
	import {
		HZRecorder
	} from '../../Utils/HZRecorder.js';
	import VConsole from 'vconsole';
	export default {
		data() {
			return {
				// 页面数据
				loadingIndex: 0,
				showLoading: true,
				showSelectRole: false,
				showMicCheck: false,
				showMicCheckIndex: 1,
				showSelectLesson: false,
				showMainUI: true,
				isShowTips: true,
				tips: '',
				recorder: null,
				innerAudioContext: '',
				roleID:""
			};
		},
		onHide() {

		},
		onShow() {
			this.orientationChange();
			// location.reload();
			this.loadingIndex = 0
			this.showLoading = true
			this.showSelectRole = false
			this.showMicCheck = false
			this.showMicCheckIndex = 1
			this.showSelectLesson = false
			this.showMainUI = true
			this.isShowTips = true
			this.tips = ''
			this.roleID=""
			this.init()
		},
		onLoad(option) {
			this.init();
			var ua = window.navigator.userAgent.toLowerCase();
			// uni.getSystemInfo({
			//   success: function (res) {
			//     console.log(`屏幕宽度: ${res.windowWidth}, 屏幕高度: ${res.windowHeight}`);
			//   }
			// });
			if(option.editor){
				new VConsole();
			}
			
		},
		mounted() {
			let that = this;
			that.innerAudioContext = uni.createInnerAudioContext();
			that.innerAudioContext.autoplay = true;
			// this.innerAudioContext.src = 'https://bjetxgzv.cdn.bspapp.com/VKCEYUGU-hello-uniapp/2cc220e0-c27a-11ea-9dfb-6da8e309e0d8.mp3';
			that.innerAudioContext.onPlay(() => {
				console.log('开始播放');
			});
			that.innerAudioContext.onError((res) => {
				console.log(res.errMsg);
				console.log(res.errCode);
			});
		},
		methods: {
			init(){
				// 页面加载时执行
				let that = this;
				let timer = setInterval(function() {
					if (that.loadingIndex == 100) {
						clearInterval(timer);
						that.showLoading = false;
						that.showSelectRole = true;
					}
					that.loadingIndex = that.loadingIndex + 1;
				}, 20)
				that.orientationChange();
			},
			// 页面方法
			loadingClick() {
				this.showLoading = false;
				this.showSelectRole = true;
			},
			selectRoleClick(id) {
				
				this.roleID = id;
				this.showSelectRole = false;
				this.showMicCheck = true;
			},
			micCheckClick(index) {
				let that = this;
				// console.log(index)
				if (index == 5) {
					that.showMicCheck = false;
					that.showSelectLesson = true;
				}
				that.showMicCheckIndex = index;
				if(index == 2){
					that.$nextTick(() => {
						try {
							window.AudioContext = window.AudioContext || window.webkitAudioContext;
							navigator.getUserMedia = navigator.getUserMedia || navigator.webkitGetUserMedia;
							window.URL = window.URL || window.webkitURL;
							audio_context = new AudioContext;
							console.log('navigator.getUserMedia ' + (navigator.getUserMedia ? 'available.' :
								'not present!'));
						} catch (e) {
							console.log('No web audio support in this browser!');
							
						}
						if (navigator.getUserMedia) {
							navigator.getUserMedia({
								audio: true
							}, function(stream) {
								that.recorder = new HZRecorder(stream)
								console.log('初始化完成');
								that.showMicCheckIndex = 4;
							}, function(e) {
								console.log('No live audio input: ' + e);
							
							});
						} else {
							that.showMicCheckIndex = 3;
							uni.showModal({
								title: '提示',
								content: '当前设备不支持录音',
								success: function(res) {
									if (res.confirm) {
										// console.log('用户点击确定');
									} else if (res.cancel) {
										// console.log('用户点击取消');
									}
								}
							});
						}
					})
				}
			},
			showMainUIClick() {
				this.showMainUI = false;
				uni.navigateTo({
					url: '/pages/index/index?roleID='+this.roleID
				});
			},
			recording(){
				let that = this;
				navigator.getUserMedia({
					audio: true
				}, function(stream) {
					that.recorder = new HZRecorder(stream)
					console.log('初始化完成');
				}, function(e) {
					console.log('No live audio input: ' + e);
				});
				that.recorder.start();
			},
			recorded(){
				this.recorder.stop();
				this.recorder.getBlob();
				
				var mp3Blob = this.recorder.getBlob();
				// console.log(mp3Blob);
				let files = new File([mp3Blob], 'input.wav', {
					type: 'audio/wav',
					lastModified: Date.now()
				});
				var fd = new FormData();
				// fd.append('file', files);
				fd.append('multipartFile', files);
				
				var url = window.URL.createObjectURL(mp3Blob);
				console.log(url);
			},
			playVoice() {
				console.log('播放录音');
				
				if (this.voicePath) {
					this.innerAudioContext.src = this.voicePath;
					this.innerAudioContext.play();
				}
			},
			browser() {
				var ua = window.navigator.userAgent.toLowerCase();
				// console.log(ua)
				if (ua.match(/MicroMessenger/i) == 'micromessenger') {
					// "这是微信浏览器"
					if (this.screenSizeGet() == "竖屏") {
						if (this.phoneSystem() == "android") {
							// this.tips = "为了更好地体验，请开启微信横屏模式\n关闭方向锁定，使用横屏浏览";
						} else if (this.phoneSystem() == "ios") {
							// this.tips = "为了更好地体验，请关闭方向锁定\n使用横屏浏览";
						}
						this.showMainUI = false;
					} else {
						this.showMainUI = true;
					}
				} else {
					// 普通浏览器
					if (this.screenSizeGet() == "竖屏") {
						// this.tips = "为了更好地体验，请关闭方向true锁定\n使用横屏浏览";
						this.showMainUI = false;
					} else {
						this.showMainUI = true;
					}
				}
			},
			screenSizeGet() {
				let screenNumber = "";
				let that = this;
				uni.getSystemInfo({
					success: res => {
						if (res.screenHeight > res.screenWidth) {
							screenNumber = "竖屏";
						} else {
							screenNumber = "横屏";
						}
					}
				})
				return screenNumber;
			},
			phoneSystem() {
				var phone_System = "android";
				switch (uni.getSystemInfoSync().platform) {
					case 'android':
						phone_System = "android";
						break;
					case 'ios':
						phone_System = "ios";
						break;
				}
				return phone_System
			},
			orientationChange() {
				this.browser();
				// console.log(this.showMainUI,"-----------");
				window.addEventListener("orientationchange", () => {
					if (window.orientation == 180 || window.orientation == 0) {
						console.log("横屏");
						// this.tips = "为了更好地体验，请使用横屏浏览";
						this.showMainUI = true;
						location.reload();
					} else if (window.orientation == 90 || window.orientation == -90) {
						console.log("竖屏");
						this.showMainUI = false;
						location.reload();
					}
				})
				
			}
		}
	};
</script>

<style scoped lang="scss">
	@media screen and (max-width: 999px) {
		.container {
			/* 页面样式 */
			width: 100vw;
			height: 100vh;
			.tipsCover{
				width: 100vw;
				height: 100vh;
				.cover {
					position: absolute;
					width: 100%;
					height: 100%;
					top: 0;
					left: 0;
					background-color: #000000;
					z-index: 9999;
				}
				
				.rotateTips {
					position: absolute;
					width: 80px;
					height: 80px;
					top: 40%;
					left: 50%;
					transform: translate(-50%, -50%);
					z-index: 9999;
				}
				
				.textTips {
					position: absolute;
					width: 100%;
					height: 200px;
					top: 60%;
					font-size: 20px;
					color: #FFFFFF;
					text-align: center;
					transform: translateY(-50%);
					/* margin-top: 100rpx; */
					z-index: 9999;
				}
			}
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
					font-size: 24rpx;
					font-weight: bold;
					color: #FDD718;

					.loadingRange {
						width: 50vw;
						height: 10rpx;
						border: 4rpx solid #FDD718;
						border-radius: 29rpx;
						margin-top: 10rpx;
						overflow: hidden;
						padding: 4rpx;

						.rangeWidth {
							width: 50%;
							height: 10rpx;
							background-color: #FDD718;
							border-radius: 25rpx;
						}
					}
				}

				.selectRole {
					width: 60vw;

					.selectRoleTitle {
						font-family: Source Han Sans;
						font-size: 20rpx;
						font-weight: bold;
						line-height: normal;
						text-align: center;
						letter-spacing: 0em;
						color: #FFFFFF;
						margin-bottom: 2vh;
					}

					.selectClick {
						display: flex;
						justify-content: space-between;
						align-items: center;

						image {
							width: 180rpx;
							height: 225rpx;
						}
					}
				}

				.micCheck {
					width: 440rpx;
					height: 200rpx;
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
							font-size: 20rpx;
							font-weight: bold;
							line-height: normal;
							text-align: center;
							letter-spacing: 0em;
							color: #fff;
						}

						.imgBtn {
							width: 200rpx;
							margin-top: 10vh;
							display: flex;
							flex-direction: column;
							justify-content: center;
							align-items: center;
							image {
								width: 140rpx;
								height: 50rpx;
							}
						}
					}

					.checking {
						p {
							font-family: Source Han Sans;
							font-size: 28rpx;
							font-weight: bold;
							line-height: normal;
							text-align: center;
							color: #FFFFFF;
						}
						.checkingBox{
							width: 300rpx;
							height: 80rpx;
							overflow: hidden;
							position: relative;
							.boxWindow{
								// width: 3000rpx;
								display: flex;
								flex-direction: row;
								position: absolute;
								left: 0;
								animation: slideToLeft 9s linear infinite;
								image {
									display: inline-block;
									margin-top: 2vh;
									width: 404rpx;
									height: 62rpx;
								}
							}
						}


					}

					.reCheck {
						width: 100%;

						p {
							font-family: Source Han Sans;
							font-size: 24rpx;
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
								width: 140rpx;
								height: 50rpx;
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
							font-size: 24rpx;
							font-weight: bold;
							text-align: center;
							color: #FFFFFF;
						}

						.imgBtn {
							display: flex;
							flex-direction: row;
							justify-content: space-around;
							margin-top: 5vh;
							image {
								width: 140rpx;
								height: 50rpx;
							}
						}
					}
				}

				.selectLesson {
					p {
						font-family: Source Han Sans;
						font-size: 24rpx;
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
							width: 200rpx;
							height: 250rpx;
						}
					}
				}
			}
		}

	}
	@keyframes slideToLeft {
	  from {
	    left: 0; /* 初始位置 */
	  }
	  to {
	    left: -1200rpx; /* 向左移动到不可见 */
	  }
	}

	@media screen and (min-width:1000px) {
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
							width: 1000rpx;
							height: 618rpx;
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
								width: 540rpx;
								height: 168rpx;
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

						.checkingBox{
							width: 800rpx;
							height: 200rpx;
							overflow: hidden;
							position: relative;
							.boxWindow{
								// width: 3000rpx;
								display: flex;
								flex-direction: row;
								position: absolute;
								left: 0;
								animation: slideToLeft 9s linear infinite;
								image {
									display: inline-block;
									margin-top: 2vh;
									width: 800rpx;
									height: 200rpx;
								}
							}
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
								width: 540rpx;
								height: 168rpx;
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
								width: 540rpx;
								height: 168rpx;
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
							height: 1240rpx;
						}
					}
				}
			}
		}
	}
</style>