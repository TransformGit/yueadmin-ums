<template>
	<section class="container">
		<div v-title :data-title="userList[0].partnerName"></div>
    <!-- 账户列表 -->
    <h3 class="title">{{userList[0].partnerName}}</h3>
    <el-table 
      border 
      :data="userList" 
      v-loading="loading" 
      highlight-current-row
      style="width: 100%">
      <el-table-column type="index" width="55"></el-table-column>
      <el-table-column prop="userId" label="用户ID" sortable></el-table-column>
      <el-table-column prop="name" label="用户名"></el-table-column>
      <el-table-column prop="email" label="邮箱" width="180"></el-table-column>
      <!-- <el-table-column prop="partnerName" label="企业名称"></el-table-column> -->
      <el-table-column prop="titleName" label="职位"></el-table-column>
      <el-table-column label="注册时间" sortable :formatter="formatTime"></el-table-column>
      <el-table-column label="详情">
        <template scope="scope">
          <el-button size="small" type="primary" @click="handleShow(scope.row)">查看</el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 分页 -->
    <el-row class="toolbar">
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="pageNo"
        :page-size="pageSize"
        :page-sizes="[10, 20, 30, 40]"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total">
      </el-pagination>
    </el-row>
    <!-- 账户信息 -->
		<el-dialog :visible.sync="userInfoVisible" title="账户信息">
			<el-form :model="userInfo" label-width="120px" v-loading="userLoading">
				<el-row>
					<el-col :span="12" :offset="5">
						<el-form-item label="">
							<img v-if="userInfo.avatarUrl" :src="userInfo.avatarUrl" alt="头像" class="avatar">
							<img v-else src="../../assets/img/avatar.gif" alt="头像" class="avatar">
						</el-form-item>
					</el-col>
				</el-row>
				<el-row>
					<el-col :span="12">
						<el-form-item label="姓名：">
							<span>{{userInfo.realname}}</span>
						</el-form-item>
					</el-col>
					<el-col :span="12">
						<el-form-item label="性别：">
							<span>{{userInfo.sexual === 0 ? '女' : '男' }}</span>
						</el-form-item>
					</el-col>
				</el-row>
				<el-row>
					<el-col :span="12">
						<el-form-item label="邮箱：">
							<span>{{userInfo.email | email}}</span>
						</el-form-item>
					</el-col>
					<el-col :span="12">
						<el-form-item label="手机号：">
							<span>{{userInfo.mobile | mobile}}</span>
						</el-form-item>
					</el-col>
				</el-row>
				<el-row>
					<el-col :span="12">
						<el-form-item label="QQ：">
							<span>{{userInfo.qq || '暂无'}}</span>
						</el-form-item>
					</el-col>
					<el-col :span="12">
						<el-form-item label="生日：">
							<span>{{userInfo.birthday || '暂无'}}</span>
						</el-form-item>
					</el-col>
				</el-row>
				<el-row>
					<el-col :span="12">
						<el-form-item label="部门：">
							<span>{{userInfo.orgName || '暂无'}}</span>
						</el-form-item>
					</el-col>
					<el-col :span="12">
						<el-form-item label="职位：">
							<span>{{userInfo.titleName || '暂无'}}</span>
						</el-form-item>
					</el-col>
				</el-row>
				<el-row>
					<el-col :span="12">
						<el-form-item label="所在地：">
							<span>{{userInfo.areaName}}</span>
						</el-form-item>
					</el-col>
					<el-col :span="12">
						<el-form-item label="注册时间：">
							<span>{{userInfo.createTime | formatDate}}</span>
						</el-form-item>
					</el-col>
				</el-row>
				<el-row>
					<el-col :span="12">
						<el-form-item label="身份证正面：">
							<img :src="userInfo.idcardFrontUrl" alt="暂无身份证正面" class="idcard-pic">
						</el-form-item>
					</el-col>
				</el-row>
				<el-row>
					<el-col :span="12">
						<el-form-item label="身份证背面：">
							<img :src="userInfo.idcardBackUrl" alt="暂无身份证背面" class="idcard-pic">
						</el-form-item>
					</el-col>
				</el-row>
			</el-form>
			<div slot="footer">
				<el-button @click="userInfoVisible = false">取消</el-button>
				<el-button type="primary" @click="userInfoVisible = false">确定</el-button>
			</div>
		</el-dialog>
	</section>
</template>
<script>
	import { getPartnerUsers, getPartnerUserInfoById } from '@/api'
	export default {
		data() {
			return {
				partnerId: '',
				loading: false,
				pageNo: 1,
				pageSize: 10,
				total: 0,
				userList: [],
				userInfo: {},
				userInfoVisible: false,
				userLoading: false,
			}
		},
		methods: {
			formatTime(row) {
				return this.$moment(row.createTime).format('YYYY-MM-DD')
			},
			handleSizeChange(val) {
				this.pageSize = val;
				this.getUserList()
			},
			handleCurrentChange(val) {
				this.pageNo = val;
				this.getUserList()
			},
			handleBack() {
				this.$router.back()
			},
			getUserList() {
				let params = {
					pageNo: this.pageNo,
					pageSize: this.pageSize,
					partnerId: this.partnerId,
				}
				this.loading = true;
				getPartnerUsers(params).then(res => {
					this.loading = false
					if(res.data.code === '0001') {
						this.userList = res.data.result.userList;
						this.total = res.data.result.pageInfo.total;
					} else {
						this.$message.error(res.data.message)
					}
				}).catch(err => {
					console.log(err)
					this.loading = false;
					this.$catchError(err)
				})
			},
			// 账户信息详情
      handleShow(row) {
      	this.userInfo = {};
      	this.userInfoVisible = true;
      	let params = {
					userId: row.userId
				}
				this.userLoading = true;
				getPartnerUserInfoById(params).then(res => {
					this.userLoading = false;
					let userInfo = res.data.result.userInfo;
					let fileInfos = res.data.result.fileInfos;
					if(JSON.stringify(fileInfos) !== '{}') {
						userInfo.avatarUrl = userInfo.avatar && fileInfos[userInfo.avatar].fileUri;
						userInfo.idcardFrontUrl = userInfo.idcardPicFront && fileInfos[userInfo.idcardPicFront].fileUri;
						userInfo.idcardBackUrl = userInfo.idcardPicBack && fileInfos[userInfo.idcardPicBack].fileUri;
					}
					this.userInfo = userInfo;
				}).catch(err => {
					console.log(err)
					this.userLoading = false;
					this.$catchError(err)
				})
      },
		},
		mounted() {
			this.partnerId = this.$route.query.partnerId;
			this.partnerId && this.getUserList()
		}
	}
</script>
<style scoped lang="scss">
	.title {
		margin: 15px 0;
		padding: 15px;
		text-align: center;
		background: #f0f0f0;
	}
	.avatar {
		width: 160px;
		height: 160px;
		border-radius: 50%;
		border: 1px solid #ddd;
	}
</style>