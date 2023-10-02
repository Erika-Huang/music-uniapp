<template>
	<view class="wrapper">
		<view class="song-name">
			{{songName}}
		</view>
		<view class="cover-lyric">
			<swiper class="swiper">
				<!-- 上面旋转图标 -->
				<swiper-item>
					<view class="cover">
						<image :src="coverImg" class="cover-img" mode="" :class="playandpausestate == 'pause' ? 'cover-img-rotate':''"></image>
					</view>
				</swiper-item>
				<!-- 下面歌词展示 -->
				<swiper-item>
					<scroll-view :scroll-top="scrollTop" scroll-y="true" class="lyric">
						<view class="line" v-for="(item,index) in lyricArr" :key="index">
							{{item[1]}}
						</view>
					</scroll-view>
				</swiper-item>
			</swiper>
		</view>
		<!-- 进度条 -->
		<view class="progress">
			<text>{{playTime}}</text>
			<slider :value="val" @change="sliderChange" min="0" :max="max"/>
			<text>{{totalTime}}</text>
		</view>
		<view class="control">
			<image src="@/static/images/prev.png" mode="" @click="changeSong('prev')"></image>
			<image :src="playandpauseimgurl" mode="" @click="playAndPause"></image>
			<image src="@/static/images/next.png" mode="" @click="changeSong('next')"></image>
			
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				scrollTop: 0,
				// 服务器歌曲的地址
				httpSongUrl: 'https://music.163.com/song/media/outer/url?id=',
				// 当前播放音乐的id
				currentId:"",
				// 播放暂停控制
				playandpausestate:"play",
				innerAudioContext:"",
				playandpauseimgurl:require('@/static/images/pause.png'),
				// 保存本地存储的数据
				songsArr:[],
				currentSongId:"",
				// 当前音乐的索引
				songsArrCurrentIndex:0,
				// 进度条相关的数据
				totalTime:"",
				playTime:"",
				val:"",
				max:"",
				// 封面和歌名
				songName:"",
				lyricArr:[],
				coverImg:""
			}	
		},
		methods: {
			// 补零函数
			zero(n){
				return n < 10 ? "0"+n : n
			},
			// 播放音乐事件
			playMusic(){
				let _this = this
				_this.innerAudioContext = uni.createInnerAudioContext();
				_this.innerAudioContext.autoplay = true;
				_this.innerAudioContext.src = this.httpSongUrl+this.currentId
				// 音频进入可播放状态
				_this.innerAudioContext.onCanplay(function(){
					// 获取音频的总时长
					let duration = _this.innerAudioContext.duration
					console.log(duration,"音频总时长");
				})
				// 音频播放状态
				_this.innerAudioContext.onPlay(function(){
					let duration = _this.innerAudioContext.duration
				})
				// 监听音频的播放时长和总时长
				_this.innerAudioContext.onTimeUpdate(function(){
					let duration = _this.innerAudioContext.duration;
					let currentTime = _this.innerAudioContext.currentTime
					// 处理duration为NaN
					if(!duration){
						duration = 0
					}
					// console.log("duration:",duration)
					// console.log("currentTime:",currentTime)
					// 时间格式的处理,处理成分钟和秒数的形式 20:30，如果小于10进行补0操作
					let M = _this.zero(Math.floor(duration / 60))  
					let S = _this.zero(Math.floor(duration % 60)) 
					// 当前播放时长的处理
					let currentM = _this.zero(Math.floor(currentTime / 60))
					let currentS = _this.zero(Math.floor(currentTime % 60))
					_this.totalTime = M+":"+S
					_this.playTime = currentM+":"+currentS
					// 赋值进度条的取值区间
					_this.val = currentTime
					_this.max = duration
				})
			},
			
			// 滑块拖动
			sliderChange(e){
				console.log(e.detail.value);
				// 把滑块和音乐对象关联起来
				this.innerAudioContext.seek(e.detail.value)
			},
			// 播放和暂停
			playAndPause(){
				let stata = this.playandpausestate;
				if(stata == "play"){
					this.innerAudioContext.pause()  //不懂
					this.playandpausestate = "pause"
					this.playandpauseimgurl = require('@/static/images/play.png')
				}else if(stata === "pause"){
					this.innerAudioContext.play()  //不懂
					this.playandpausestate = "play"
					this.playandpauseimgurl = require('@/static/images/pause.png')
				}
				
			},
			// 上一首下一首切换
			changeSong(state){
				let _this = this;
				let songsArr = _this.songsArr;
				console.log(songsArr);
				for(let i = 0;i<songsArr.length;i++){
					if(songsArr[i].id === _this.currentId){
						_this.songsArrCurrentIndex = i
					}
				}
				// 上一曲
				if(state == "prev"){
					if(_this.songsArrCurrentIndex > 0){
						_this.songsArrCurrentIndex--
					}
				}
				// 下一区
				if(state == "next"){
					if(_this.songsArrCurrentIndex < songsArr.length-1){
						_this.songsArrCurrentIndex++
					}
				}
				_this.innerAudioContext.stop()
				console.log(_this.songsArrCurrentIndex,"testid获取");
				console.log(_this.songsArr);
				// 更新当前音乐的id
				_this.currentId = _this.songsArr[_this.songsArrCurrentIndex].id
				console.log(_this.currentId,"-----");
				// 播放歌曲
				_this.playMusic()
				// 获取歌词
				_this.getLyric()
			},
			// 歌词显示
			getLyric(){
				let _this = this;
				uni.request({
					url:"http://47.108.157.232:8889/getMusicInfo?id="+_this.currentId,
					success:(res) => {
						console.log(res.data);
						// 获取歌词字符串
						let lyricStr = res.data.music_lyric
						// 把字符串以换行符分割转为数组
						let lyricArr = lyricStr.split("\n")
						// 如果数组的最后一个元素是空,那么就删除掉
						if(lyricArr[lyricArr.length-1] == ""){
							lyricArr.pop()
						}
						// 把数组中的时间和歌词分分开,找出时间
						let reg = /\[\d{2}:\d{2}.\d{2,3}\]/   //[00:00:00]
						// 定义空数组来存储最终的结果
						let finallyLyric = []
						for(let i in lyricArr){
							let lyricLine = lyricArr[i]
							// console.log(lyricLine);
							// 获取匹配的时间字符串
							let time = lyricLine.match(reg)[0]
							// 去掉[]
							let time2 = time.substring(1,time.length - 2)
							// 时间转换
							let times = parseInt(time2.split(":")[0]) * 60 + parseFloat(time2.split(":")[1])
							let lyricLine2 = lyricLine.replace(time,"")
							// 把第一行歌词和时间处理后的结果存入数组中
							if(lyricLine2 != ""){
								finallyLyric.push([times,lyricLine2])
							}
						}
						// 把最后的结果赋值给lyricArr
						_this.lyricArr = finallyLyric
						// 对data歌名/封面进行赋值
						_this.songName = res.data.music_name
						_this.coverImg = res.data.image_url
					}
				})
			},
			// 取出本地存储的数据
			getSongsArrFormStorage(){
				let that = this;
				console.log(that,"that-------");
				uni.getStorage({
					key:"songsArr",
					success(res) {
						console.log(res);
						that.songsArr = res.data;
						console.log(that.songsArr,"从本地存储拿到的数据");
					}
				})
			},
			// 自动播放下一首歌曲
			autoPlayNext(){
				let _this = this 
				_this.innerAudioContext.onEnded(function(){
					console.log(666);
					_this.changeSong("next")
				})
			}
		},
		onLoad(option) {
			// 拿到跳转过来的音乐id
			this.currentId = option.id
			this.innerAudioContext = uni.createI
			this.playMusic()
			this.getLyric()
			this.getSongsArrFormStorage()
			this.autoPlayNext()
		},
		onUnload() {
			// 离开页面的时候暂停歌
			this.innerAudioContext.stop();
		},
	}
