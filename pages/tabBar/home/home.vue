<template>
	<view>
		<!-- 状态栏 -->
		<page-status></page-status>
		<!-- 自定义顶部导航栏 -->
		<pageHeader />

		<!-- 轮播图 -->
		<view class="swiper">
			<view class="swiper-box">
				<swiper circular="true" autoplay="true" @change="swiperChange">
					<swiper-item v-for="swiper in swiperList" :key="swiper.id">
						<image :src="swiper.img"></image>
					</swiper-item>
				</swiper>
				<view class="indicator">
					<view :class="{'on':currentSwiper >= index}" class="dots" v-for="(swiper,index) in swiperList" :key="index"></view>
				</view>
			</view>
		</view>
		<!-- 分类列表 -->
		<view class="category-list">
			<view @tap="handleCategory(item)" class="category" v-for="(item,index) in categoryList" :key="index">
				<view class="img">
					<image :src="item.img"></image>
				</view>
				<view class="text">{{item.name}}</view>
			</view>
		</view>

		<!-- 广告图banner -->
		<view v-if="promotion.length > 0" class="banner">
			<image src="/static/img/category/ad.jpg"></image>
		</view>

		<!-- 活动区 -->
		<view class="promotion" v-if="promotion.length > 0">
			<view class="text">优惠活动</view>
			<view class="list">
				<view @tap="handlePromotion(item)" class="column" v-for="(item,index) in promotion" :key="index">
					<view class="top">
						<view class="title">{{item.title}}</view>
					</view>
					<view class="left">
						<view class="ad">{{item.ad}}</view>
						<view class="into">点击进入</view>
					</view>
					<view class="right">
						<image :src="item.img"></image>
					</view>
				</view>
			</view>
		</view>
	<view>
		<view class="header">
			<view @tap="handleSelect(index)" class="target" v-for="(target,index) in filterByList" :key="index" :class="{'on':target.selected}">
				{{target.text}}
			</view>
		</view>
		<view class="promotion" v-if="promotion.length > 0">
			<view class="text">新书预览</view></view>
		<!-- 占位 tabbar 有定位 -->
		<view class="place"></view>
		<!-- 商品列表 -->
		<view class="goods-list">
			<view class="product-list">
				<view class="product" @tap="handleGoods(goods)" v-for="goods in goodsList" :key="goods.goods_id">
					<image mode="widthFix" :src="goods.img"></image>
					<view class="name">{{goods.name}}}</view>
					<view class="info">
						<view class="price">
							<text>￥</text>
							{{goods.price}}
						</view>
						<view class="slogan">{{goods.slogan}} <text>人付款</text></view>
					</view>
				</view>
			</view>
			<view class="loading-text">{{loadingText}}</view>
		</view>
	</view>
	</view>

</template>

