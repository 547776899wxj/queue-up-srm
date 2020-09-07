<template>
	<view class="content" @longpress="open" @click="open">
		<image class="bg" src="/static/famous.png"></image>
		<view class="header">
			<!-- <view class="header-title">{{ title }}</view> -->
			<view class="header-time">
				<view>
					<text>{{ dateText.year }}年{{ dateText.month }}月{{ dateText.date }}日</text>
				</view>
				<view>
					<text class="pr-15">{{ dateText.day }}</text>
					<text>{{ dateText.time }}</text>
				</view>
			</view>
		</view>
		<view class="info">
			<view class="info-patient" v-for="(item, index) in data" :key="index">
				<view class="name">
					<!-- <text class="pr-15" v-if="item.number">{{ item.number }}号</text> -->
					<text class="pl-15">{{ item.name }}</text>
				</view>
				<view class="room">{{ item.room }}</view>
			</view>
		</view>
		<popupSet ref="popupSet" @confirm="confirm" @close="close" :dataInit="dataPopup" :showPlaySound="true" :showIType="true" :showScreenNumber="true"></popupSet>
	</view>
</template>

<script>
import popupSet from '../../components/popup-set/popup-set.vue';
// #ifdef APP-PLUS
	var FvvUniTTS = uni.requireNativePlugin('Fvv-UniTTS');
// #endif

