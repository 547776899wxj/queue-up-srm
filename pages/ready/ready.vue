<template>
	<view class="content"  @longpress="open" @click="open">
		<image class="bg" src="/static/ready.png" ></image>
		<view class="header">
			<view class="header-time">
				<view class="pr-15">
					<text>{{ dateText.year }}年{{ dateText.month }}月{{ dateText.date }}日</text>
				</view>
				<view >
					<text>{{ dateText.time }}</text>
				</view>
			</view>
		</view>
		<view class="header-title">{{title}}</view>
		<view class="info">
			<view class="info-item" v-if="data.length>0">
				<view class="item-user">
					<view class="info-item-nubmer mt-15">{{data[0].number}} {{data[0].name}}</view>
					<view class="info-item-nubmer mt-15">{{data[0].number}} {{data[0].name}}</view>
				</view>
				<view class="info-item-title">{{data[0].room}}</view>
			</view>
			<view class="info-item mt-39" v-if="data.length>1">
				<view class="item-user">
					<view class="info-item-nubmer mt-15">{{data[1].number}} {{data[1].name}}</view>
					<view class="info-item-nubmer mt-15">{{data[1].number}} {{data[1].name}}</view>
				</view>
				<view class="info-item-title">{{data[1].room}}</view>
			</view>
			<view class="info-item mt-15" v-if="data.length>2">
				<view class="item-user">
					<view class="info-item-nubmer mt-15">{{data[2].number}} {{data[2].name}}</view>
					<view class="info-item-nubmer mt-15">{{data[2].number}} {{data[2].name}}</view>
				</view>
				<view class="info-item-title">{{data[2].room}}</view>
			</view>
			<view class="info-item mt-20" v-if="data.length>3">
				<view class="item-user">
					<view class="info-item-nubmer mt-15">{{data[3].number}} {{data[3].name}}</view>
					<view class="info-item-nubmer mt-15">{{data[3].number}} {{data[3].name}}</view>
				</view>
				<view class="info-item-title">{{data[3].room}}</view>
			</view>
		</view>
		<uni-popup ref="popup" type="center" :maskClick="false">
			<view class="popup">
				<view class="popup-header">设置</view>
				<view>
					<view class="uni-form-item ">
						<view class="popup-title">标题：</view>
						<input class="uni-input" v-model="title" placeholder="请输入标题" />
					</view>
					<view class="uni-form-item uni-form-btn">
						<view class="popup-title">屏幕：</view>
						<input class="uni-input" v-model="screenNumber" type="number" placeholder="第一个屏幕输入:1" />
					</view>
					<view class="uni-form-item "><button type="default" class="chooseBtn" @click="navTo()">选择页面</button></view>
					
					<view class="uni-form-item ">
						<button class="popup-btn" @click="close">取消</button>
						<button class="popup-btn" @click="confirm">确定</button>
					</view>
				</view>
			</view>
		</uni-popup>
	</view>
</template>