</script>

<style lang="less">
	// 754f4b   483f5e
	@keyframes zhuan {
		from {
			transform: rotate(0deg);
		}

		to {
			transform: rotate(360deg);
		}
	}

	.wrapper {
		width: 750rpx;
		height: 1100rpx;
		background: radial-gradient(circle, #ecdc51, #483f5e);
	
	.song-name {
			text-align: center;
			padding: 70rpx 0rpx;
			font-size: 50rpx;
			color: #fff;
		}

		//cover-lyrice
		.cover-lyric {
			width: 100%;
			height: 600rpx;

			.swiper {
				width: 100%;
				height: 600rpx;

				.cover {
					width: 100%;
					height: 100%;
					// background-color: blue;
					display: flex;
					justify-content: center;
					align-items: center;

					.cover-img {
						width: 400rpx;
						height: 400rpx;
						border-radius: 200rpx;
						animation: zhuan 5s linear infinite;
						
					}
					.cover-img-rotate {
						animation-play-state: paused
					}
				}

				.lyric {
					width: 100%;
					height: 600rpx;

					// background-color: #fff;
					.line {
						height: 50rpx;
						text-align: center;
						line-height: 50rpx;
						font-size: 16rpx;
						color: #fff;
					}
				}
			}
		}

		//end cover-lyrice
		.progress {
			display: flex;
			justify-content: space-around;
			align-items: center;
			color: #fff;

			slider {
				width: 350rpx;
			}
		}

		.control {
			display: flex;
			justify-content: space-around;
			align-items: center;
			margin-top: 50rpx;

			image {
				width: 80rpx;
				height: 80rpx;
			}

			.middle {
				width: 100rpx;
				height: 100rpx;
			}
		}
	}
</style>
