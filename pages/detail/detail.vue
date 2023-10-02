<template>
  <view class="detail">
	  <!-- 上面布局 -->
	<view class="detail-head" :style="{background:bgColor}">
		   <!-- 左边 -->
			<view class="detail-head-left">
				<image :src="detailData.url" alt=""/>
				<h4>播放量: 48.6亿</h4>	
			</view>
			<!-- 右边 -->
            <view class="detail-head-right">
				<h3 class="title">{{detailData.name}}</h3>
				<view class="logo">
					<image src="@/static/images/musicLogo.jpg" mode=""></image>
					<text>网易云音乐</text>
				</view>
				<view class="tip">
					{{detailData.describes}}
				</view>
			</view>
		</view>
		<!-- 下面部分 -->
		<view class="content">
			<!-- 上面按钮 -->
			<view class="content-title">
				<image src="@/static/images/play1.png" mode=""></image>
				<strong>播放全部</strong>
				<text>(共100首)</text>
			</view>
			<!-- 下面列表 -->
			<view class="detail-list">
			<view class="item" v-for="(item,index) in detailData.music" :key="index" @click="goPlaymusic(item.id)">
				<strong>{{index + 1}}</strong>
				<view class="info">
					<view class="song-name">
						{{item.music_name}}
					</view>
					<view class="info-bottom">
						<image src="@/static/images/11.png" mode=""></image>
						<text>{{item.music_author}}</text>
					</view>
				</view>
				<image class="play" src="@/static/images/play2.png" mode=""></image>
			</view>
		</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				id: 0,
				detailData: {},
				detailHead: {}, //头部数据渲染
				bgColor:'',
				songArr:[]
			}
		},
		methods: {
			// 从本地存储拿存储的数据
			getDataStorage(){
				let _this = this;
				uni.getStorage({
					key: 'topdata',
					success: function (res) {
						// console.log(res.data,"444");
						res.data.forEach(function(v){
							// console.log(v);
							if(_this.id == v.id){
								console.log(v,"000");
								_this.detailHead = v;
								let name = v.name;
								console.log(v.name);
								switch(name){
									case '飙升榜':
									_this.bgColor = "#df78a5"
									break;
									case '新歌榜':
										_this.bgColor = "#66c5bd"
									break;
									case '原创榜':
										_this.bgColor = "#6aadde"
									break;
									case '热歌榜':
										_this.bgColor = "#e46e62"
									break;
									
								}
							}
						})
					}
				});
			},
			// 详情页面
			goPlaymusic(id){
				console.log(id,"-----");
				uni.navigateTo({
					url: '/pages/playmusic/playmusic?id='+id
				});
			},
		   getDetailData(){
		   let _this = this
			uni.request({
				url: 'http://47.108.157.232:8889/getRankInfo?id='+_this.id, 
				success: (res) => {
			    console.log(res.data,"999");
				_this.detailData = res.data;
				let dataArr = res.data.music
				dataArr.forEach(function(v){
					// console.log(v,"888");
					_this.songArr.push({
						id:v.id,
						name:v.music_name
					})
					// console.log(_this.songArr,"包含id和歌名");
					  // 把数据存储到本地存储
					uni.setStorage({
						key: 'songsArr',
						data: _this.songArr,
				});
			  })
		    }
		  });
		},
		   onLoad(option){
			console.log(option,"1111");
			  this.id = option.id
			  this.getDataStorage()
			  this.getDetailData()
		}
	},
}
</script>

<style lang="less">
.detail{
	width: 100%;
   .detail-head{
      height: 320rpx;
	  padding: 50rpx 40rpx;
	  display: flex;
	  color: #fff;
      // background-color: #bc648e;
	  .detail-head-left{
		width: 300rpx;
		height: 300rpx;
		margin-right: 30rpx;
		image{
		  margin-bottom: 20rpx;
		  width: 100%;
		  height: 95%;
		  border-radius: 20rpx;
		}
	  }
	  .detail-head-right{
		width: 320rpx;
		height: 300rpx;
		padding-top: 30rpx;
		image{
		  width: 50rpx;
		  height: 50rpx;
		  border-radius: 50%;
		  margin-right: 26rpx;
		}
		.logo{
		  margin: 25rpx 0;
		  display: flex;
		  align-items: center;
		}
	  }
	}
	// 下面部分
	.content{
	   width: 84%;
	   margin: 40rpx auto;
       // background-color: #fcccee;
	   .content-title{
		   display: flex;
		   align-items: center;
		   image{
			   width: 60rpx;
			   height: 60rpx;
		   }
		   strong{
			   margin: 0 20rpx 0 20rpx;
		   }
	   }
	   .detail-list{
		   margin-top: 40rpx;
		   .item{
		     display: flex;
			 position: relative;
			 margin-bottom: 20rpx;
			 strong{
				 width: 20rpx;
				 margin-right: 60rpx;
			 }
			 .info-bottom{
				 image{
					 width: 40rpx;
					 height: 20rpx;
					 margin-right: 16rpx;
				 }  
			 }
			 .play{
				 width: 60rpx;
				 height: 60rpx;
				 position: absolute;
				 right: 0;
			 }
		   }
	   }
	}
}
</style>