<script>
	import pageHeader from './pageHeader.vue'
	import interfaces from '../../../utils/interfaces.js'
	export default {
		data() {
			return {
				swiperList: [], // 轮播图片
				categoryList: [], // 分类
				promotion: [], // 活动
				currentSwiper: 0,
				goodsList: [], // 商品列表大数组
				filterby: "all", // 默认选择的tab
				page: 1,
				size: 30,
				loadingText: "正在加载....",
				filterByList: [{
						text: "全部",
						selected: true,
						filterby: "all"
					},
					{
						text: "口碑",
						selected: false,
						filterby: "public"
					},
					{
						text: "热门",
						selected: false,
						filterby: "hot"
					}
				]
			}
		},
		components: {
			pageHeader
		},
		onLoad() {
			this.initData();
			uni.setNavigationBarTitle({
				title: option.name
			})
			
			// 加载数据
			this.loadData();
		},
		
		methods: {
			handlePromotion(item) {
				uni.showToast({
					title: item.title,
					icon: 'none'
				})
			},
			handleCategory(item) {
				// 分类跳转
				// console.log(item.name);
				uni.navigateTo({
					url:"../../goods/goodsList?name="+item.name
				})
			},
			initData() {
				this.request({
					url: interfaces.getMallData,
					success: ((res) => {
						// console.log(res);
						this.swiperList = res.data.swiperList;
						this.categoryList = res.data.categoryList;
						this.promotion = res.data.promotion;
					})
				});
			},
			swiperChange(event) {
				// console.log(event.detail.current);
				this.currentSwiper = event.detail.current;
			},
			handleGoods(goods){
				// 页面跳转 商品详情
				uni.navigateTo({
					url:'./goods?goodsInfo='+JSON.stringify(goods)
				})
			},
			handleSelect(index) {
				this.filterByList[index].selected = true;
			
				// 其他的selected false
				for (let i = 0; i < this.filterByList.length; i++) {
					if (i != index) {
						this.filterByList[i].selected = false;
					}
				}
				
				// 数据请求
				this.filterby = this.filterByList[index].filterby;
				this.loadingText = "加载中...";
				this.goodsList = [];
				this.loadData();
				this.loadData2();
			},
			loadData() {
				this.request({
					url: "https://mock.mengxuegu.com/mock/625e2bc566abf914b1f1c131/mxg-education-app/hoodslist/all/1/6#!method=get",
					success: ((res) => {
						if(res.data.length > 0){
							res.data.forEach(item => {
								this.goodsList.push(item);
							})
						}else{
							this.loadingText = "到底了";
						}
					})
				})
			},
			loadData2() {
				this.request({
					url: "https://mock.mengxuegu.com/mock/625e2bc566abf914b1f1c131/mxg-education-app/goodlists2#!method=get",
					success: ((res) => {
						if(res.data.length > 0){
							res.data.forEach(item => {
								this.goodsList.push(item);
							})
						}else{
							this.loadingText = "到底了";
						}
					})
				})
			},
			loadData3() {
				this.request({
					url: "https://mock.mengxuegu.com/mock/625e2bc566abf914b1f1c131/mxg-education-app/goodslist#!method=get",
					success: ((res) => {
						if(res.data.length > 0){
							res.data.forEach(item => {
								this.goodsList.push(item);
							})
						}else{
							this.loadingText = "到底了";
						}
					})
				})
			},
			onLoad() {
				this.initData();
				uni.setNavigationBarTitle({
					title: option.name
				})
				
				// 加载数据
				this.loadData();
			},
			onPullDownRefresh(){
				setTimeout(() => {

					this.loadingText = "加载中...";
					this.loadData3();
					this.loadData();
					this.loadData2();

					uni.stopPullDownRefresh();
				},500)
			}, 
			// 上啦加载
			onReachBottom(){
				setTimeout(() => {
				
					this.loadingText = "加载中...";
					this.loadData2();
					this.loadData3();
					this.loadData();
					uni.stopPullDownRefresh();
				},500)
			}
		}
	}
</script>

