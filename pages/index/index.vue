<template>
	<view>
		<swiper :circular="true" class="swipers" @change="change" :current="current" :vertical="true"
			:indicator-dots="false" :autoplay="false" :interval="3000" :duration="300">
			<swiper-item class="swipers-item" v-for="(item,index) in swiperList" :key="item.courseDetailsId">
				<view class="swipers-items">
					<!-- 视频 -->
					<video @timeupdate="timeupdate" @waiting="waiting()" object-fit="cover"
						v-if="current === index && item.videoUrl" :play-strategy="2" :show-loading="true"
						codec="software" :muted="false" :show-center-play-btn="true" :loop="true" @ended="ended"
						@play="videoPlay('myVideo'+item.courseDetailsId)" :enable-progress-gesture="false"
						:poster="item.titleImg" :ref="'myVideo'+item.courseDetailsId"
						:id="'myVideo'+item.courseDetailsId" :src="item.videoUrl" :autoplay="item.autoPlay"
						class="swipers-items-video"></video>
					<!-- :autoplay="item.autoPlay" -->
					<!-- 没有视频权限则显示封面图 -->
					<image v-else @click="showPay = true" :src="item.titleImg" class="swipers-items-imgsbg"
						mode="aspectFill"></image>
					<!-- 返回图标 -->
					<!-- <u-icon name="arrow-left" @click="goBack()" class="swipers-items-back" :style="{top:meunTop+'px'}"
						color="#ffffff" size="38"></u-icon> -->
					<!-- 视频信息 -->
					<view class="swipers-items-info">
						<!-- 标题 -->
						<view class="swipers-items-info-title">
							{{item.courseDetailsName}}
						</view>
						<!-- 简介 -->
						<view class="swipers-items-info-content" v-html="item.content">

						</view>
						<!-- 集数 -->
						<view @click="openShow()" class="swipers-items-info-num">
							第{{item.num}}集（共{{meunList.length}}集）选集 >
						</view>
					</view>
					<!-- 右侧操作 -->
					<view class="swipers-items-right">
						<view class="swipers-items-right-item">
							<view class="swipers-items-right-item-img" @click.stop="dianzan(item)">
								<u-icon name="heart-fill" v-if="item.isGood!=0" color="red" size="60"></u-icon>
								<u-icon name="heart-fill" v-else color="#ffffff" size="60"></u-icon>

							</view>
							<view class="swipers-items-right-item-txt">
								{{item.goodNum}}
							</view>
						</view>
						<view class="swipers-items-right-item" @click="share(item)">
							<view class="swipers-items-right-item-img">

								
							</view>
							<view class="swipers-items-right-item-txt">
								分享
							</view>
						</view>
						<view class="swipers-items-right-item" v-if="item.isCollect == 0">
							<view class="swipers-items-right-item-img" @click.stop="shoucang(item)">
							</view>
							<view class="swipers-items-right-item-txt">
								追剧
							</view>
						</view>
						<view class="swipers-items-right-item" v-else>
							<view class="swipers-items-right-item-img" @click.stop="shoucang(item)">
								</image>
							</view>
							<view class="swipers-items-right-item-txt">
								已追
							</view>
						</view>
					</view>
				</view>
			</swiper-item>
		</swiper>
		<u-popup :closeable="true" close-icon="arrow-down" close-icon-size="40" close-icon-color="#ffffff"
			:mask-custom-style="maskCustomStyle" v-model="show" height="800rpx" border-radius="24" mode="bottom">
			<view class="list">
				<view class="list-title flex align-center">
					<u-icon name="list" class="list-title-icon" color="#ffffff" size="40"></u-icon>
					当前播放第{{num}}个视频
				</view>
				<view class="list-box">
					<scroll-view :scroll-into-view="scrollIntoViews" :show-scrollbar="true" scroll-y="true"
						style="width: 100%;height: 700rpx;padding-bottom: 30rpx;">
						<view :id="item.viewInfo" class="list-box-item flex justify-between" @click="selectPlay(item)"
							v-for="(item,index) in meunList" :key="index">
							<view class="list-box-item-l">
								<image :src="item.titleImg" mode="aspectFill"></image>
							</view>
							<view class="list-box-item-r">
								<view class="list-box-item-r-title flex align-center justify-between">
									第{{index+1}}集
									<view class="list-box-item-r-title-t" v-if="('video'+index) == scrollIntoView">
										正在播放
									</view>
								</view>
								<view class="list-box-item-r-titles">
									《{{item.courseDetailsName}}》
								</view>
								<view class="list-box-item-r-content" v-html="item.content">

								</view>
							</view>
						</view>
					</scroll-view>
				</view>
			</view>
		</u-popup>
		<!-- 购买视频 -->
		<u-popup :closeable="true" :custom-style="customStyle" :safe-area-inset-bottom="false" close-icon="arrow-down"
			close-icon-size="40" close-icon-color="#ffffff" :mask-custom-style="maskCustomStyle" v-model="showPay"
			border-radius="24" mode="bottom">
			<view class="list">
				<view class="list-title flex align-center">
					<u-icon name="lock" class="list-title-icon" color="#efbb6b" size="40"></u-icon>
					当前视频 没有播放权限
				</view>
				<view class="list-item">
					<view class="list-item-box" @click="payVideo(1)">
						购买整部视频
					</view>

				</view>
				<view class="list-item">
					<view class="list-item-box" @click="payVideo(2)">
						购买单集视频
					</view>
				</view>
				<view class="list-item" v-if="isVips == '是'">
					<view class="list-item-box" @click="goNav('/pages/me/vip/index')">
						开通会员
					</view>
				</view>
			</view>
		</u-popup>
		<!-- #ifndef MP-WEIXIN -->
		<tki-qrcode ref="qrcode" v-if="isStart" :val="erweima" :size="200" background="#fff" foreground="#000"
			pdground="#000" :onval="true" :loadMake="true" @result="qrR" :show="false"></tki-qrcode>
		<!-- @success：成功事件  imgSrc：图片地址 QrSrc：二维码图片地址  
		Title：标题 PriceTxt：价格 OriginalTxt：原始价格 LineType -->
		<cc-poster v-if="haibaoShow==true && erweima" @error="posterError" @success="posterSuccess" :imgSrc="imgSrc"
			:QrSrc="qrCodeData" :Title="title" :PriceTxt="title" :LineType="false"></cc-poster>
		<!-- #endif -->

		<!-- #ifdef MP-WEIXIN -->
		<cc-poster v-if="haibaoShow==true && qrCodeData" @error="posterError" @success="posterSuccess" :imgSrc="imgSrc"
			:QrSrc="qrCodeData" :Title="title" :PriceTxt="title" :LineType="false"></cc-poster>
		<!-- #endif -->
		<u-popup width="686" v-model="showImg" mode="center" border-radius="14">
			<image :show-menu-by-longpress='true' :src="haibaoImg" style="width: 100%;" mode="widthFix"></image>
		</u-popup>
	</view>