export default {
	components: { popupSet },
	data() {
		return {
			dateText: {
				year: '',
				month: '',
				date: '',
				day: '',
				time: ''
			},
			title: '',
			weekday: [],
			data: [
				// {
				// 	room:'科室1',
				// 	number:'A1002',
				// 	name:'张无忌',
				// },
			],
			cliniqueCode: '',
			iType: '',
			popupShow: false,
			seqNumber: '',
			screenNumber: '',
			playSound:false,
			voiceDataInit:[],
			voiceData:[],
			
			test: '测试',
			testNubmer: 0,
			dataPopup:{
				title:'',
				iType:'',
				screenNumber:'',
				playSound:false,
			}
		};
	},
	onLoad() {
		this.iType = uni.getStorageSync('iType') || '';
		this.screenNumber = uni.getStorageSync('screenNumber') || '';
		this.playSound = uni.getStorageSync('playSound') || false;
		let date = new Date();
		this.weekday = new Array(7);
		this.weekday[0] = '星期日';
		this.weekday[1] = '星期一';
		this.weekday[2] = '星期二';
		this.weekday[3] = '星期三';
		this.weekday[4] = '星期四';
		this.weekday[5] = '星期五';
		this.weekday[6] = '星期六';
		this.newDate();
		setTimeout(() => {
			this.newDate();
			setInterval(() => {
				this.newDate();
			}, 60000);
		}, date.getSeconds() * 1000);
		if (this.iType && this.screenNumber) {
			this.init();
			this.dataPopup.iType = this.iType;
			this.dataPopup.screenNumber = this.screenNumber;
			this.dataPopup.playSound = this.playSound;
			
		}
	},
	methods: {
		//当前时间
		newDate() {
			let date = new Date();
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
			this.$refs.popupSet.open();
			this.popupShow = true;
		},
		// 关闭设置
		close(){
			this.popupShow = false;
		},
		//确定设置
		confirm(res) {
			this.iType = res.iType;
			this.screenNumber = res.screenNumber;
			this.playSound = res.playSound;
			this.popupShow = false;
			this.data = [];
			this.init();
		},
		// 初始化数据
		init() {
			if (this.popupShow) {
				return false;
			}
			this.data = [];
			// 测试使用
			let datas = [{"PATIENTNAME":"王素霞","LB":"CT","ROOM_NAME":"64排CT","WAIT_STATUS":"4","CALL_TIME1":"16:31:40","PATIENTCODE":"2-808","ERNAME":"64排CT","CALL_TIME":"16:31:40"},

			{"PATIENTNAME":"吴良付","LB":"EDO","ROOM_NAME":"检查室二","WAIT_STATUS":"6","CALL_TIME1":"15:32:53","PATIENTCODE":"14-03","ERNAME":"检查室二","CALL_TIME":"15:32:53"},

			{"PATIENTNAME":"田江芬","LB":"EDO","ROOM_NAME":"检查室三","WAIT_STATUS":"4","CALL_TIME1":"16:26:29","ERNAME":"检查室三","CALL_TIME":"16:26:29"}];
			datas[0].PATIENTCODE = datas[0].PATIENTCODE + this.testNubmer++
			datas[0].PATIENTNAME = datas[0].PATIENTNAME + this.testNubmer
			let voiceDataInit = [];
			datas.forEach((data, index) => {
				let name = this.$util.hideName(data.PATIENTNAME);
				let dataMap = {
					room: data.ROOM_NAME,
					number:'',
					name: name
				};
				this.data = this.data.concat(dataMap);
				if(name && this.playSound){
					let number = this.$util.chineseNumeral(dataMap.number+'')||'';
					number = number?number+'号,':'';
					let speakText = `请,${number}${data.PATIENTNAME}到,${dataMap.room}检查`;
					if(this.data.length==0){
						this.voiceData.push(speakText);
						this.voiceDataInit.push(speakText);
					}else{
						voiceDataInit = voiceDataInit.concat(speakText);
					}
				}
			});
			if(this.playSound){
				if(voiceDataInit.length>0){
					this.voiceData = this.$util.findDifferentElements(voiceDataInit,this.voiceDataInit);
					this.voiceDataInit = voiceDataInit;
				}
				if(this.voiceData.length>0){
					this.voiceQueue();	
				}else{
					setTimeout(() => {
						this.init()
					}, 5000);
				}
			}else{
				setTimeout(() => {
					this.init();
				}, 5000);
			}			

			// uni.request({
			// 	url: 'http://129.1.20.21:8019/Queue/EXAM_Get_Queue',
			// 	// url: 'http://192.168.0.159:8018/Queue/Get_Queue',
			// 	data: {
			// 		lb: this.iType,
			// 		room_name_type: this.screenNumber,
			// 	},
			// 	timeout: 3000,
			// 	success: res => {
			// 		let datas = res.data.Data;
					
			// 	},
			// 	fail: res => {
			// 		uni.showToast({
			// 			title: '请求失败',
			// 			icon: 'none'
			// 		});
			// 	}
			// });
		},
		// 语音队列
		voiceQueue(){
			// #ifdef APP-PLUS
				FvvUniTTS.init((callback) => {
					FvvUniTTS.setSpeechRate(60);
					FvvUniTTS.speak({
						text:this.voiceData[0]
					});
				});
			// #endif
			if(this.voiceData.length>1){
				this.onDone(this.voiceData[1]);
			}else{
				let date = 4300;
				if(this.voiceData[0].length>12){
					date = date + ((this.voiceData[0].length - 12)*300 ) 
				}
				setTimeout(() => {
					this.init()
				}, date);
			}
		},
		// 播放完执行
		onDone(data){
			let date = 4300;
			if(data.length>12){
				date = date + ((data.length - 12)*300 ) 
			}
			setTimeout(() => {
				this.voiceData.shift();
				if(this.voiceData.length>0){
					this.voiceQueue()
				}
			}, date);
			
		},
	}
};
</script>

<style>
.pr-15 {
	padding-right: 15px;
}
.pl-15 {
	padding-left: 15px;
}
page {
	height: 100%;
}
.header {
	display: flex;
	justify-content: center;
	align-items: center;
	position: relative;
	height: 300px;
}
.room {
    width: 665px;
    margin-left: 71px;
}
.content {
	position: relative;
	height: 100%;
}

.bg {
	position: absolute;
	height: 1080px;
	width: 1920px;
	z-index: -1;
}
.header-time {
	display: flex;
	position: absolute;
	justify-content: center;
	flex-direction: column;
    right: 117px;
    padding-top: 15px;
    bottom: 60px;
}
.header-title {
	color: rgb(253, 250, 7);
	font-size: 63px;
	font-weight: 800;
	letter-spacing: 12px;
}
.header-time view {
	font-size: 40px;
	color: #000;
	letter-spacing: 5px;
}
.info {
	padding-left: 35px;
	padding-right: 35px;
}
.info-patient {
	display: flex;
	height: 128px;
}
.name {
    width: 468px;
    margin-left: 274px;
}
.info-patient view {
	font-size: 60px;
	color: rgb(205,44,47);
	display: flex;
	align-items: center;
	justify-content: center;
	white-space: nowrap;
	overflow: hidden;
	text-overflow: ellipsis;
}

</style>