<style lang="scss">
	.swiper {
		width: 100%;
		margin-top: 10upx;
		display: flex;
		justify-content: center;

		.swiper-box {
			width: 92%;
			height: 30.7vw;
			overflow: hidden;
			border-radius: 15upx;
			box-shadow: 0upx 8upx 25upx rgba(0, 0, 0, 0.2);
			//兼容ios，微信小程序
			position: relative;
			z-index: 1;

			swiper {
				width: 100%;
				height: 30.7vw;

				swiper-item {
					image {
						width: 100%;
						height: 40vw;
					}
				}
			}

			.indicator {
				position: absolute;
				bottom: 20upx;
				left: 20upx;
				background-color: rgba(255, 255, 255, 0.4);
				width: 150upx;
				height: 5upx;
				border-radius: 3upx;
				overflow: hidden;
				display: flex;

				.dots {
					width: 0upx;
					background-color: rgba(255, 255, 255, 1);
					transition: all 0.3s ease-out;

					&.on {
						width: (100%/3);
					}
				}
			}
		}
	}

	.category-list {
		width: 92%;
		margin: 0 4%;
		padding: 0 0 30upx 0;
		border-bottom: solid 2upx #f6f6f6;
		display: flex;
		justify-content: space-between;
		flex-wrap: wrap;

		.category {
			width: 25%;
			margin-top: 50upx;
			display: flex;
			flex-wrap: wrap;

			.img {
				width: 100%;
				display: flex;
				justify-content: center;

				image {
					width: 11vw;
					height: 11vw;
				}
			}

			.text {
				margin-top: 16upx;
				width: 100%;
				display: flex;
				justify-content: center;
				font-size: 24upx;
				color: #3c3c3c;
			}
		}
	}

	.banner {
		width: 92%;
		margin: 40upx 4%;

		image {
			width: 100%;
			height: 36vw;
			border-radius: 4upx;
			box-shadow: 0upx 5upx 25upx rgba(0, 0, 0, 0.3);
		}
	}

	.promotion {
		width: 92%;
		margin: 10upx 4% 30upx 4%;

		.text {
			width: 100%;
			height: 60upx;
			font-size: 34upx;
			font-weight: 600;
			margin: 10upx;
		}

		.list {
			width: 100%;
			display: flex;
			justify-content: space-between;

			.column {
				width: 43%;
				padding: 15upx 3%;
				background-color: #ebf9f9;
				border-radius: 10upx;
				overflow: hidden;
				display: flex;
				justify-content: space-between;
				flex-wrap: wrap;

				.top {
					width: 100%;
					height: 40upx;
					display: flex;
					align-items: center;
					margin-bottom: 5upx;

					.title {
						font-size: 30upx;
					}
				}

				.left {
					width: 50%;
					height: 18.86vw;
					display: flex;
					flex-wrap: wrap;
					align-content: space-between;

					.ad {
						margin-top: 5upx;
						width: 100%;
						font-size: 22upx;
						color: #acb0b0;
					}

					.into {
						width: 100%;
						font-size: 24upx;
						color: #aaa;
						margin-bottom: 5upx;
					}
				}

				.right {
					width: 18.86vw;
					height: 18.86vw;

					image {
						width: 18.86vw;
						height: 18.86vw;
					}
				}
			}
		}
	}
	.header {
		width: 92%;
		padding: 0 4%;
		height: 79upx;
		display: flex;
		justify-content: space-around;
		align-items: flex-end;
		position: fixed;
		z-index: 10;
		background-color: #fff;
		border-bottom: solid 1upx #eee;
	
		.target {
			width: 20%;
			height: 60upx;
			display: flex;
			justify-content: center;
			align-items: center;
			font-size: 28upx;
			margin-bottom: -2upx;
			color: #aaa;
	
			&.on {
				color: #555;
				border-bottom: 4upx solid #f06c7a;
				font-weight: 600;
				font-size: 30upx;
			}
	
			.icon {
				font-size: 26upx;
			}
		}
	}
	
	.place {
		background-color: #ffffff;
		height: 100upx;
	}
	
	.goods-list {
		.loading-text {
			width: 100%;
			display: flex;
			justify-content: center;
			align-items: center;
			height: 60upx;
			color: #979797;
			font-size: 24upx;
		}
	
		.product-list {
			width: 92%;
			padding: 0 4% 3vw 4%;
			column-count: 2;
			column-gap: 1em;
	
			.product {
				break-inside: avoid;
				border-radius: 20upx;
				background-color: #fff;
				margin: 0 0 15upx 0;
				box-shadow: 0upx 5upx 25upx rgba(0, 0, 0, 0.1);
	
				image {
					width: 100%;
					border-radius: 20upx 20upx 0 0;
				}
	
				.name {
					width: 92%;
					padding: 6upx 4%;
					display: -webkit-box;
					-webkit-box-orient: vertical;
					-webkit-line-clamp: 2;
					text-align: justify;
					overflow: hidden;
					font-size: 30upx;
				}
	
				.info {
					display: flex;
					justify-content: space-between;
					align-items: flex-end;
					width: 92%;
					padding: 10upx 4% 10upx 4%;
	
					.price {
						color: #e65339;
						font-size: 30upx;
						font-weight: 600;
					}
	
					.slogan {
						color: #807c87;
						font-size: 24upx;
					}
				}
			}
	
		}
	}
</style>