<script>
	import uniPopup from '@/components/uni-popup/uni-popup.vue'
	export default {
		data() {
			return {
				dateText: {
					year: '',
					month: '',
					date: '',
					day: '',
					time: ''
				},
				title:'麻醉检查室',
				weekday: [],
				data:[
					// {
					// 	room:'科室1',
					// 	number:'A1002',
					// 	name:'张无忌',
					// },
				
				],
				cliniqueCode:'',
				iType:'',
				popupShow:false,
				seqNumber:'',
				screenNumber: '',
				playSound:false,
				voiceDataInit:[],
				voiceData:[],
				test:'测试',
				testNubmer:0,
			}
		},
		onLoad() {
			this.screenNumber = uni.getStorageSync('screenNumber') || '';
			this.title = uni.getStorageSync('title') || '';
			let date = new Date();
			this.weekday = new Array(7);
			this.weekday[0] = '星期日';
			this.weekday[1] = '星期一';
			this.weekday[2] = '星期二';
			this.weekday[3] = '星期三';
			this.weekday[4] = '星期四';
			this.weekday[5] = '星期五';
			this.weekday[6] = '星期六';
			if(this.screenNumber){
				this.init();
			}
		},
		methods: {
			//选择页面
			navTo(){
				uni.setStorageSync('pageSetBoolean',false);
				uni.redirectTo({
					url: '../index/index',
				});
			},
			//当前时间
			newDate(dataTime) {
				let date = new Date(dataTime);
				this.dateText = {
					year: date.getFullYear(),
					month: date.getMonth() + 1,
					date: date.getDate(),
					day: this.weekday[date.getDay()],
					time: date.getHours() + ':' + (date.getMinutes() < 10 ? '0' + date.getMinutes() : date.getMinutes())
				};
			},
			// 打开设置
			open(){
				this.$refs.popup.open();
				this.popupShow = true;
			},
			// 关闭设置
			close(){
				this.$refs.popup.close();
				this.popupShow = false;
				
			},
			//确定设置
			confirm(){
				uni.showLoading({
					title: '加载中',
				});
				uni.setStorageSync('screenNumber', this.screenNumber);
				uni.setStorageSync('title', this.title);
				this.popupShow = false;
				this.init();
				this.$refs.popup.close();
				uni.hideLoading();
			},
			// 初始化数据
			init(){
				if(this.popupShow){
					return false;
				}
				
				// 测试使用
				// let res = {data:{"Data":[
				// {"ername":"检查室二","patientcode":"10-03","patientname":"林新梅","lb":"EDO","call_time":"10:16:23","wait_status":"6","nextName":"潘子敏","nextCode":"10-04","room_name":null,"call_time1":null},
				
				// {"ername":"检查室一","patientcode":"10-04","patientname":"潘子敏","lb":"EDO","call_time":"10:16:31","wait_status":"6","nextName":"蒲维奇","nextCode":"14-02","room_name":null,"call_time1":null},
				
				// {"ername":"检查室三","patientcode":"14-02","patientname":"蒲维奇","lb":"EDO","call_time":"14:42:00","wait_status":"6","nextName":"张秋萍","nextCode":"14-03","room_name":null,"call_time1":null},
				
				// {"ername":"检查室四","patientcode":"14-03","patientname":"张秋萍","lb":"EDO","call_time":"15:14:27","wait_status":"6","nextName":"","nextCode":"","room_name":null,"call_time1":null}],
				
				// "ServiceTime":"2020-09-16 10:57:58"}
				// }
				
				uni.request({
				    url: 'http://129.1.20.21:8019/Queue/MZ_Get_Queue', 
					data:{
						room_name_type: this.screenNumber,
					},
					timeout:3000,
				    success: (res) => {
						let datas = res.data.Data;
						let dataMaps = [];
						this.newDate(res.data.ServiceTime);
						datas.forEach((data,index) =>{
							let dataMap = {
								room: data.room_name||data.ername,
								number: data.patientcode||'',	
								name: this.$util.hideName(data.patientname),
								nextName:this.$util.hideName(data.nextName),
								nextCode:data.nextCode,
							}
							dataMaps = dataMaps.concat(dataMap)
						})
						this.data = dataMaps;
						setTimeout(() => {
							this.init();
						}, 5000);
				    },
					fail:(res) => {
						uni.showToast({
							title:'请求失败',
							icon:'none'
						})
						setTimeout(() => {
							this.init();
						}, 5000);
					}
				});
			},
		}
	}
</script>

<style>
.pr-15{
	padding-right: 15px;
}
.pl-15{
	padding-left: 15px;
}
page {
	height: 100%;
}
.uni-form-item.uni-form-btn{
	padding: 0;
}

.chooseBtn{
	font-size: 30px;
	width: 438px;
}
.header{
	display: flex;
	justify-content: center;
	align-items: center;
	position: relative;
	height: 118px;
}
.room{
	width: 735px;
	margin-left: 50px;
}
.content {
	position: relative;
	height: 100%;
}

.bg {
	position: absolute;
	width: 1080px;
	height: 1920px;
	z-index: -1;
}
.header-time {
	display: flex;
	position: absolute;
	justify-content: center;
	right: 45px;
	padding-top: 15px;
	top: 70px;
}
.header-title{
	color: rgb(253,250,7);
	font-size: 63px;
	font-weight: 800;
	letter-spacing: 12px;
	text-align: center;
	 height: 123px;
	 margin-top: 54px;
}
.header-time view {
	font-size: 38px;
	color: #000;
	letter-spacing:5px;
}
.info{
	margin-left: 35px;
	margin-right: 35px;
}
.info-item{
	height: 360px;
	color: #fff;
	display: flex;
	justify-content: center;
	align-items: center;
	
}
.mt-39{
	margin-top: 39px;
}
.mt-15{
	margin-top: 15px;
}
.mt-20{
	margin-top: 20px;
}
.info-item:first-child{
	height: 416px;
}

.info-item-nubmer{
	font-size: 65px;
	width: 580px;
	text-align: center;
}
.info-item-title{
	font-size: 80px;
	width: 455px;
	text-align: center;
	overflow: hidden;
}
.info-patient {
	display: flex;
	height: 234px;
}
.name{
	width: 735px;
	margin-left: 143px;
}
.info-patient view {
	font-size: 60px;
	color: #000;
	display: flex;
	align-items: center;
	justify-content: center;
	white-space: nowrap;
	overflow: hidden;
	text-overflow: ellipsis;
}

.popup-btn{
		font-size: 30px;
		color: #fff;
		padding-left: 40px;
		padding-right: 40px;
		background-color: rgb(68,114,196);
		margin-left: 40px;
		margin-right: 40px;
	}
	.popup{
		background-color: #fff;
		width: 600px;
		font-size: 40px;
		z-index: 100;
	}
	.popup-header{
		background-color: rgb(68,114,196);
		text-align: center;
		padding: 10px 0 ;
	}
	.uni-form-item{
		display: flex;
		align-items: center;
		padding: 40px;
		justify-content: center;
	}
	.popup-title{
		font-size: 30px;
	}
	.uni-input{
		font-size: 25px;
		border: 1px solid;
		padding: 20px 30px;
	}
</style>
