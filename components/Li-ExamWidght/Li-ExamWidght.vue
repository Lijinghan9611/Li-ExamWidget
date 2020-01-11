<template>
	<view>
		<view class="exam-item-testing">
			<view class="exam-item-inner" v-if='dataList.length > 0'>
				<view class="line_under exam-item-title">
					{{currentIndex+1}}.{{currentItem.fldName?currentItem.fldName:''}}
				</view>
				<!--  单选 -->
				<view v-show="checkShow(0)">
					<view class="exam-item-option" v-for="(item,rowIndex_1) in currentItem.QuestionOptionList" v-bind:key='rowIndex_1'
					 @click="radioChange(item)">
						<radio :checked="item.fldOptionIndex == currentCheck" />
						<text>{{checkNum(item.fldOptionIndex)}}</text>.<text>{{item.fldOptionText}}</text>
					</view>
				</view>

				<!-- 多选 -->
				<checkbox-group @change="checkboxChange" v-show="checkShow(1)">
					<view v-for="(item,rowIndex_2) in currentItem.QuestionOptionList" v-bind:key='rowIndex_2'>
						<view class="exam-item-option">
							<checkbox :value="setCheckboxVal(item.fldOptionIndex)" :checked="checkboxCheck(item.fldOptionIndex)" />
							<text>{{checkNum(item.fldOptionIndex)}}</text>.<text>{{item.fldOptionText}}</text>
						</view>
					</view>
				</checkbox-group>

				<!--  填空题 -->
				<view v-show="currentItem.questionType == 2">
					<view class="exam-item-option">
						<textarea @blur="bindTextAreaBlur" @input='bindTextinput' placeholder='请输入您的答案' :value='getTxtValue(currentIndex)' />
						</view>
				</view>

				<view class="exam-button-row">
					<view>
						<button size='mini' @tap="finish" style="margin-right: 20rpx;">{{finishText}}</button>
						<button v-show="currentIndex != 0" size='mini' @tap="lastQuestion" style="margin-right: 20rpx;">{{lastText}}</button>
						<button v-show="currentIndex < dataList.length-1"   size='mini' @tap="nextQuestion">{{nextText}}</button>
						<text v-show="showIndexText" @tap="switchIndexBox"  style="float: right;color: gray;">
							<text v-if='numBoxType == 0'>{{showIndexBox?'▽':'△'}}</text>
							{{indexText}}
						</text>
					</view>
				</view>
				
				<view v-show="showIndexBox && numBoxType == 0" class="exam-indexbox">
					<view class="exam-indexbox-item" 
					v-bind:style="{'background':quesIndex == currentIndex? '#F0F0F0':''}"
					v-bind:class="{ 'exam-indexbox-item-selected': checkItem(item)}"
					v-for="(item,quesIndex) in dataList"
					v-bind:key='item.questionID' 
					@click="currentSelectFinish(quesIndex)">
					{{quesIndex + 1}}
					</view>
				</view>
			</view>
		</view>
		
		<view class="exam-mask" v-show='showMask' @tap="hideMask"></view>
		<view :animation="indexboxAnimationData" class="exam-mask-content" v-show='showMask'>
			<view class="exam-indexbox">
				<view class="exam-indexbox-item-mask" 
				v-bind:style="{'background':quesIndex == currentIndex? '#F0F0F0':''}"
				v-bind:class="{ 'exam-indexbox-item-selected': checkItem(item)}"
				v-for="(item,quesIndex) in dataList"
				v-bind:key='item.questionID' 
				@click="currentSelectFinish(quesIndex)">
				{{quesIndex + 1}}
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		name: 'Li-ExamWidght',
		props: {
			index: {
				info: Number,
				default: 0
			},
			dataList: {
				type: Array,
				default:function(){
					return [];
				}
			},
			finishText:{
				info: String,
				default: '完成'
			},
			lastText:{
				info: String,
				default: '上一题'
			},
			nextText:{
				info: String,
				default: '下一题'
			},
			indexText:{
				info: String,
				default: '题号'
			},
			showIndexText: {  //题号文字是否显示，可用于外部切换
				info: Boolean,
				default: true
			},
			numBoxType: {  //题号框显示方式
				info: Number,
				default: 0
			},
			numBoxShow: {  //题号框是否显示，可用于外部切换
				info: Boolean,
				default: false
			}
		},
		data() {
			return {
				windowHeight:500,
				showIndexBox:false,
				currentHasChange:false,
				currentIndex:0,
				currentItem:{}, //当前项
				currentCheck: undefined, //单选选定
				currentCheckBoxCheck: [] ,//多选选定
				currentText:""  ,//填空题
				showMask:false  ,//是否显示遮罩和题号
				indexboxAnimationData:[]  //题号区域动画
			};
		},
		watch: {
			index: function(val, oldVal) {
				this.currentIndex = val;
				this.currentSelectFinish(this.currentIndex);
				//console.log('new: %s, old: %s', val, oldVal)
			},
			dataList: function(val, oldVal) {
				this.currentItem = this.getCurrentItem();
			},
			numBoxShow: function(val, oldVal) {
				this.switchIndexBox();
			}
		},
		computed: {
			
		},
		created(){
			let that = this;
			uni.getSystemInfo({
			    success: function (res) {
					that.windowHeight = res.windowHeight;
					if(that.numBoxType == 1){
						that.hideMask()
					}
			    }
			});
		},
		methods: {
			checkShow(type){
				if(this.dataList.length == 0){
					return false;
				}		
				if(this.currentItem.questionType == type && this.currentItem.QuestionOptionList!= undefined)
					return true;
				else
					return false;
			},
			
			checkItem(item){
				if(!item.fldAnswer)
					return false;
				let res = (item.questionType != 1 && item.fldAnswer) || (item.questionType == 1 && item.fldAnswer.length != 0)
				if(res)
					return true;
				else
					return false;
			},
			radioChange(item) {  //单选改变
				this.currentHasChange = true;
				this.currentCheck = item.fldOptionIndex;
				this.dataList[this.currentIndex].fldAnswer = item.fldOptionIndex;
				this.$emit('select', {
					question: this.getCurrentItem(),
					anwser: item
				});
			},
			checkboxChange(e) {  //多选改变
				this.currentHasChange = true;
				this.currentCheckBoxCheck = e.detail.value;
				this.dataList[this.currentIndex].fldAnswer = this.currentCheckBoxCheck.join(',');
				this.$emit('select', {
					question: this.getCurrentItem(),
					anwser: this.dataList[this.currentIndex].fldAnswer
				});
			},
			bindTextAreaBlur(e){  //填空结束
				// this.currentText = e.detail.value;
				// this.currentHasChange = true;
				// this.dataList[this.currentIndex].fldAnswer = this.currentText;
			},
			bindTextinput(e){  //填空值改变
				this.currentText = e.detail.value;
				this.currentHasChange = true;
				this.dataList[this.currentIndex].fldAnswer = this.currentText;
			},
			setCheckboxVal(val) {  //设置多选框的值，如果是数字，则转换成字符串
				if (typeof(val) == 'number')
					return val + "";

				return val;
			},
			checkboxCheck(num) { //检查多选框是否已经选定
				let that = this;
				for (let i = 0; i < that.currentCheckBoxCheck.length; i++)
					if (that.currentCheckBoxCheck[i] == num)
						return true;

				return false;
			},
			getTxtValue(currentIndex){
				return this.currentItem.fldAnswer?this.currentItem.fldAnswer:""
			},
			checkNum(num) { //把数字转换为字母
				if (typeof(num) == 'number')
					return String.fromCharCode(0x60 + num).toUpperCase();
				else
					return num;
			},
			lastQuestion() {  //上一题
				let newIndex = this.currentIndex - 1;
				this.currentSelectFinish(newIndex);
			},
			nextQuestion() {  //下一题
				let newIndex = this.currentIndex + 1;
				this.currentSelectFinish(newIndex);
			},
			currentSelectFinish(newIndex){   //切换题目
				let that = this;
				let __oldIndex = that.currentIndex;
				let __newIndex = newIndex;
				if(!that.dataList[__oldIndex])
					return;
					
				if(that.numBoxType == 1 && that.showMask){  //隐藏遮罩
					that.hideMask();
				}
				
				if(that.dataList[__oldIndex].questionType == 0){
					that.$emit('selectFinish',
					{ 
						currentItem:{  //当前项
							question: that.dataList[__oldIndex],
							anwser: that.currentCheck,
							hasChange:that.currentHasChange,
							index:__oldIndex,
							total:that.dataList.length
						},
						newItem:{  //新项
							question: that.dataList[__newIndex],
							index:__newIndex,
							total:that.dataList.length
						}
					});
				}
				else if(that.dataList[__oldIndex].questionType == 1){
					that.$emit('selectFinish', 
					{
						currentItem:{  //当前项
							question: that.dataList[__oldIndex],
							anwser: that.currentCheckBoxCheck?that.currentCheckBoxCheck.join(','):'',
							hasChange:that.currentHasChange,
							index:__oldIndex,
							total:that.dataList.length,
						},
						newItem:{  //新项
							question: that.dataList[__newIndex],
							index:__newIndex,
							total:that.dataList.length
						}
					});
				}
				else if(that.dataList[__oldIndex].questionType == 2){
					that.$emit('selectFinish', 
					{
						currentItem:{  //当前项
							question: that.dataList[__oldIndex],
							anwser: that.currentText,
							hasChange:that.currentHasChange,
							index:__oldIndex,
							total:that.dataList.length,
						},
						newItem:{  //新项
							question: that.dataList[__newIndex],
							index:__newIndex,
							total:that.dataList.length
						}
					});
				}
				
				that.currentIndex = newIndex;
				that.currentItem = that.getCurrentItem();
				that.currentHasChange = false;
				
				that.checkQuestionSelected();
			},
			checkQuestionSelected(){  //初始化选择
				if(this.currentItem.questionType == 0){
					this.currentCheck = this.currentItem.fldAnswer;//单选选定
				}
				else if(this.currentItem.questionType == 1){
					this.currentCheckBoxCheck = this.currentItem.fldAnswer?this.currentItem.fldAnswer.split(','):[]; //多选选定
				}
				else if(this.currentItem.questionType == 2){
					this.currentText = this.currentItem.fldAnswer?this.currentItem.fldAnswer:[]; //多选选定
				}
			},
			finish(){  //全部选择完成
				this.$emit('finish', {
					questions: this.dataList
				});
			},
			getCurrentItem(){
				if(this.dataList.length == 0)
					return {};
				if(this.dataList[this.currentIndex])
					return this.dataList[this.currentIndex];
				else
					return {};
			},
			hideMask(){  //隐藏遮罩
				let that = this;
				//创建动画
				let animation = uni.createAnimation({
					duration: 300,
					timingFunction: 'ease',
				})
				let height = that.windowHeight/2;
				animation.translateY(height).step();
				this.indexboxAnimationData = animation.export();
				setTimeout(function(){
					that.showMask = false;
				}, 300);
			},
			showMaskFun(){
				let that = this;
				that.showMask = true;
				//创建动画
				let animation = uni.createAnimation({
					duration: 300,
					timingFunction: 'ease',
				})
				animation.translateY(0).step();
				setTimeout(function(){
					that.indexboxAnimationData = animation.export()
				}, 50);
			},
			switchIndexBox(){  //题号切换
				if(this.numBoxType == 0)
					this.showIndexBox = !this.showIndexBox;
				else if(this.numBoxType == 1){
					this.showMaskFun();
				}
			}
		}
	}
