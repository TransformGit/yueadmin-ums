<template>
	<section>
		<div v-title :data-title="this.$route.name"></div>
		<el-row :gutter="20">
			<el-col :span="7">
				<el-card v-loading="loading">
					<div class="account-info">
						<!-- <el-popover
						  ref="avatarPop"
						  placement="top"
						  title=""
						  width="200"
						  trigger="hover"
						  :content="avatarUrl ? '更换头像' : '上传头像'"
						  popper-class="text-center">
						</el-popover> -->
						<div class="avatar" @click="uploadAvatar">
							<img v-if="userInfo.avatarUrl" :src="userInfo.avatarUrl" title="点击更新头像">
							<img v-else src="../../assets/img/avatar.gif" title="点击上传头像" alt="头像"/>
						</div>
						<h3>{{userInfo.name}}</h3>
						<ul class="list-group">
							<li class="list-group-item active">
								<a href="javascript:;" title="编辑" @click="handleEdit">		   账户信息
									<i class="fa fa-edit fa-lg"></i>
								</a>
							</li>
							<li class="list-group-item">
								<label>邮箱</label>
								<span>{{userInfo.email}}</span>
							</li>
							<li class="list-group-item">
								<label>姓名</label>
								<span v-if="userInfo.realname">{{userInfo.realname}}</span>
								<span v-else>未设置</span>
							</li>
							<li class="list-group-item">
								<label>性别</label>
								<span>{{userInfo.sexual === 1 ? '男' : userInfo.sexual === 0 ? '女' : '未知'}}</span>
							</li>
							<li class="list-group-item">
								<label>QQ</label>
								<span v-if="userInfo.qq">{{userInfo.qq}}</span>
								<span v-else>未设置</span>
							</li>
							<li class="list-group-item">
								<label>生日</label>
								<span v-if="userInfo.birthday">{{userInfo.birthday}}</span>
								<span v-else>未设置</span>
							</li>
							<li class="list-group-item">
								<label>部门</label>
								<span v-if="userInfo.orgName">{{userInfo.orgName}}</span>
								<span v-else>未知</span>
							</li>
							<li class="list-group-item">
								<label>所在地</label>
								<span v-if="userInfo.areaName">{{userInfo.areaName}}</span>
								<span v-else>未设置</span>
							</li>
						</ul>
					</div>
				</el-card>
			</el-col>
			<el-col :span="17">
				<el-card class="card-primary">
					<div slot="header">
						<span>企业信息</span>
					</div>
					<div v-if="!partnerInfo" class="partner-info">
						<h4>尚未注册企业！</h4>
					</div>
					<div v-else-if="partnerInfo.isVerified === 0" class="partner-info">
						<h4>注册企业信息已提交，请等待平台审核...</h4>
					</div>
					<div v-else-if="partnerInfo.isVerified === 2" class="partner-info">
						<h4>注册企业信息未通过审核！</h4>
					</div>
					<div v-else-if="partnerInfo.isVerified === 1" class="partner-info">
						<h3>{{partnerInfo.name}}</h3>
						<img v-if="partnerLogo" :src="partnerLogo" :title="partnerInfo.name">
						<p>{{partnerInfo.note}}</p>
					</div>
				</el-card>
			</el-col>
		</el-row>
		<!-- 头像上传 -->
		<el-dialog :visible.sync="avatarVisible" title="头像上传">
			<el-upload
				v-loading="uploading"
			  class="uploader avatar-uploader"
			  accept="image/jpeg, image/png"
			  name='uploadFile'
			  :action="uploadAction"
			  :data="{category: 'avatar'}"
			  :show-file-list="false"
			  :on-progress="handleProgress"
			  :on-success="handleAvatarSuccess"
			  :on-error="handleError"
			  :before-upload="beforeAvatarUpload">
			  <img v-if="avatarUrl" :src="avatarUrl">
			  <i v-else class="el-icon-plus uploader-icon"></i>
			</el-upload>
			<div slot="footer">
				<el-button @click="avatarVisible = false">取消</el-button>
				<el-button type="primary" @click="uploadSubmit">确定</el-button>
			</div>
		</el-dialog>
		<!-- 账户信息编辑 -->
		<el-dialog :visible.sync="userFormVisible" title="账户信息编辑">
			<el-row>
				<el-col :span="18" :offset="3">
					<el-form :model="userForm" ref="userForm" :rules="rules" label-width="120px">
						<el-form-item label="用户名：" prop="name">
							<el-input v-model="userForm.name" placeholder="输入用户名"></el-input>
						</el-form-item>
						<el-form-item label="真实姓名：" prop="realname">
							<el-input v-model="userForm.realname" placeholder="输入真实姓名"></el-input>
						</el-form-item>
						<el-form-item label="性别：" prop="sexual">
							<el-radio class="radio" v-model="userForm.sexual" :label="1">男</el-radio>
  						<el-radio class="radio" v-model="userForm.sexual" :label="0">女</el-radio>
						</el-form-item>
						<el-form-item label="QQ：" prop="qq">
							<el-input v-model="userForm.qq" placeholder="输入QQ"></el-input>
						</el-form-item>
						<el-form-item label="生日：">
							<el-date-picker
					      v-model="userForm.birthday"
					      type="date"
					      placeholder="选择日期"
					      :default-value="new Date('2000-01-01')"
					      :picker-options="pickerOptions"
					      style="width: 100%"
					      @change="dateChange">
					    </el-date-picker>
						</el-form-item>
						<el-form-item label="所在地：">
							<el-row :gutter="5">
								<el-col :span="7">
									<el-select v-model="region.province" placeholder="选择省" @change="provinceChange">
								    <el-option
								      v-for="item in regionList.province"
								      :key="item.id"
								      :label="item.name"
								      :value="item.id">
								    </el-option>
								  </el-select>
								</el-col>
								<el-col :span="7">
									<el-select v-model="region.city" placeholder="选择市" @change="cityChange">
								    <el-option
								      v-for="item in regionList.city"
								      :key="item.id"
								      :label="item.name"
								      :value="item.id">
								    </el-option>
								  </el-select>
								</el-col>
								<el-col :span="7">
									<el-select v-model="region.area" placeholder="选择区/县" @change="areaChange">
								    <el-option
								      v-for="item in regionList.area"
								      :key="item.id"
								      :label="item.name"
								      :value="item.id">
								    </el-option>
								  </el-select>
								</el-col>
							</el-row>
						</el-form-item>
						<el-form-item label="身份证号：" prop="idcardNum">
							<el-input v-model="userForm.idcardNum"></el-input>
						</el-form-item>
						<el-form-item label="身份证正面：" prop="idcard">
							<el-upload
							  class="uploader idcard-uploader"
							  accept="image/jpeg, image/png"
							  name='uploadFile'
							  :action="uploadAction"
							  :data="{category: 'idcard'}"
							  :show-file-list="false"
							  :on-success="handleIdCardFrontSuccess"
							  :before-upload="beforeAvatarUpload">
							  <img v-if="userForm.idcardFrontUrl" :src="userForm.idcardFrontUrl">
							  <i v-else class="el-icon-plus uploader-icon"></i>
							</el-upload>
						</el-form-item>
						<el-form-item label="身份证背面：" prop="idcard">
							<el-upload
							  class="uploader idcard-uploader"
							  name='uploadFile'
							  :action="uploadAction"
							  :data="{category: 'idcard'}"
							  accept="image/jpeg, image/png"
							  :show-file-list="false"
							  :on-success="handleIdCardBackSuccess"
							  :before-upload="beforeAvatarUpload">
							  <img v-if="userForm.idcardBackUrl" :src="userForm.idcardBackUrl">
							  <i v-else class="el-icon-plus uploader-icon"></i>
							</el-upload>
						</el-form-item>
					</el-form>
				</el-col> 
			</el-row>
			<div slot="footer">
				<el-button @click="userFormVisible = false">取消</el-button>
				<el-button type="primary" @click="submitForm">确定</el-button>
			</div>
		</el-dialog>
	</section>
