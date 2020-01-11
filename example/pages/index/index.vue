<template>
	<view>
		<exam-widght 
		:dataList='QuestionList' 
		:index="index" 
		@select='selectItem' 
		@selectFinish='selectFinish' 
		@finish='finish'
		:numBoxType='0'
		:showIndexText='showIndexText'
		:numBoxShow='numBoxShow'
		>
		</exam-widght>
		
		<view style="text-align: center;padding: 20rpx;">------------分割线-----------</view>
		<view>
			<button size="mini" @tap="jump()">跳转10题</button>
			 
			<button style="margin-left: 20rpx;" size="mini" @tap="switchIndexBox()">题号显示切换</button>
		</view>
	</view>
</template>

<script>
	import examWidght from '@/components/Li-ExamWidght/Li-ExamWidght.vue';
	var _this;
	export default {
		components: {
			examWidght
		}, 
		data() {
			return {
				fldTestPaperID: undefined,
				fldTestRecordID: undefined,
				showIndexText:true, //题号文字是否显示
				numBoxShow:false, 
				items: [],
				index: 0,
				QuestionList: [],
			}
		},
		onLoad(opt) {
			console.log(111)
			_this = this;
			setTimeout(function(){
				_this.getTestPaper();
			},100)
		},
		onUnload() {},
		methods: {
			jump(){
				this.index = 9;
			},
			switchIndexBox(){
				this.numBoxShow = !this.numBoxShow;
			},
			finish(item) {
				console.log("-----finish------")
				console.log(item)
			},
			selectItem(item) {
				console.log("-----selectItem------")
				console.log(item)
			},
			selectFinish(item) {
				console.log("-----selectFinish------")
				console.log(item)
				let index = item.newItem.index + 1;
				let title = index + '/' + item.newItem.total

				if (item.newItem.question.questionType == 0)
					title += "  单选题";
				else if (item.newItem.question.questionType == 1)
					title += "  多选题";
				else
					title += "  填空题";
				uni.setNavigationBarTitle({
					title: title
				});
			},
			getTestPaper() {
				let data = require('../../static/data.json')  ;
				console.log(data.QuestionList)
				_this.QuestionList = data.QuestionList;
				if (_this.QuestionList.length > 0) {
					let title = '1/' + _this.QuestionList.length
					if (_this.QuestionList[0].questionType == 0)
						title += "  单选题";
					else if (_this.QuestionList[0].questionType == 1)
						title += "  多选题";
					else
						title += "  填空题";
					uni.setNavigationBarTitle({
						title: title
					});
				}
			}
		}
	}
</script>

<style>

</style>
