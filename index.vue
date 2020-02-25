<template>
	<view class="flow-box" :style="'height: ' + loadingTop + 'px'">
		<view
			class="item"
			:class="left[index] == 1 ? 'left' : ''"
			hover-class="item-hover"
			:style="'top:' + top[index] + 'px;opacity:' + ((index + 1) <= mark ? 1 : 0)"
			v-for="(item, index) in newList"
			:key="index"
			:data-type="item.type"
			:data-id="item.id"
			@click="choose(item)"
		>
			<view v-text="getType(item)" class="float_tag" v-if="item.type === 4">
			</view>
			<view class="pic" v-if="item.pic && item.pic.picUrl">
				<view class="pic_tags" v-if="item.labelList.length > 0">
					<view class="pic_tag" v-for="(label, index) in item.labelList" :key="index">{{ label.label }}</view>
				</view>
				<image class="image" mode="widthFix" :src="item.pic.picUrl" style="width: 100%; display: block;" @error="errorImg(item.pic)" @load="load"></image>
			</view>
			<view class="content-item">
				<view class="content-text" v-if="item.title">{{ item.title }}</view>
				<view class="user">
					<image class="user-image" :src="item.photo || '../../static/img/default.png'" @error="errorImgPhoto(item)"></image>
					<view class="user_name">{{ item.username }}</view>
					<image class="heart-icon" src="../../static/img/home/hearted.png" v-if="item.likeFlag === 0"></image>
					<image class="heart-icon" src="../../static/img/home/heartsm.png" v-else></image>
					<view class="heart-div">{{ item.likeNum || 0 }}</view>
				</view>
				<!-- <view class="tags" v-if="item.labelList > 0">
					<view class="biao-tags" v-for="(label, index) in item.labelList" :key="index">
						<image class="tags-icon" src="../../static/img/home/tags.png"></image>
						<view>{{ label.label }}</view>
					</view>
				</view> -->
				<!-- <view class="foot-div">
					<view class="tags foot-tags" v-if="item.type === 3 || item.type === 4"><view class="ri-tags" v-text="getType(item)"></view></view>
					<view class="eye-div">
						<image class="eye-icon" src="../../static/img/home/eye.png"></image>
						{{ item.watchNum || 0 }}
					</view>
					<view class="heart-div">
						<image class="heart-icon" src="../../static/img/home/hearted.png" v-if="item.likeFlag === 0"></image>
						<image class="heart-icon" src="../../static/img/home/heartsm.png" v-else></image>
						{{ item.likeNum || 0 }}
					</view>
				</view> -->
			</view>
		</view>
		<view class="loading" v-show="loading" :style="'top: ' + loadingTop + 'px'">
			<image src="../../static/img/nairenk-waterfall-flow/loading.gif" style="width: 80upx; height: 80upx;"></image>
		</view>
	</view>
</template>

<script>
export default {
	props: {
		// 数据列表
		list: {
			type: Array,
			default() {
				return [];
			}
		},
		// 加载动画
		loading: {
			type: Boolean,
			default: false
		},
		imgsNum: {
			type: Number,
			default: 0
		}
	},
	data() {
		return {
			mark: 0,
			newList: [],
			boxHeight: [],
			top: [],
			left: [],
			loadingTop: 0,
			imgLoadeds: []
		};
	},
	watch: {
		// 数据
		list: function(newVal, oldVal) {
			this.mark = oldVal.length;
			if (newVal != oldVal) {
				this.newList = this.list;
				// this.$nextTick(function () {
				// 	setTimeout(() => {
				// 		this.waterFall();
				// 	}, 120)
				// })
			}
		}
	},
	methods: {
		reset() {
			this.loadingTop = 0;
			this.boxHeight = [];
			this.top = [];
			this.left = [];
			this.imgLoadeds = [];
			this.mark = 0;
		},
		getType(item) {
			let result = '三维案例';
			switch (item.type) {
				case 1:
					result = '三维案例';
					break;
				case 2:
					result = '帖子';
					break;
				case 3:
					result = '科普文章';
					break;
				case 4:
					// result = '美丽日志-' + item.totalPiece + '篇';
					result = '美丽日志';
					break;
			}
			return result;
		},
		load(index) {
			this.imgLoadeds.push(index);
			if (this.imgLoadeds.length >= this.imgsNum) {
				this.$nextTick(function() {
					setTimeout(() => {
						this.waterFall();
					}, 0);
				});
			}
		},
		// 瀑布流定位
		waterFall() {
			const query = uni.createSelectorQuery().in(this);
			query
				.selectAll('.flow-box .item')
				.boundingClientRect(res => {
					let len = this.newList.length;
					let height = 0;
					let flag = 0

					for (let i = this.mark; i < len; i++) {
						height = res[i].height;
						if (i < 2) {
							this.$set(this.newList[i], 'top', 0);
							this.$set(this.newList[i], 'left', i);
							if(len === 1) {
								this.loadingTop = res[0].height
							}else if(len === 2) {
								this.loadingTop = res[0].height > res[1].height ? res[0].height : res[1].height
							}
							this.boxHeight.push(height);
							this.top.push(0);
							this.left.push(i);
						} else {
							let minHeight = this.boxHeight[0];
							let index = 0;
							if (minHeight > this.boxHeight[1]) {
								minHeight = this.boxHeight[1];
								index = 1;
								}
							this.boxHeight[index] = minHeight + height + 6;
							if(i === len - 1) {
								flag = this.boxHeight[index]
								if(this.boxHeight[index] < this.boxHeight[0]) {
									this.boxHeight[index] = this.boxHeight[0]
								}
								if(this.boxHeight[index] < this.boxHeight[1]) {
									this.boxHeight[index] = this.boxHeight[1]
								}
							}
							this.top.push(minHeight + 6);
							this.left.push(index);
							this.$set(this.newList[i], 'top', minHeight + 6);
							this.$set(this.newList[i], 'left', index);
							this.loadingTop = this.boxHeight[index];
							if(flag !== 0) {
								this.boxHeight[index] = flag
							}
						}
						if(i === len - 1) {
							this.mark = len
						}
					}
				})
				.exec();
		},
		// 选中
		choose(e) {
			// console.log(e);
			let type = e.type;
			let id = e.id;
			const info = {
				type,
				id
			};
			this.$emit('click', info);
		},
		errorImg(pic) {
			pic.picUrl = require('@/static/img/nairenk-waterfall-flow/error.jpg');
			// item.pic[0].picUrl = require("@/static/img/nairenk-waterfall-flow/error.jpg")
		},
		errorImgPhoto(item) {
			item.photo = require('@/static/img/default.png');
		}
	}
};
</script>

