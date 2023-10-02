<template>
		<!-- 本示例未包含完整css，获取外链css请参考上文，在hello uni-app项目中查看 -->
	<view>
		<swiper class="swiper" circular :indicator-dots="indicatorDots" :autoplay="autoplay" :interval="interval"
			:duration="duration">
			<swiper-item v-for="(item,index) in bgImages" :key="index">
			<image :src="item.url" mode=""></image>
			</swiper-item>
		</swiper>
		<view>
			<input type="text" placeholder="搜索歌曲" class="uni-input" @click="goSearch()"/>
		</view>
		
		<view class="hot">
			<view class="item" v-for="(item,index) in listData" :key="index" @click="goDetail(item.id)">
				<!-- 左列表 -->
				<view class="item-left">
					<image :src="item.url" mode=""></image>
					<text>{{item.name}}</text>
				</view>
				<!-- 右展示 -->
				<view class="item-right">
					<text v-for="(i,num) in item.music_list" :key="num">{{num+1}}.{{i}}</text>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
		    bgImages:[
				{id:1,url:require("@/static/images/1.jpg")},
				{id:2,url:require("@/static/images/2.jpg")},
				{id:3,url:require("@/static/images/3.jpg")}
			],
            indicatorDots: true,
            autoplay: true,
            interval: 2000,
            duration: 500,
			listData:[
				{}
			]
        }
		},
		onLoad() {  //页面加载
           this.getData()
		},
		methods: {
           // 获取数据
		   getData(){
			   uni.request({
			       url: 'http://47.108.157.232:8889/getRankList', 
			       success: (res) => {
			           console.log(res.data);
					   this.listData = res.data
					   // 把数据存储到本地存储
					   uni.setStorage({
					   	key: 'topdata',
					   	data: res.data,
					   });  
			       }
			   });
		},
		// 详情页面
		goDetail(id){
			console.log(id,"-----");
			uni.navigateTo({
				url: '/pages/detail/detail?id='+id
			});
		},
		// 搜索页面
		goSearch(){
			uni.navigateTo({
				url: '/pages/search/search'
			});
		}
	}
}
</script>

<style lang="less">
.swiper {
	height: 300rpx;
	image{
		width: 100%;
	}
	}
	.uni-input{
		width: 80%;
		border: 1px solid #999;
		height: 45rpx;
		border-radius: 14rpx;
		margin: 40rpx auto;
		padding-left: 8rpx;
	}
	//榜单
	.hot{
		width: 94%;
		margin: 0 auto;
		.item{
			height: 240rpx;
			display: flex;
			margin-bottom: 20rpx;
			.item-left{
				width: 280rpx;
				height: 240rpx;
				margin-right: 20rpx;
				position: relative;
				image{
					margin-bottom: 20rpx;
					width: 100%;
					height: 100%;
				}
				text{
					color: white;
					font-weight: bold;
					position: absolute;
					left: 8rpx;
					bottom: 10rpx;
				}
			}
			.item-right{
				width: 500rpx;
				display: flex;
				flex-direction: column;
				justify-content: space-around;
			}
		}
	}
</style>