</script>

<style>
	page{
		font-size: 28rpx;
	}
	.exam-item-testing {
		margin: 20rpx 10rpx 20rpx 10rpx;
		border-radius: 10rpx;
		background-color: #fff;
	}

	.exam-item-inner {
		padding: 0 20rpx;
	}

	.exam-item-title {
		font-size: 35rpx;
		padding: 10rpx 0;
	}

	.exam-item-option {
		font-size: 30rpx;
		padding: 10rpx 0;
	}
	.exam-item-option textarea{
		border:1px solid gainsboro;
		border-radius: 10rpx;
		height:200rpx;
		width: 100%;
	}

	.exam-button-row {
		text-align: center;
	}
	
	.exam-indexbox{
		padding-bottom: 20rpx;
	}

	.exam-indexbox:before, .exam-indexbox:after{
		display: table;
		content: ' ';
	}
	.exam-indexbox:after{
		clear: both;
	}
	.exam-indexbox-item{
		text-align: center;
		vertical-align: middle;
		line-height: 56rpx;
		float: left;
		border:1rpx solid gainsboro;
		height: 56rpx;
		width: 56rpx;
		margin: 5rpx;
		padding: 10rpx;
		border-radius: 10rpx;
		background-color: #fff;
		-moz-box-shadow: 0px 1px 1px #ABABAB;
		-webkit-box-shadow: 0px 1px 1px #ABABAB;
		box-shadow: 0px 1px 1px #ABABAB;
	}
 
	.exam-indexbox-item-selected{
		color: #007AFF;
	}

	/* 以下为实现0.5px底部边界 */
	.line_under {
		position: relative;
		/* .line_under:before{顶部top: 0;background: #000;} */
	}

	.line_under:before,
	.line_under:after {
		position: absolute;
		content: " ";
		height: 1px;
		width: 100%;
		left: 0;
		transform-origin: 0 0;
		-webkit-transform-origin: 0 0;
	}

	.line_under:after {
		bottom: 0;
		border-bottom: 1px solid gainsboro;
	}

	@media only screen and (-webkit-min-device-pixel-ratio: 1.5) {

		.line_under:after,
		.line_under:before {
			-webkit-transform: scaleY(0.667);
		}
	}

	@media only screen and (-webkit-min-device-pixel-ratio: 2) {

		.line_under:after,
		.line_under:before {
			-webkit-transform: scaleY(0.5);
		}
	}
	
	.exam-mask{
		position: absolute;
		z-index: 1;
		width: 100%;
		height: 100%;
		background-color: #999999;
		opacity:0.5;
		left: 0;
		top:0;
	}
	
	.exam-mask-content{
		position: absolute;
		z-index: 1;
		width: 100%;
		height: 50%;
		background-color: white;
		left: 0;
		bottom:0;
		padding: 15rpx 5rpx;
		
	}
	
	.exam-indexbox-item-mask{
		text-align: center;
		vertical-align: middle;
		line-height: 63rpx;
		float: left;
		border:1rpx solid gainsboro;
		height: 63rpx;
		width: 63rpx;
		margin: 5rpx;
		padding: 10rpx;
		border-radius: 10rpx;
		background-color: #fff;
		-moz-box-shadow: 0px 1px 1px #ABABAB;
		-webkit-box-shadow: 0px 1px 1px #ABABAB;
		box-shadow: 0px 1px 1px #ABABAB;
	}
</style>