</template>

<script>
	import tkiQrcode from '../../components/tki-qrcode/tki-qrcode.vue';
	export default {
		components: {
			tkiQrcode
		},
		data() {
			return {
				customStyle: {
					background: '#202020'
				},
				isStart: false, //是否开始生成二维码
				erweima: '',
				qrCodeData: '',
				haibaoImg: '',
				showImg: false,
				haibaoShow: false,
				title: '',
				imgSrc: '',
				current: 0,
				maskCustomStyle: {
					background: 'rgba(0, 0, 0, 0.5)'
				},
				show: false,
				videoList: [],
				videoContext: null, //记录当前视频的上下文
				isVIP: false, //是否是vip
				courseId: '',
				meunList: [], //菜单
				meunTop: 0, //返回图标距离顶部的距离
				num: 1, //当前播放的集数
				showPay: false, //购买视频的弹窗
				info: {}, //整部的信息
				courseDetailsId: '', //详情id
				scrollIntoView: '', //当前播放视频的位置
				scrollIntoViews: 'video0', //当前播放视频的位置
				isVips: '否', //是否显示会员
			};
		},
		computed: {
			swiperList() {
				return this.videoList
			}
		},
		onLoad(e) {
			// #ifdef MP-WEIXIN
			if (e.scene) { //这里为线上操作
				//此处的二维码  page/index/index?brokerId=123
				let scene = decodeURIComponent(e.scene);
				scene = scene.split(',')
				uni.setStorageSync('invitation', scene[0])
				this.courseId = scene[1]
				this.courseDetailsId = scene[2]
				this.getDataList(this.courseId, this.courseDetailsId);
			}
			// #endif
			if (e.invitation) {
				uni.setStorageSync('invitation', e.invitation)
			}
			if (e.id) {
				this.courseId = e.id
				if (e.courseDetailsId) {
					this.courseDetailsId = e.courseDetailsId
				}
				console.log(this.courseDetailsId, '跳转进来的id')
				this.getDataList(this.courseId, this.courseDetailsId);
			}
			this.getDataList()
		},
		onShow() {
			this.isVips = uni.getStorageSync('isVips') ? uni.getStorageSync('isVips') : '否'
			let that = this
			uni.$on('back', (data) => {
				if (data.flag == true) {
					that.showPay = false
					that.getDataList(that.courseId, that.courseDetailsId, true);
				}
			})
		},
		onReady() {
			// #ifdef MP-WEIXIN
			let info = uni.getMenuButtonBoundingClientRect()
			this.meunTop = info.top + ((info.height - 19) / 2)
			// #endif
			// #ifndef MP-WEIXIN
			this.meunTop = 37
			// #endif
		},
		watch: {
			//监听当前播放的集数
			current() {
				this.comNumVideo()
			}
		},
		methods: {
			//播放进度变化回掉
			timeupdate(e) {
				//隐藏loding
				uni.hideLoading()
			},
			//缓冲中
			waiting(e) {
				//在h5状态下视频出现缓冲则显示loding
				// #ifdef H5
				uni.showLoading()
				// #endif
			},
			openShow() {
				this.show = true
				this.$nextTick(() => {
					this.scrollIntoViews = this.scrollIntoView
				})

			},
			/**
			 * @param {String} path //返回的二维码地址
			 */
			qrR(path) {
				this.qrCodeData = path
				this.haibaoShow = true

			},
			//生成失败
			posterError() {
				this.haibaoImg = ''
				this.showImg = false
				// 生成完成后初始化分享
				this.haibaoShow = false
				this.isStart = false
				this.imgSrc = ''
				uni.showToast({
					title: '海报生成失败',
					mask: false,
					duration: 2000,
					icon: "none"
				});
			},
			/**生成成功
			 * @param {String} img 成功回调的图片
			 */
			posterSuccess(img) {
				this.haibaoImg = img.tempFilePath
				this.showImg = true
				// 生成完成后初始化分享
				this.haibaoShow = false
				this.isStart = false
				this.imgSrc = ''
				uni.hideLoading()
				uni.showToast({
					title: '长按图片保存海报',
					mask: false,
					duration: 2000,
					icon: "none"
				});
			},
			//分享
			share(item) {
				this.imgSrc = item.titleImg
				// #ifndef MP-WEIXIN
				let urls = config.APIHOST2 + '/me/detail/detail?id=' + item.courseId + '&courseDetailsId=' + item
					.courseDetailsId + '&invitation=' + uni.getStorageSync('invitationCode')
				this.erweima = urls
				this.isStart = true
				// #endif


				// #ifdef MP-WEIXIN
				let that = this
				let data = {
					invitationCode: uni.getStorageSync('invitationCode') + ',' + item.courseId + ',' + item
						.courseDetailsId,
					page: 'me/detail/detail'
				}
				uni.downloadFile({
					url: config.APIHOST + '/app/invite/mpCreateQr?invitationCode=' + data.invitationCode +
						'&page=' + data.page,
					success(res) {
						if (res.statusCode === 200) {
							that.qrCodeData = res.tempFilePath
							that.haibaoShow = true
						} else {
							uni.hideLoading()
							uni.showToast({
								title: '分享失败',
								icon: 'none'
							})
						}
					}
				})
				// #endif
			},
			/**
			 * @param {Object} type 类型 
			 * 1:购买整部视频
			 * 2:购买单集视频
			 */
			payVideo(type) {
				//处理数据跳转支付
			},
			//去开通会员
			goNav(url) {
				uni.navigateTo({
					url: url
				})
			},
			//收藏
			shoucang(item) {
				//此处调用收藏接口
			},
			//点赞
			dianzan(item) {
				//此处调用点赞接口
			},
			//计算正在观看哪个视频
			comNumVideo() {
				let courseDetailsId = this.videoList[this.current].courseDetailsId
				this.meunList.map((item, index) => {
					if (item.courseDetailsId == courseDetailsId) { //找到了
						this.num = item.num
					}
				})
			},
			//选择集数后在更新剩下的数据
			setVideoList(index) {
				//判断后续是否还有两条视频
				let lengths = this.meunList.length - (index + 1)
				//有最少两条
				if (lengths >= 2) {
					//直接拿两条赋值进去
					this.videoList = [...this.videoList, ...this.meunList.slice(index + 1, index + 3)]
				}
				//只剩一条数据
				if (lengths == 1) {
					//把剩下的一条给放进去
					this.videoList = [
						//选中的那条
						...this.videoList,
						//身下的一条
						this.meunList[this.meunList.length - 1],
						//最后一条则放总数组的第一条
						this.meunList[0]
					]
				}
				//选择的就是最后一条数据
				if (lengths == 0) {
					//后两条拿总数组的前两条就可以了
					this.videoList = [...this.videoList, ...this.meunList.slice(0, 2)]
				}
				//更新视图
				this.$forceUpdate()
			},
			//选择播放
			selectPlay(item) {
				// 根据选择的courseDetailsId拿到meunList列表中的相同数据的下标
				const index = this.meunList.findIndex(menu => menu.courseDetailsId === item.courseDetailsId);
				this.videoList = [this.meunList[index]]
				this.current = 0
				//重新计算一下当前观看的是哪个视频
				this.comNumVideo()
				// 把选择的视频的自动播放设置为true
				this.videoList[this.current].autoPlay = true
				//让当前选择的视频播放
				this.startPlay(this.current)
				//更新视图
				this.$forceUpdate()
				//关闭选择弹窗
				this.show = false
				//记录当前播放位置
				this.scrollIntoView = 'video' + index
				//插入历史记录
				this.setHistor(this.videoList[this.current].courseId, this.videoList[this.current]
					.courseDetailsId)
				//等视图跟新完了，把之前剩下的数据补进去在$nextTick中处理
				this.$nextTick(() => {
					this.setVideoList(index)
				})
			},
			// 资源详情（//此处获取视频的资源数据，处理数据）
			getDataList(id, courseDetailsId, type) {
				//以下为测试数据，实际请用接口获取
				let videoList = [
					{
						courseId:1,//视频id
						courseDetailsId:1,//当前视频id
						courseDetailsName:'回到古代当太zi4',//视频标题
						content:'回到古代当太zi4',//视频简介
						isGood:0,//是否点赞 1:点赞 0:未点赞
						isCollect:0,//是否追剧 1:已追 0:未追
						goodNum:1,//点赞数量
						videoUrl:'https://sdp003.wanshibao.com:49/m3u8/5cfewiWOsb9LUzdXoF8CY_nJ_nYB-TxjUPg8FEnaGwXdEHWmLOzdcwfxYHH9cMAtsiYJ_eP4pBudI9Q.m3u8',//视频播放地址
						titleImg:'https://sdp003.wanshibao.com:49/m3u8/pic/1a77B6xVZnZRDq-p5qMzR6lGtqvEAqQArAihNz9Ixp-w5lB2qhr1HE6UbwTozBi_8LOhsoevKxw.jpg',//视频封面图片
					},
					{
						courseId:1,//视频id
						courseDetailsId:2,//当前视频id
						courseDetailsName:'回到古代当太zi5',//视频标题
						content:'回到古代当太zi5',//视频简介
						isGood:0,//是否点赞 1:点赞 0:未点赞
						isCollect:0,//是否追剧 1:已追 0:未追
						goodNum:1,//点赞数量
						videoUrl:'https://sdp003.wanshibao.com:49/m3u8/e3132AbBDNItEc6T23Zo_RveoW5xpWSLYANwcPoJuJ3eFKMwUn9srZ5iW6g_Xw1_c3JcOvIHjD4S2rI.m3u8',//视频播放地址
						titleImg:'https://sdp003.wanshibao.com:49/m3u8/pic/bed1wbZXz-Gjf4qwoKwmJ0y5L8UkWz65whyMJfJaW0XPwapPPr-z-teVOs4cwjJ2rbqlxo2mUWE.jpg',//视频封面图片
					},
					{
						courseId:1,//视频id
						courseDetailsId:3,//当前视频id
						courseDetailsName:'回到古代当太zi6',//视频标题
						content:'回到古代当太zi6',//视频简介
						isGood:0,//是否点赞 1:点赞 0:未点赞
						isCollect:0,//是否追剧 1:已追 0:未追
						goodNum:1,//点赞数量
						videoUrl:'https://sdp003.wanshibao.com:49/m3u8/82afKKYJ2yza-iPefiofrJ0A6hcJ8UnJcWLvv9RVGoW3FcNqvG3JmrQZy5l00w6VYcIroikyCxD3lII.m3u8',//视频播放地址
						titleImg:'https://sdp003.wanshibao.com:49/m3u8/pic/15a5xE-c8eR_k76QA86VgjeHYUCZF_18TvT7QJzLH5HnHvWNlHdmBzIOdWxUj54uY9prCZdXU1o.jpg',//视频封面图片
					},
					{
						courseId:1,//视频id
						courseDetailsId:4,//当前视频id
						courseDetailsName:'回到古代当太zi7',//视频标题
						content:'回到古代当太zi7',//视频简介
						isGood:0,//是否点赞 1:点赞 0:未点赞
						isCollect:0,//是否追剧 1:已追 0:未追
						goodNum:1,//点赞数量
						videoUrl:'https://sdp003.wanshibao.com:49/m3u8/4333NUZUz_Asi9Bq19Fszc_yUKUWSG2Fj04N7-PDWng3xsm3fk4cCg7dqOTY4D4VBoiP3SC6_ZpJiIo.m3u8',//视频播放地址
						titleImg:'https://sdp003.wanshibao.com:49/m3u8/pic/ec4epSaxrB3IxB3oe6k4F2THW4nbSyKMZHmACkwvMlNmQQYwp7awyhJt-bB8Xey-2uJl8eDz9Sg.jpg',//视频封面图片
					}
				]
				videoList.map((item,index)=>{
					item.num = index + 1
					if (item.videoUrl) {
						item.autoPlay = false
					} else {
						item.autoPlay = false
					}
					item.viewInfo = 'video' + index
				})
				//菜单数组
				this.meunList = videoList
				if (type == true) { //购买视频后返回的
					let courseDetailsId = this.videoList[this.current].courseDetailsId
					this.meunList.some((item) => {
						if (item.courseDetailsId == courseDetailsId) {
							this.videoList[this.current].videoUrl = item.videoUrl
							//自动播放
							this.startPlay(this.current)
						}
						return true
					})
				} else { //直接跳转进来的
					let indexs = -1
					if (courseDetailsId) { //从记录进来的
						this.meunList.map((item, index) => {
							if (item.courseDetailsId == courseDetailsId) {
								indexs = index
							}
						})
						if (indexs != -1) { //找到了
							if (Number(indexs + 1) === this.meunList.length) { //最后一条
								if (this.meunList.length == 1) { //只有一条
									this.videoList = this.meunList.slice(0, 3)
								} else if (this.meunList.length == 2) {
									this.videoList = this.meunList.slice(0, 3)
								} else {
									this.videoList = [
										this.meunList[this.meunList.length - 1],
										this.meunList[0],
										this.meunList[1],
									]
								}
				
							} else if (Number(indexs) === Number(this.meunList.length - 1)) { //倒数第二条
								if (this.meunList.length == 1) { //只有一条
									this.videoList = this.meunList.slice(0, 3)
								} else if (this.meunList.length == 2) { //有两条的时候
									this.videoList = [
										this.meunList[1],
										this.meunList[0],
									]
								} else {
									this.videoList = [
										this.meunList[this.meunList.length - 2],
										this.meunList[this.meunList.length - 1],
										this.meunList[0],
									]
								}
				
							} else {
								//如果不是最后一条，也不是倒数第二条，则从找到的位置开始往后拿三条数据放入数组
								this.videoList = this.meunList.slice(indexs, indexs + 3)
							}
						} else {
							//没找到直接拿前三条数据即可
							this.videoList = this.meunList.slice(0, 3)
						}
					} else { //新的视频
						// //视频数组//直接拿前三条
						this.videoList = this.meunList.slice(0, 3)
						indexs = 0
					}
					console.log(this.videoList)
					//重新计算当前在播放哪个视频
					this.comNumVideo()
					if (indexs != -1 && this.meunList.length > 0) {
						this.$nextTick(() => {
							this.scrollIntoView = 'video' + indexs
						})
				
					}
				}
				let numIdCurr = this.videoList[0].courseDetailsId
				this.videoContextId = 'myVideo' + numIdCurr
				this.$nextTick(() => {
					this.videoContext = uni.createVideoContext(this.videoContextId, this)
					this.videoContext.play()
				})
				this.$forceUpdate()
				//插入历史记录
				this.setHistor(this.videoList[this.current].courseId, this.videoList[this.current]
					.courseDetailsId)
			},
			//记录历史记录
			setHistor(courseId, courseDetailsId) {
				// 这边调用接口记录观看历史
			},
			//swiper上下切换事件
			change(e) {
				//拿出当前的swiper索引
				let current = Number(e.detail.current)
				// 根据courseDetailsId在meunList中找到这个数据的下标
				let indexs = -1;
				//考虑向上滑动的时候
				// 有三条的时候
				if (current == 2 && this.current == 0) {
					//向上滑到头的时候先拿第一条数据的courseDetailsId
					let courseDetailsId = this.videoList[0].courseDetailsId
					indexs = this.meunList.findIndex(item => item.courseDetailsId === courseDetailsId);
					if (indexs != -1) { //找到了，indexs就是对应下标
						//先判断meunList数组剩下的元素是否够三条
						const lengths = (this.meunList.slice(0, indexs)).length
						if (lengths >= 3) { //够三条
							// 因为轮播图开启了首尾衔接,所以在滑动到尾部重新进入第一页之前替换全部数据为之前的三条
							//因为slice(str,end)，包含str，不包含end，所以拿三条数据则-3
							this.videoList = this.meunList.slice(indexs - 3, indexs)
						} else { //不够三条
							let arr = new Array(3)
							switch (lengths) {
								//只有一条
								case 1:
									//只有一条的时候需要给数组补两条凑够三条，把meunList数组的前两条拿出来补在后面
									arr[2] = this.meunList[indexs - 1] //把剩下的一条取出来放到要滑动的那一页
									//因为剩下的数据不够了，则拿meunList末尾的两条数据补齐
									//第二个用最后一条数据
									arr[1] = this.meunList[this.meunList.length - 1]
									//第一个用倒数第二条数据
									arr[0] = this.meunList[this.meunList.length - 2]
									//赋值
									this.videoList = arr
									break;
									//只有二条
								case 2:
									//还剩下两条数据的时候
									//把剩下的倒数第一条取出来放到要滑动的那一页
									arr[2] = this.meunList[indexs - 1]
									//把剩下的最后一条放到第二个元素
									arr[1] = this.meunList[indexs - 2]
									//最后一个元素则由meunList的最后一条补齐
									arr[0] = this.meunList[this.meunList.length - 1]
									//最后赋值
									this.videoList = arr
									break;
								default:
									//一条都没有的时候 [1,2,3,4,5,6]
									//直接把meunList末尾的三条放入数组即可
									this.videoList = this.meunList.slice(-3)
									break;
							}
						}
					}
				}
				//考虑向下滑动
				if (current == 0 && this.current == 2) { //是否滑动到第一条，虽然刚进入页面current为0，但是首次不触发
					//拿出当前的courseDetailsId
					let courseDetailsId = this.videoList[2].courseDetailsId
					indexs = this.meunList.findIndex(item => item.courseDetailsId === courseDetailsId);
					//记录当前数据在meunList中的下标
					if (indexs != -1) { //找到了，indexs就是对应下标
						//先判断meunList数组剩下的元素是否够三条
						const lengths = (this.meunList.slice(indexs + 1, this.meunList.length)).length
						if (lengths >= 3) { //够三条
							// 因为轮播图开启了首尾衔接,所以在滑动到尾部重新进入第一页之前替换全部数据为之后的三条
							this.videoList = this.meunList.slice(indexs + 1, indexs + 4)
						} else { //不够三条
							let arr = new Array(3)
							switch (lengths) {
								//只有一条
								case 1:
									//只有一条的时候需要给数组补两条凑够三条，把meunList数组的前两条拿出来补在后面
									arr[0] = this.meunList[indexs + 1]
									arr[1] = this.meunList[0]
									arr[2] = this.meunList[1]
									this.videoList = arr
									break;
								case 2:
									//只有二条
									arr[0] = this.meunList[indexs + 1]
									arr[1] = this.meunList[indexs + 2]
									arr[2] = this.meunList[0]
									this.videoList = arr
									break;
								default:
									//刚好没有数据了 直接拿meunList的前三条数据
									this.videoList = this.meunList.slice(0, 3)
									break;
							}
						}
					}
				}
				//获取当前播放的视频在菜单数组中的位置
				let scrollIndex = this.meunList.findIndex(ite => ite.courseDetailsId == this.videoList[current]
					.courseDetailsId)
				//记录当前播放位置
				this.scrollIntoView = 'video' + scrollIndex
				//控制播放
				this.startPlay(current)
				//设置轮播图索引
				this.current = current
				//插入历史记录
				this.setHistor(this.videoList[this.current].courseId, this.videoList[this.current].courseDetailsId)
			},
			startPlay(current) {
				if (this.videoContext) { //判断之前是否有视频的上下文
					this.videoContext.stop()
					this.videoContext = null
				}
				let numIdCurr = this.videoList[current].courseDetailsId
				if (this.videoList[current].videoUrl) { //已经购买可直接播放
					// 播放时记录当前播放的id
					this.videoContextId = 'myVideo' + numIdCurr
					this.videoContext = uni.createVideoContext(this.videoContextId, this)
					this.$nextTick(() => {
						//播放当前的
						this.videoContext.play()
						this.$forceUpdate()
					})


				} else { //没有视频链接则表示没有权限，需要购买 弹出购买弹窗
					this.showPay = true
				}
			},
			//播放时的回调
			videoPlay(videoId) {
				//此处可以做一些自己的操作

			},
			//监听视频播放完成
			ended() {
				//此处可以自动播放下一集
				// // 查看是否有下一个视频
				// if(this.current < this.videoList.length){
				// 	// 有则自动翻到下一个视频
				// 	this.current += 1
				// }
			},
			//返回
			goBack() {
				uni.navigateBack()
			},
		},
	}
