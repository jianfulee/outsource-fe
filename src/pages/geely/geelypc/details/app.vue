<template>
<div id="app" class="g-main">
	<div class="m-status">
		<div v-if="unCmp">
			<img src="../../../../assets/img/受理中.png">
		</div>
		<div v-else>
			<img src="../../../../assets/img/已完成.png">
		</div>
	</div>
	<div class="m-details">
		<div class="m-list">
			<el-row :gutter="20">
				<el-col :span="8">
					<label>工单标题</label>
					<div class="f-list" :title="detailsData.title">{{ detailsData.title }}</div>
				</el-col>
				<el-col :span="8">
					<label>工单编号</label>
					<div class="f-list" :title="detailsData.id">{{ detailsData.id }}</div>
				</el-col>
				<el-col :span="8">
					<label>创建时间</label>
					<div class="f-list" :title="detailsData.createTime | time">{{ detailsData.createTime | time }}</div>
				</el-col>
			</el-row>
		</div>
		<div class="m-list">
			<el-row :gutter="20">
				<el-col :span="8">
					<label>工单分类</label>
					<div class="f-list" :title="detailsData.typeStr">{{ detailsData.typeStr }}</div>	
				</el-col>
				<el-col :span="8">
					<label>工单状态</label>
					<div class="f-list" :title="detailsData.status | status">{{ detailsData.status | status }}</div>
				</el-col>
			</el-row>
		</div>
		<div class="m-content">
			<el-col>
				<label :span="4">工单内容</label>
				<div :span="20" class="f-content">
					<div>{{ detailsData.content }}</div>	
				</div>
			</el-col>
		</div>
		<div class="m-comment">
			<el-col>
				<label>处理结果</label>
				<div  :span="20" class="f-comment">
					<div>{{ detailsData.comment }}</div>	
				</div>
			</el-col>
		</div>
		<div class="m-usermsg">
			<el-col :span="24">
				<label>用户信息</label>
					<el-row class="f-usermsg">
						<el-col :span="8">
							<label>姓名</label>
							<div class="u-usermsg" :title="detailsData.userName">{{ detailsData.userName }}</div>	
						</el-col>
						<el-col :span="8">
							<label>手机号码</label>
							<div class="u-usermobile" :title="detailsData.userMobile">{{ detailsData.userMobile }}</div>
						</el-col>
						<el-col :span="8">
							<label>邮箱</label>
							<div class="u-usermsg" :title="detailsData.userEmail">{{ detailsData.userEmail }}</div>
						</el-col>
					</el-row>
			</el-col>
		</div>
	</div>
	<hr v-if="noEvaluate">
	<div class="m-evaluate" v-if="noEvaluate">
		<el-row :gutter="20">
			<el-col :span="24">
				<label>工单评价</label>
				<div class="f-evaluate">
					<el-row>
						<el-col :span="8">
							<label>{{ detailsData.custom[0] && detailsData.custom[0].name }}</label>
							<el-rate 
								v-model="resValue" 
								:colors="['#99A9BF', '#F7BA2A', '#FF9900']" :disabled="!detailsData.canEvaluate">
							</el-rate>
						</el-col>
						<el-col :span="8">
							<label>{{ detailsData.custom[1] && detailsData.custom[1].name }}</label>
							<el-rate 
								v-model="effValue" 
								:colors="['#99A9BF', '#F7BA2A', '#FF9900']" :disabled="!detailsData.canEvaluate">
							</el-rate>
						</el-col>
						<el-col :span="8">
							<label>{{ detailsData.custom[2] && detailsData.custom[2].name }}</label>
							<el-rate 
								v-model="attValue" 
								:colors="['#99A9BF', '#F7BA2A', '#FF9900']" :disabled="!detailsData.canEvaluate">
							</el-rate>
						</el-col>
					</el-row>
				</div>	
				<div class="f-evaluate">
					<el-input
					type="textarea"
					:autosize="{ minRows: 2, maxRows: 4}"
					placeholder="其他建议或意见"
					:disabled="!detailsData.canEvaluate"
					:maxlength="500"
					v-model="advValue">
						</el-input>
				</div>
			</el-col>
		</el-row>
		<el-row :gutter="20">
			<el-col :span="4" :offset="9">
				<el-button
				type="primary"
				@click="evaluate" :disabled="!detailsData.canEvaluate">
				确认评价
				</el-button>
			</el-col>
			<el-col :span="6">
				<el-button
			    @click="cancel" :disabled="!detailsData.canEvaluate">
				取消
				</el-button>
			</el-col>
		</el-row>
	</div>
</div>
</template>

