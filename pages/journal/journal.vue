<template>
	<view class="content" @longpress="open" @click="open">
		<image class="bg" src="/static/journal.png"></image>
		<view class="info">
			<view class="info-patient" v-for="(item, index) in data" :key="index" :class="{patientCenter:index>1&&index<=6, patientBottom:index>6}">
				<view >{{ item }}</view>
			</view>
			<view class="remark">
				{{remarks}}
			</view>
		</view>
		<popupSet ref="popupSet" @confirm="confirm" iTypeText="病区号" @close="close" :dataInit="dataPopup"  :showIType="true" ></popupSet>
	</view>
</template>

<script>
import popupSet from '../../components/popup-set/popup-set.vue';
import X2JS from '../../common/xml2json.js';

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
			data: {
				0:'',
				1:'',
				2:'',
				3:'',
				4:'',
				5:'',
				6:'',
				7:'',
				8:'',
				9:'',
				10:'',
				11:'',
				12:'',
				13:'',
				14:'',
			},
			cliniqueCode: '',
			iType: '',
			popupShow: false,
			seqNumber: '',
			screenNumber: '',
			playSound:false,
			voiceDataInit:[],
			voiceData:[],
			remarks:'',
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
		//输出结果
		this.iType = uni.getStorageSync('iType') || '';
		
		if (this.iType ) {
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
			this.init();
		},
		// 初始化数据
		init() {
			if (this.popupShow) {
				return false;
			}
			// 测试使用
			// let data = String('<string xmlns=""> <record><column  name="count_no" value=""><dept><deptName>部门名称 如儿科病区</deptName><bedNum>床号用逗号隔开，如23,35,36</bedNum></dept></column><column  name="today_in" value=""><dept><bedNum>床号用逗号隔开，如23,35,36</bedNum></dept></column><column  name="mov_in" value=""><dept><bedNum>如23,35,36</bedNum></dept></column><column  name="today_out" value=""><dept><bedNum>如23,35,36</bedNum></dept></column><column  name="mov_out" value=""><dept><bedNum>如23,35,36</bedNum></dept></column><column  name="tomorrow_out" value=""><dept><bedNum>如23,35,36</bedNum></dept></column><column  name="mov_room" value=""><dept><bedNum>如23,35,36</bedNum></dept></column><column  name="count_OPS" value=""><dept><bedNum>如23,35,36</bedNum></dept></column><column  name="count_critical" value=""><dept><bedNum>如23,35,36</bedNum></dept></column><column  name="mov_bed" value=""><dept><bedNum>如23,35,36</bedNum></dept></column><column  name="blood_sugar" value=""><dept><bedNum>如23,35,36</bedNum></dept></column><column  name="count_bedsore" value=""><dept><bedNum>如23,35,36</bedNum></dept></column><column  name="count_falling" value=""><dept><bedNum>如23,35,36</bedNum></dept></column><column  name="count_pee" value=""><dept><bedNum>如23,35,36</bedNum></dept></column><column  name="count_in_out" value=""><dept><bedNum>如23,35,36</bedNum></dept></column><column  name="remarks" value=""><dept><bedNum>如23,35,36</bedNum></dept></column><column  name="remarks" value=""><dept><bedNum>明月几时有，把酒问青天。不知天上宫阙，今夕是何年。我欲乘风归去，又恐琼楼玉宇，高处不胜寒。起舞弄清影，何似在人间。</bedNum></dept></column><column  name="count_in_out" value=""><dept><bedNum>如23,35,36</bedNum></dept></column></record></string>')
			// let res = {data:{"Data":"<record><column  name=\"count_no\" value=\"17\"><dept><deptName>急诊科(病区)</deptName><bedNum>12</bedNum></dept><dept><deptName>结直肠外科(肛肠三科)(病区)</deptName><bedNum>2</bedNum></dept><dept><deptName>微创外科(病区)</deptName><bedNum>3</bedNum></dept></column><column  name=\"today_in\" value=\"\"><dept><deptName>急诊科(病区)</deptName><bedNum>10、6、4、17、13</bedNum></dept></column><column  name=\"today_out\" value=\"\"><dept><deptName>急诊科(病区)</deptName><bedNum>11、13</bedNum></dept></column><column  name=\"mov_bed\" value=\"\"><dept><deptName></deptName><bedNum></bedNum></dept></column><column  name=\"count_critical\" value=\"\"><dept><deptName>急诊科(病区)</deptName><bedNum>4、6、3</bedNum></dept></column><column  name=\"count_in_out\" value=\"\"><dept><deptName>急诊科(病区)</deptName><bedNum>6、3</bedNum></dept></column><column  name=\"custody\" value=\"\"><dept><deptName>急诊科(病区)</deptName><bedNum>4、6、3</bedNum></dept></column><column  name=\"count_falling\" value=\"\"><dept><deptName></deptName><bedNum>3、7、9、12</bedNum></dept></column><column  name=\"remarks\" value=\"\"><dept><deptName></deptName><bedNum>总值班护士长：廖秀娥~~~ ~~~ 值班医生：王榕</bedNum></dept></column><column  name=\"division\" value=\"\"><dept><deptName></deptName><bedNum></bedNum></dept></column><column  name=\"chemotherapy\" value=\"\"></column><column  name=\"tomorrow_chemotherapy\" value=\"\"></column><column  name=\"pipeLine\" value=\"\"><dept><deptName>急诊科(病区)</deptName><bedNum>6、3</bedNum></dept></column></record>"}}
			
			uni.request({
				url: 'http://129.1.20.21:8020/Queue/getQueryNursingDate',
				data: {
					dqh000: this.iType,
					types:'1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25',
				},
				method:'POST',
				timeout: 3000,
				success: res => {
					setTimeout(()=>{
						this.init();
					},10000)
					let data = res.data.Data;
					let x2js = new X2JS();
					let jsonObj = x2js.xml_str2json(res.data.Data);
					if(jsonObj.record.column.length==0){
						this.data = {
							0:'', 
							1:'',
							2:'',
							3:'',
							4:'',
							5:'',
							6:'',
							7:'',
							8:'',
							9:'',
							10:'',
							11:'',
							12:'',
							13:'',
							14:'',
						}
						return;
					}
					jsonObj.record.column.forEach((item,index)=>{
						let bedNum = '';
						try{
							bedNum = (item.dept.bedNum||'');
						}catch(e){
							bedNum = '';
						}
						switch(item._name) {
							case 'count_no':
								let total = 0;
								let text = '';
								item.dept.forEach((item)=>{
									text += item.deptName + item.bedNum
									total += +item.bedNum
								})
								this.data[0] ='总数:'+total+text;
								break;
							case 'today_in':
							   this.data[1] = item.dept.deptName + bedNum
								break;
							case 'mov_in':
							   this.data[2] = item.dept.deptName + bedNum
								break;
							case 'today_out':
							   this.data[3] = item.dept.deptName + bedNum
								break;
							case 'mov_out':
							   this.data[4] = item.dept.deptName + bedNum
								break;
							case 'tomorrow_out':
							   this.data[5] = item.dept.deptName + bedNum
								break;
							case 'mov_room':
							   this.data[6] = item.dept.deptName + bedNum
								break;
							case 'count_OPS':
							   this.data[7] = bedNum
								break;
							case 'count_bedsore':
							   this.data[8] = bedNum
								break;
							case 'mov_bed':
							   this.data[9] = bedNum
								break;
							case 'count_falling':
							   this.data[10] = bedNum
								break;
							case 'blood_sugar':
							   this.data[11] = bedNum
								break;
							case 'count_pee':
							   this.data[12] = bedNum
								break;
							case 'count_critical':
							   this.data[13] = bedNum
								break;
							case 'count_in_out':
							   this.data[14] = bedNum
								break;
							case 'remarks':
							   this.remarks = bedNum
								break;
						} 
					})
					
				},
				fail: res => {
					uni.showToast({
						title: '请求失败',
						icon: 'none'
					});
					setTimeout(()=>{
						this.init();
					},10000)
				}
			});
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
	padding-top: 145px;
	padding-left: 40px;
	padding-right: 40px;
	display: flex;
	flex-wrap: wrap;
	position: relative;
}
.remark{
	position: absolute;
	top: 395px;
	width: 405px;
	height: 646px;
	overflow: hidden;
	font-size: 36px;
	right: 40px;
	font-weight: bold;
}
.info-patient {
	height: 82px;
	line-height: 82px;
	margin-left: 150px;
	width: 1687px;
	overflow: hidden;
}
.info-patient.patientCenter{
	width: 1274px;
}
.info-patient.patientBottom{
	width: 564px;
}
.name {
    width: 468px;
    margin-left: 274px;
}
.info-patient view {
	color: #333333;
	font-size: 36px;
	font-weight: bold;
}

</style>