<style scoped>
.flow-box {
	position: relative;
	color: #1a1a1a;
}
.flow-box .item {
	position: absolute;
	left: 12upx;
	width: calc(50% - 24upx);
	border: 1upx solid #f9f9f9;
	background: rgba(255, 255, 255, 1);
	box-shadow: 0px 0px 6upx rgba(0, 0, 0, 0.16);
	border-radius: 8upx;
}

.item-hover {
	background-color: #eee;
}

.float_tag {
	position: absolute;
	background-color:rgba(168,143,247,1);
	box-shadow:0px 0px 4upx rgba(0,0,0,0.1);
	border-radius:8upx 0px 8upx 0px;
	font-size:16upx;
	font-weight:400;
	line-height:22upx;
	color:rgba(255,255,255,1);
	top: 0;
	left: 0;
	padding: 4upx 12upx;
	z-index: 10;
}

.flow-box .left {
	left: 380upx;
}
.flow-box .pic {
	position: relative;
	/* margin: 8upx 8upx 0; */
}

.pic .image {
	border-radius: 8upx;
	will-change: transform;
}
.pic_tags {
	position: absolute;
	bottom: 8upx;
	right: 8upx;
	display: flex;
	align-items: center;
	z-index: 10;
}
.pic_tags .pic_tag {
	background:rgba(51,51,51,0.5);
	border-radius:14upx;
	margin-right: 8upx;
	font-size:16upx;
	font-weight:300;
	line-height:22upx;
	color:rgba(255,255,255,1);
	padding: 2upx 8upx;
}
.flow-box .content-item {
	padding: 8upx 12upx 24upx 8upx;
	/* display: flex;
		justify-content: space-between;
		flex-wrap: wrap; */
}
.flow-box .content-item text {
	width: 100%;
	font-size: 24upx;
	margin-bottom: 20upx;
}
.flow-box .user {
	display: flex;
	overflow: hidden;
	align-items: center;
	margin-top: 12upx;
}

.user .user-image {
	width: 36upx;
	height: 36upx;
	border-radius: 50%;
	margin-right: 12upx;
}

.user .user_name {
	flex: 1;
	font-size:20upx;
	font-weight:400;
	line-height:28upx;
	color:rgba(102,102,102,1);
	word-break: break-all;
	text-overflow: ellipsis;
	overflow: hidden;
	white-space: nowrap;
}

.content-item .content-text {
	text-align: left;
	font-size: 24upx;
	font-weight: 400;
	line-height: 34upx;
	color: rgba(51,51,51,1);
	text-overflow: -o-ellipsis-lastline;
	overflow: hidden;
	text-overflow: ellipsis;
	display: -webkit-box;
	-webkit-line-clamp: 2;
	line-clamp: 2;
	-webkit-box-orient: vertical;
}

.foot-div {
	display: flex;
	justify-content: flex-end;
	align-items: center;
	margin-top: 8upx;
	width: 100%;
}

.eye-div {
	display: flex;
	align-items: center;
	font-size: 16upx;
	font-weight: 400;
	line-height: 22upx;
	color: rgba(117, 117, 117, 1);
	margin-right: 12upx;
}

.eye-div .eye-icon {
	width: 26upx;
	height: 16upx;
	margin-right: 2upx;
}

.heart-div {
	font-size:20upx;
	font-weight:400;
	line-height:28upx;
	color:rgba(102,102,102,1);
}

.heart-icon {
	width: 24upx;
	height: 24upx;
	margin-right: 8upx;
}

.loading {
	background-color: #FFFFFF;
	position: absolute;
	width: 100%;
	text-align: center;
	padding: 20upx 0;
}
</style>