<script>
	const util = require('../../../../util.js')
	const moment = require('moment')
	export default {
		data() {
			return {
				detailsData: {
					userMobile: '',
					worksheetType: '',
				},
				unCmp: true,
				noEvaluate: false,
				resValue: 0,
				attValue: 0,
				effValue: 0,
				advValue:''
			}
		},
		created: function() {
			this.init();
			this.getInitInfo();
		},
		methods: {
			init() {
				var urlObj = util.parseQueryString(location.search);
			},
			getInitInfo() {
				var urlObj = util.parseQueryString(location.search);
				var _$$this = this;
				var filter = {
					id: urlObj.id
				}
				_$$this.$http.get('/geely/api/ticket/get/',{
						emulateJSON: true,
						params: filter
					}).then((_ret) => {
						if(_ret.body.code == 200){
							_$$this.$data.detailsData.id = urlObj.id
							_.merge(_$$this.$data.detailsData, _ret.body.result, true);
							// 工单评价是否显示以及其他建议值
							if (_ret.body.result.custom) {
								if (_ret.body.result.custom.length) {
									_$$this.$data.noEvaluate = true;
									this.$data.detailsData.custom.forEach(function(item){
										if (item.name == '其他建议或意见'){
											_$$this.$data.advValue = item.value;
										}
									})
								}
							}else if (_ret.body.result.custom.length == 0) {
								_$$this.$data.noEvaluate = false;
							}
							// 工单内容在未完结时显示为空，完结后显示最近一条操作的内容
							if(_ret.body.result.comments.length > 0){
								if(_ret.body.result.status == 20){
									_$$this.$data.detailsData.comment = _ret.body.result.comments[0].comment;
								}else {
									_$$this.$data.detailsData.comment = "";
								}
							}
							// 工单状态直观显示
							if(_ret.body.result.status == 20){
								_$$this.$data.unCmp = false;
							}else{
								_$$this.$data.unCmp = true;
							}
							// 工单状态值
							if(_ret.body.result.typeStr == ""){
								_$$this.$data.detailsData.typeStr = '未分类';
							}else{
								_$$this.$data.detailsData.typeStr = _ret.body.result.typeStr;
							}					
						}else{
							_$$this.$message.error(_ret.body.message);
						}
					}).catch((_err) => {					
						_$$this.$message.error('服务器出错了');
					}).finally((_ret) => {
						this.shuxingxing();
						this.$forceUpdate();
					})						
			},
			evaluate() {
				var _$$this = this;
				if (!_$$this.$data.resValue || !_$$this.$data.effValue || !_$$this.$data.attValue) {
					this.$message('请填写完整的三项评价');
					return;
				}
				var filter = {
					id: _$$this.$data.detailsData.id,
					consequence:_$$this.$data.resValue,
					effiency: _$$this.$data.effValue,
					attitude: _$$this.$data.attValue,
					advice: _$$this.$data.advValue
				};
				_$$this.$http.get('/geely/api/worksheet/evaluate/', {
					emulateJSON: true,
					params: filter
				}).then((_ret) => {
					if (_ret.body.code == 200) {
						_$$this.$data.detailsData.canEvaluate = false;
						_$$this.$message({
          					message: '非常感谢您对我们的评价，我们将会积极改进',
          					type: 'success'
        				});
						// 评价状态刷新
						_$$this.$forceUpdate();
					}else{
						_$$this.$message.error(_ret.body.message);
					}
				}).catch((_err) => {
					_$$this.$message.error('服务器出错了');
				})
			},
			cancel() {
				var _$$this = this;
				_$$this.$data.resValue = null;
				_$$this.$data.attValue = null;
				_$$this.$data.effValue = null;
				_$$this.$data.advValue = "";
			},
			shuxingxing() {
				var _$$this = this;
				if (this.$data.detailsData.custom) {
					this.$data.detailsData.custom.forEach(function(item){
						if (item.name.indexOf('处理结果') > -1) _$$this.$data.resValue = item.value.length;
						if (item.name.indexOf('服务态度') > -1) _$$this.$data.attValue = item.value.length;
						if (item.name.indexOf('处理时效') > -1) _$$this.$data.effValue = item.value.length;
					})
				}
			}
		},
		filters: {
				time: function(value) {
					if (!value)  return '';
					return moment(value).format('YYYY-MM-DD HH:mm:ss');
				},
				status: function(value) {
					var map = {
						5 : '受理中',
						10 : '受理中',
						20 : '已完结',
						25 : '受理中'
					}
					return map[value];
				}
			}
	}
</script>

<style>
.g-main{
	position: absolute; 
	width: 76%;
	left: 12%;
	min-width: 850px;
	color: #737475;
}
.m-status{
	margin: 20px 12px;
	text-align: center;
}
img{
	width: 80%;
	height: 16%;
}
label{
	float: left;
	color: #222222;
}
.m-list{
	height: 46px;
	background-color: #fbfdff;
	margin: 20px 0;
	border-top: 1px solid #e4e8f1;
	border-bottom: 1px solid #e4e8f1;
}
.f-list{
	float: left;
	width:70%;
	min-width: 180px;
	margin-left: 15px;
	text-overflow: ellipsis;
	white-space: nowrap;
	overflow: hidden;
}
.m-content, .m-comment{
	height: 90px;
	margin: 20px 0;
}
.el-row{
	padding: 12px 0;
}
.f-content, .f-comment{
	float: left;
    height: 88px;
    width: calc(100% - 78px);
    min-width: 750px;
    margin-left: 10px;
    background-color: #fbfdff;
    border: 1px solid #e4e8f1;
    overflow-y: auto; 
}
.f-content div, .f-comment div{
	padding-left: 5px;
}
.m-usermsg{
	height: 36px;
	padding: 10px 0;
}
.f-usermsg{
    float: left;
    width: calc(100% - 76px);
    min-width: 750px;
    margin-left: 10px;
    margin-top: -12px;
    background-color: #fbfdff;
    border: 1px solid #e4e8f1;
}
.f-usermsg .el-col{
	padding-left: 5px;
}
.u-usermsg{
	float: left;
	width:80%;
	min-width: 200px;
	margin-left: 5px;
	text-overflow: ellipsis;
	white-space: nowrap;
	overflow: hidden;
}
.u-usermobile{
	float: left;
	width:70%;
	min-width: 180px;
	margin-left: 5px; 
	text-overflow: ellipsis;
	white-space: nowrap;
	overflow: hidden;
}
.f-evaluate{
	margin: -15px 0 0 75px; 
}
.f-evaluate label{
	margin-right: 8px;
}
.el-rate{
	margin-left: 75px;
}
.el-textarea{
	margin-top: 20px;
}
</style>