</script>

<style lang="scss">
	page {
		background-color: #000;
	}

	/deep/.u-drawer-bottom {
		background-color: #202020 !important;
	}

	.list {
		width: 100%;
		height: 100%;
		background-color: #202020;
		color: #ffffff;
		margin: 40rpx 0;

		.list-title {
			height: 100rpx;
			width: 100%;
			// padding-top: 30rpx;
			font-size: 32rpx;
			font-weight: bold;

			.list-title-icon {
				padding-left: 30rpx;
				padding-right: 30rpx;
			}

		}

		.list-item {
			padding: 0 30rpx;

			.list-item-box {
				width: 100%;
				line-height: 90rpx;
				background-color: #fff;
				color: #000;
				font-weight: bold;
				text-align: center;
				margin-bottom: 40rpx;
				border-radius: 20rpx;
			}

		}

		.list-box {
			width: 100%;
			height: calc(100% - 100rpx);
		}

		.list-box-item {
			width: calc(100% - 40rpx);
			height: auto;
			margin: 0 auto;
			margin-bottom: 20rpx;
		}

		.list-box-item-l {
			width: 140rpx;
			height: 100%;
			border-radius: 10rpx;

			image {
				width: 100%;
				height: 160rpx;
				border-radius: 10rpx;
			}
		}

		.list-box-item-r {
			width: calc(100% - 160rpx);
			height: 100%;
			color: #ffffff;
			font-size: 32rpx;

			.list-box-item-r-title {
				width: 100%;
				font-weight: bold;
			}

			.list-box-item-r-title-t {
				padding-right: 30rpx;
				font-weight: 400;
				font-size: 24rpx;
				color: #5074FF;
			}

			.list-box-item-r-titles {
				margin-top: 10rpx;

			}

			.list-box-item-r-content {
				width: 100%;
				color: rgba(255, 255, 255, 0.6);
				margin-top: 10rpx;
				overflow: hidden; //超出的文本隐藏
				display: -webkit-box;
				-webkit-line-clamp: 2; // 超出多少行
				-webkit-box-orient: vertical;
			}
		}
	}

	.swipers {
		width: 100%;
		height: 100vh;

		.swipers-item {
			width: 100%;
			height: 100%;
		}

		.swipers-items {
			width: 100%;
			height: 100%;
			position: relative;
			background-color: #000;
		}

		.swipers-items-imgsbg {
			position: absolute;
			width: 100%;
			height: 90%;
			top: 50%;
			left: 50%;
			transform: translate(-50%, -50%);
		}

		.swipers-items-video {
			width: 100%;
			height: 100%;
		}

		.swipers-items-back {
			position: absolute;
			// top: 75rpx;
			left: 20rpx;
			z-index: 999;
		}

		.swipers-items-info {
			width: 80%;
			height: auto;
			position: absolute;
			bottom: 100rpx;
			left: 20rpx;
			color: #ffffff;
			font-size: 30rpx;
			z-index: 999;

			.swipers-items-info-content {
				margin-top: 10rpx;
				font-size: 26rpx;
			}

			.swipers-items-info-num {
				margin-top: 20rpx;
			}
		}

		.swipers-items-right {
			width: 60rpx;
			position: absolute;
			right: 20rpx;
			top: 50%;
			transform: translate(0, -50%);
			z-index: 999;

			.swipers-items-right-item {
				width: 100%;
				margin-bottom: 40rpx;
			}

			.swipers-items-right-item-img {
				image {
					width: 60rpx;
					height: 60rpx;
				}
			}

			.swipers-items-right-item-txt {
				width: 100%;
				text-align: center;
				font-size: 24rpx;
				color: #ffffff;
				font-weight: bold;
				margin-top: 10rpx;
			}
		}
	}
</style>