</template>
<script>
	import Region from '@/assets/js/region'
	import { mapGetters } from 'vuex'
	import { getMyInfo, updateMyInfo, getMyPartner, updateAvatar } from '@/api'
	export default {
		data() {
			return {
				userForm: {
    			name: '',
					realname: '',
					qq: '',
					sexual: '',
					areaId: '',
					birthday: '',
					idcardNum: '',
					idcardFrontUrl: '',
					idcardBackUrl: '',
				},
				partnerInfo: null,
				region: {
					province: '',
					city: '',
					area: ''
				},
				regionList: {
					province: [],
					city: [],
					area: [],
				},
				provinceId: '',
				cityId: '',
				// uploadAction: '/uploadFileUrl',
				uploadAction: '/ums/baseInter/uploadFile.do',
				avatarUrl: '',
				idcardFrontUrl: '',
				idcardBackUrl: '',
				loading: false,
				uploading: false,
				avatarVisible: false,
				userFormVisible: false,
				partnerLogo: '',
				areaFormat: false,
				rules: {
					name: [
						{ required: true, message: '请输入用户名', trigger: 'blur' },
					],
					realname: [
						{ required: true, message: '请输入真实姓名', trigger: 'blur' },
					],
					birthday: [
						{ type: 'date', message: '请选择出生日期', trigger: 'change' },
					],
					idcardNum: [
						{ min: 18, max: 18, message: '请输入正确身份证号', trigger: 'blur' },
					],
				},
			}
		},
		methods: {
			// 获取用户信息
			getUserInfo() {
				this.loading = true;
				getMyInfo().then(res => {
					this.loading = false;
					if(res.data.code === '0001') {
						let userInfo = res.data.result.userInfo;
						let fileInfos = res.data.result.fileInfos;
						if(JSON.stringify(fileInfos) !== '{}') {
							userInfo.avatarUrl = userInfo.avatar && fileInfos[userInfo.avatar].fileUri;
							userInfo.idcardFrontUrl = userInfo.idcardPicFront && fileInfos[userInfo.idcardPicFront].fileUri;
							userInfo.idcardBackUrl = userInfo.idcardPicBack && fileInfos[userInfo.idcardPicBack].fileUri;
						}
						this.$store.dispatch('saveUserInfo', userInfo)
					} else {
						this.$message.error(res.data.message)
					}
				}).catch(err => {
					console.log(err)
					this.loading = false;
					this.$catchError(err)
				})
			},
			// 图片上传校验
			beforeAvatarUpload(file) {
        const isJPG = file.type === 'image/jpeg' || file.type === 'image/png';
        const isLt2M = file.size / 1024 / 1024 < 2;
        if (!isJPG) {
          this.$message.error('上传图片只能是 JPG 或 PNG 格式!');
        }
        if (!isLt2M) {
          this.$message.error('上传图片大小不能超过 2MB!');
        }
        return isJPG && isLt2M;
      },
      // 上传中
      handleProgress() {
      	this.uploading = true;
      },
      // 上传失败
      handleError(err) {
      	console.log(err)
      	this.uploading = false;
      	this.$message.error('上传失败，请稍后重试')
      },
			// 头像上传成功
			handleAvatarSuccess(res, file) {
				console.log(res)
				this.uploading = false;
				if(res.code === '0001') {
					this.$message.success('上传成功')
					this.avatarUrl = URL.createObjectURL(file.raw);
					this.userInfo.avatar = res.result.fileInfo.fileUuid;
				} else {
					this.$message.error(res.message)
				}
      },
      // 身份证正面上传成功
			handleIdCardFrontSuccess(res, file) {
        if(res.code === '0001') {
					this.$message.success('上传成功')
					this.idcardFrontUrl = URL.createObjectURL(file.raw);
					this.userForm.idcardPicFront = res.result.fileInfo.fileUuid;
				} else {
					this.$message.error(res.message)
				}
      },
      // 身份证背面上传成功
			handleIdCardBackSuccess(res, file) {
        if(res.code === '0001') {
					this.$message.success('上传成功')
					this.idcardBackUrl = URL.createObjectURL(file.raw);
					this.userForm.idcardPicBack = res.result.fileInfo.fileUuid;
				} else {
					this.$message.error(res.message)
				}
      },
      dateChange(val) {
      	this.userForm.birthday = val
      },
      pickerOptions: {
      	disabledDate(time) {
      		return time.getTime() < Date.now()
      	}
      },
      uploadAvatar() {
      	this.avatarVisible = true
      },
      // 头像提交
      uploadSubmit() {
      	let data = {
      		avatar: this.userInfo.avatar
      	}
      	updateAvatar(data).then(res => {
      		if(res.data.code === '0001') {
      			this.userInfo.avatarUrl = this.avatarUrl;
      			this.$store.dispatch('saveUserInfo', this.userInfo)
      			this.$message.success('更新成功')
      		} else {
      			this.$message.error(res.data.message)
      		}
      	}).catch(err => {
      		console.log(err)
      		this.$catchError(err)
      	})
      	this.avatarVisible = false
      },
      // 账户信息编辑
      handleEdit() {
      	// this.userForm = JSON.parse(JSON.stringify(this.userInfo));
      	this.userForm = {
      		...this.userInfo
     			// userId: this.userInfo.userId,
    			// name: this.userInfo.name,
					// realname: this.userInfo.realname,
					// qq: this.userInfo.qq,
					// sexual: this.userInfo.sexual,
					// areaId: this.userInfo.areaId,
					// birthday: this.userInfo.birthday,
					// idcardNum: this.userInfo.idcardNum,
					// idcardFrontUrl: this.userInfo.idcardFrontUrl,
					// idcardBackUrl: this.userInfo.idcardBackUrl,
      	}
      	// console.log(this.userForm)
      	// this.idcardFrontUrl = this.userForm.idcardFrontUrl;
      	// this.idcardBackUrl = this.userForm.idcardBackUrl;
      	if(!this.userForm.birthday) {
      		this.userForm.birthday = new Date()
      	}
      	if(this.userForm.areaId && !this.areaFormat) {
      		this.formatRegion(this.userForm.areaId)
					this.areaFormat = true;
      	}
      	this.userFormVisible = true
      },
      // 账户信息提交
      submitForm() {
      	this.$refs.userForm.validate(valid => {
      		if(!valid) {
      			this.$notify.error({
      				title: '提示',
      				message: '请将个人信息填写完整'
      			})
      			return;
      		}
      		let data = Object.assign({}, this.userForm);
      		// console.log(data)
      		updateMyInfo(data).then(res => {
      			if(res.data.code === '0001') {
      				this.$message.success(res.data.message)
      				this.getUserInfo()
      			} else {
      				this.$message.error(res.data.message)
      			}
      		}).catch(err => {
      			console.log(err)
      			this.$catchError(err)
      		})
      		this.userFormVisible = false
      	})
      },
      // 企业信息
      getPartInfo() {
      	getMyPartner().then(res => {
      		if(res.data.code === '0001') {
      			this.partnerInfo = res.data.result.partnerInfo;
      			let fileInfos = res.data.result.fileInfos;
      			this.partnerLogo = fileInfos[this.partnerInfo.logo].fileUri;
      		} else if(res.data.code === 'U0000') {
      			this.partnerInfo = null;
      		} else {
      			this.$message.error(res.data.message)
      		}
      	}).catch(err => {
      		console.log(err)
      		this.$catchError(err)
      	})
      },
      // 选择省、市、区
      provinceChange (pid) {
      	if(this.provinceId === pid) return;
      	this.regionList.city = Region.filter(region => region.pid === pid);
      	this.region.city = this.regionList.city[0].id;
      },
      cityChange (cid) {
      	if(this.cityId === cid) return;
      	this.regionList.area = Region.filter(region => region.pid === cid);
      	this.region.area = this.regionList.area[0].id;
      },
      areaChange(aid) {
      	this.userForm.areaId = aid;
      },
      // 所在地回显
			formatRegion(areaId) {
				let origin = Region.filter(region => region.id === areaId)[0];
				if(!origin) return;
				if (origin.level === 3) {
					let city = Region.filter(region => region.id === origin.pid)[0];
					let province = Region.filter(region => region.id === city.pid)[0];
					this.regionList.city = Region.filter(region => region.pid === province.id);
					this.regionList.area = Region.filter(region => region.pid === city.id);
					this.region.province = province.id;
					this.region.city = city.id;
					this.region.area = origin.id;
				}
			},
		},
		computed: {
		  ...mapGetters([
	  		'userInfo'
	  	]),
	  },
		mounted() {
			this.regionList.province = Region.filter(region => region.level === 1)
			this.getPartInfo()
		}
	}
</script>
<style scoped lang="scss">
	.account-info {
		h3 {
			margin: 10px 0;
			color: #337ab7;
			text-align: center;
		}
		.avatar {
			width: 200px;
			height: 200px;
			margin: auto;
			img {
				display: inline-block;
				width: 100%;
				height: 100%;
				border: 1px solid #ccc;
				border-radius: 50%;
				cursor: pointer
			}
		}
	}
	.el-popover {
		text-align: center
	}
	.el-card {
		> div {
			color: #fff;
	    background-color: #337ab7;
	    border-color: #337ab7;
		}
	}
	.list-group-item {
		&.active {
			a {
				color: #fff;
				i {
					margin-left: 10px;
				}
			}
		}
	}
	.partner-info {
		text-align: center;
		h3, h4, p {
			margin: 20px 0;
		}
		h3 {
			font-weight: bold;
		}
		h4 {
			font-size: 18px;
		}
		img {
			width: 60%
		}
	}
</style>