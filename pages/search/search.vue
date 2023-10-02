<template>
	<view class="search">
		<!-- 上面搜索框 -->
		<view class="search-head">
			<input type="text" placeholder="请输入歌曲" v-model="songName"/>
			<img src="@/static/images/find.png" alt="" @click="searchMusic">
		</view>
		<!-- 下面展示 -->
		<!-- 猜你喜欢 -->
		<view class="search-main" v-if="songs.length === 0">
			<view class="title">
				猜你喜欢
			</view>
			<view class="song-list">
				<view class="song-item" v-for="(item,index) in hotSongs" :key="index">
					<strong>{{index+1}}</strong>
					<view class="item-name">
						{{item}}
					</view>
					<view class="item-num">
						1230
					</view>
				</view>
			</view>
		</view>
		
		<!-- 搜索列表展示 -->
		<view class="search-show" v-else>
			<view class="song-list" v-for="(item,index) in songs" :key="index">
				<view class="song-left">
					<view class="song-name">
						{{item.music_name}}
					</view>
					<view class="song-dec">
						<image src="@/static/images/11.png" mode=""></image>
						<text>{{item.music_author}}-{{item.music_name}}</text>
					</view>
				</view>
				<image class="play" src="@/static/images/play2.png" mode=""></image>
			</view>
		</view>
		
	</view>
</template>

<script>
	export default {
		data() {
			return {
				songName: "",
				songs: [],
				hotSongs: [],
			}
		},
		methods: {
			// 搜索歌曲
			searchMusic() {
				let _this = this;
				if (_this.songName !== ""){
				  uni.request({	
					  url:"http://47.108.157.232:8889/getMusicList?title="+_this.songName,
					    success(res) {
							// console.log(res.data);
							_this.songs = res.data;
						}
				  })
				}
			},
				// 获取猜你喜欢
				getHotSongs(){
					let _this = this;
					 uni.request({
						 url:"http://47.108.157.232:8889/getRecommend",
						 success: (res) => {
							// console.log(res.data,"1111"); //包含所有歌名数组
							// console.log(_this.getRandomNum());
							let num= _this.getRandomNum(res.data)
							for(let index in num){
								// console.log(index,"1111");
								_this.hotSongs.push(res.data[num[index]])
							}
							console.log(_this.hotSongs);
						 }
				})
			},
			// 获取指定数字之间的随机整数
			getInt(min,max){
				return Math.floor(Math.random()*(max-min+1)+min)
				},
			// 获取随机数
			getRandomNum(arr){
				let num1 = this.getInt(0,arr.length-1)
				let num2 = this.getInt(0,arr.length-1)
				let num3 = this.getInt(0,arr.length-1)
				return [num1,num2,num3]
				console.log(num1,num2,num3);
			}
			},
			onLoad(option) {
				this.getHotSongs()
				
			}
	}
</script>

<style lang="less" scoped>
.search{
width: 90%;
margin: 0 auto;
padding-top: 60rpx ;
  .search-head{
	display: flex;
	align-items: center;
	justify-content: space-between;
	input{
		width: 85%;
		// border: 1px solid #999;
		height: 60rpx;
		background-color: #f7f7f7;
		border-radius: 20rpx;
		padding-left: 20rpx ;
	}
	img{
		width: 60rpx;
		height: 60rpx;
	}
  }
  .search-show{
	margin-top: 40rpx;
	.song-list{
		display: flex;
		justify-content: space-between;
		border-bottom: 1px solid #ccc;
		padding-bottom: 20rpx;
		margin-bottom: 30rpx;
		.song-name{
			font-size: 26rpx;
			font-weight: bold;
			color: #648bbe;
		// 	margin-bottom: 10rpx;
		}
		.song-dec{
			display: flex;
			align-items: center;
			color: #666;
			image{
				width:30rpx;
				height: 20rpx;
				margin-right: 10rpx;
			}
		}
		image{
			width: 60rpx;
			height: 60rpx;
			margin-top: 4rpx;
		}
	}
  }
  // 猜你喜欢
  .search-main{
	 margin-top: 40rpx;
	 .song-list{
		 .song-item{
			 display: flex;
			 margin-top: 30rpx;
			 position: relative;
			 strong{
				 color: red;
				 margin-right: 30rpx;
			 }
			 .item-name{
				 font-weight: bold;
			 }
			 .item-num{
				 color: #666;
				 position: absolute;
				 right: 0;
			 }
		 }
	 }
  }
	   
}
</style>
