<!--
    安环anhuan，首页
        大型机械设备详情页equipmentDetail（参考与：工点进度详情process/siteDetail.vue）
 -->

<template>
	<div class="siteDetail">
		<div class="nav">
			<span @click="() => $router.go(-1)"
				>返回<i class="el-icon-arrow-right"></i
			></span>
			<span>安全环保<i class="el-icon-arrow-right"></i></span>
			<span>大型机械设备</span>
		</div>

		<div class="content">
			<div class="time">
				<div>
					<p><i></i>设备统计</p>
				</div>
				<div>
					<span>项目：</span>
					<span class="el-select">
						<el-select
							size="mini"
							v-model="id"
							@change="selectValue"
							:popper-append-to-body="false"
						>
							<el-option
								v-for="item in options"
								:key="item.name"
								:label="item.name"
								:value="item.id"
							></el-option>
						</el-select>
					</span>
					<!-- <span class="spanLeft">(最新统计时间：{{ time }})</span> -->
				</div>
			</div>

			<!-- 图表：barChart柱状图 -->
			<div class="barChart">
				<!-- 统计数量 -->
				<div style="display: flex" class="sumlist_container">
					<div v-for="item in sumlist" :key="item.sum" class="sumlist">
						<div class="item_name">{{ item.name }} :</div>
						<div class="item_sum">{{ item.sum }}</div>
					</div>
				</div>
				<!-- <echarts-bar
					:dimensionsList="dimensionsList1"
					:barList="barList1"
					:serieslist="serieslist1"
				></echarts-bar> -->
				<echarts-bar
					:dimensionsList="dimensionsList1"
					:barList="barList1"
				></echarts-bar>
			</div>

			<!-- 列表：table表格 -->
			<div class="table">
				<el-button
					size="mini"
					type="primary"
					class="daochu"
					@click="exportTable"
					>导出</el-button
				>
				<el-table
					size="medium"
					style="width: 100%"
					border
					:data="tableData.slice((current - 1) * size, current * size)"
					class="dark-table"
					stripe
					:span-method="objectSpanMethod"
				>
					<!-- type="index" -->
					<el-table-column
						header-align="center"
						align="center"
						type="index"
						label="序号"
						width="50"
						:index="indexChange"
					>
						<!-- 	<template slot-scope="scope">
							<span>{{ scope.$index }}</span>
						</template> -->
					</el-table-column>

					<el-table-column
						header-align="center"
						align="center"
						prop="projectName"
						label="项目名称"
					>
						<template slot-scope="scope">
							<!-- 	<span v-if="scope.$index != 0">
								{{ scope.row.projectName }}
							</span>
							<span v-else>总计</span> -->
							<span v-if="current == 1 && scope.$index == 0"> 总计 </span>
							<span v-else>
								{{ scope.row.projectName }}
							</span>
						</template>
					</el-table-column>

					<el-table-column
						header-align="center"
						align="center"
						prop="sectionName"
						label="标段名称"
					>
						<template slot-scope="scope">
							<span class="spanName" @click="detailsDialog(scope.row)">
								{{ scope.row.sectionName }}
							</span>
						</template>
					</el-table-column>

					<el-table-column
						header-align="center"
						align="center"
						prop="sum"
						label="设备总数"
					></el-table-column>

					<el-table-column
						header-align="center"
						align="center"
						prop="sumDa"
						label="大型起重吊装及制运架设备"
					></el-table-column>

					<el-table-column
						header-align="center"
						align="center"
						prop="sumGui"
						label="轨道运行设备"
					></el-table-column>

					<el-table-column
						header-align="center"
						align="center"
						prop="sumQi"
						label="其它大型设备"
					></el-table-column>
				</el-table>

				<el-pagination
					@size-change="handleSizeChange"
					@current-change="handleCurrentChange"
					:current-page.sync="current"
					:page-size="size"
					layout="total, prev, pager, next, jumper"
					:total="totalCount"
				></el-pagination>
			</div>

			<!-- 弹窗：设备详情 -->
			<el-dialog class="touzixiangqing" :visible.sync="visible" width="90%">
				<div class="dialog">
					<p class="title"><i></i>标段设备详情</p>
					<div class="select">
						<span>项目：</span>
						<span class="el-select">
							<el-select
								size="mini"
								v-model="projectId"
								@change="selectProject"
							>
								<el-option
									v-for="item in ProjectArray"
									:key="item.projectId"
									:label="item.projectName"
									:value="item.projectId"
								></el-option>
							</el-select>
						</span>

						<span>标段：</span>
						<span class="el-select">
							<el-select size="mini" v-model="sectionId" clearable>
								<el-option
									v-for="item in sectionArray"
									:key="item.sectionId"
									:label="item.sectionName"
									:value="item.sectionId"
								></el-option>
							</el-select>
						</span>

						<span>设备类型：</span>
						<span class="el-select">
							<el-cascader
								clearable
								v-model="value"
								:options="typeArray"
								size="mini"
								:show-all-levels="false"
							></el-cascader>
						</span>

						<span>设备状态：</span>
						<span class="el-select">
							<el-select size="mini" v-model="status" clearable>
								<el-option
									v-for="item in statusArray"
									:key="item.status"
									:label="item.statusName"
									:value="item.status"
								></el-option>
							</el-select>
						</span>

						<span>设备权属：</span>
						<span class="el-select">
							<el-select size="mini" v-model="quanshu" clearable>
								<el-option
									v-for="item in quanshuArray"
									:key="item.quanshu"
									:label="item.quanshuName"
									:value="item.quanshu"
								></el-option>
							</el-select>
						</span>

						<el-button type="primary" size="mini" @click="changeDialogData"
							>查询</el-button
						>
						<el-button type="primary" size="mini" @click="exportDialogData"
							>导出</el-button
						>
					</div>
					<div class="dialogTable">
						<el-table
							size="mini"
							style="width: 100%"
							border
							:data="dialogData"
							class="dark-table"
							stripe
						>
							<el-table-column
								header-align="center"
								align="center"
								type="index"
								label="序号"
								width="50"
								:index="indexDialog"
							>
							</el-table-column>

							<el-table-column
								header-align="center"
								align="center"
								prop="bianhao"
								label="设备编号"
								show-overflow-tooltip
							></el-table-column>
							<el-table-column
								header-align="center"
								align="center"
								prop="name"
								label="设备名称"
								show-overflow-tooltip
							></el-table-column>

							<el-table-column
								header-align="center"
								align="center"
								prop="typename"
								label="设备类型"
								show-overflow-tooltip
							></el-table-column>

							<el-table-column
								header-align="center"
								align="center"
								prop="danwei"
								label="单位"
								width="80"
								show-overflow-tooltip
							></el-table-column>

							<el-table-column
								header-align="center"
								align="center"
								prop="xinghao"
								label="规格型号"
								show-overflow-tooltip
							></el-table-column>

							<el-table-column
								header-align="center"
								align="center"
								prop="zhizaochang"
								label="制造厂"
								show-overflow-tooltip
							></el-table-column>

							<el-table-column
								header-align="center"
								align="center"
								prop="brithday"
								label="出厂年月"
							></el-table-column>

							<el-table-column
								header-align="center"
								align="center"
								prop="dateofuse"
								label="使用年月"
							></el-table-column>

							<el-table-column
								header-align="center"
								align="center"
								prop="shigongdanwei"
								label="施工单位"
								show-overflow-tooltip
							></el-table-column>

							<el-table-column
								header-align="center"
								align="center"
								prop="siteName"
								label="所在工点"
								show-overflow-tooltip
							></el-table-column>

							<el-table-column
								header-align="center"
								align="center"
								prop="user"
								label="操作人员"
								width="80"
								show-overflow-tooltip
							>
								<template slot-scope="scope">
									<p v-if="scope.row.user.indexOf('证书') != -1">
										{{ scope.row.user.split("证书")[0] }}
									</p>
									<p v-else>
										{{ scope.row.user }}
									</p>
								</template>
							</el-table-column>

							<el-table-column
								header-align="center"
								align="center"
								prop="status"
								label="设备状态"
								width="80"
							>
								<template slot-scope="scope">
									<p v-if="scope.row.status == 1">好</p>
									<p v-if="scope.row.status == 2">一般</p>
									<p v-if="scope.row.status == 3">差</p>
								</template>
							</el-table-column>

							<el-table-column
								header-align="center"
								align="center"
								prop="sbquanshu"
								label="设备权属"
								width="80"
							>
								<template slot-scope="scope">
									<p v-if="scope.row.sbquanshu == 1">自有</p>
									<p v-if="scope.row.sbquanshu == 2">租赁</p>
									<p v-if="scope.row.sbquanshu == 3">其它</p>
								</template>
							</el-table-column>

							<el-table-column
								header-align="center"
								align="center"
								prop="wenjianname"
								label="操作"
								width="80"
								show-overflow-tooltip
							>
								<template slot-scope="scope">
									<span
										class="spanName"
										style="cursor: pointer; color: #0bf6ff"
										@click="detailsDialog2(scope.row)"
									>
										详情
									</span>
								</template>
							</el-table-column>
						</el-table>

						<el-pagination
							:current-page="page"
							:page-size="limit"
							:total="total"
							layout="total, prev, pager, next, jumper"
							@size-change="pageSizeChangeHandle"
							@current-change="pageCurrentChangeHandle"
							class="el-pagination"
							:background="isBackground"
						>
						</el-pagination>
					</div>
				</div>
			</el-dialog>
		</div>

		<!-- 弹窗-2 ======================================== -->
		<el-dialog class="touzixiangqing" :visible.sync="visible2" width="70%">
			<div class="dialog">
				<p class="title"><i></i>设备详情</p>
				<div class="dialogTable">
					<!-- 设备编号、设备名称 -->
					<el-row style="border-top: 1px solid #0062be; margin-top: 20px">
						<el-col :span="4"><div class="grid-content">设备编号</div></el-col>
						<el-col :span="8">
							<div class="grid-content">
								{{ dialogData2.bianhao }}
							</div>
						</el-col>
						<el-col :span="4"><div class="grid-content">设备名称</div></el-col>
						<el-col :span="8">
							<div class="grid-content">
								{{ dialogData2.name }}
							</div>
						</el-col>
					</el-row>
					<!-- 单位、规格型号 -->
					<el-row>
						<el-col :span="4">
							<div class="grid-content">单位</div>
						</el-col>
						<el-col :span="8">
							<div class="grid-content">
								{{ dialogData2.danwei }}
							</div>
						</el-col>
						<el-col :span="4">
							<div class="grid-content">规格型号</div>
						</el-col>
						<el-col :span="8">
							<div class="grid-content">
								{{ dialogData2.xinghao }}
							</div>
						</el-col>
					</el-row>
					<!-- 制造厂、出厂年月 -->
					<el-row>
						<el-col :span="4">
							<div class="grid-content">制造厂</div>
						</el-col>
						<el-col :span="8">
							<div class="grid-content">
								{{ dialogData2.zhizaochang }}
							</div>
						</el-col>
						<el-col :span="4">
							<div class="grid-content">出厂年月</div>
						</el-col>
						<el-col :span="8">
							<div class="grid-content">
								{{ dialogData2.brithday }}
							</div>
						</el-col>
					</el-row>
					<!-- 使用年月、施工单位 -->
					<el-row>
						<el-col :span="4">
							<div class="grid-content">使用年月</div>
						</el-col>
						<el-col :span="8">
							<div class="grid-content">
								{{ dialogData2.dateofuse }}
							</div>
						</el-col>
						<el-col :span="4">
							<div class="grid-content">施工单位</div>
						</el-col>
						<el-col :span="8">
							<div class="grid-content">
								{{ dialogData2.shigongdanwei }}
							</div>
						</el-col>
					</el-row>
					<!-- 所在工点、操作人员 -->
					<el-row>
						<el-col :span="4">
							<div class="grid-content">所在工点</div>
						</el-col>
						<el-col :span="8">
							<div class="grid-content">
								{{ dialogData2.siteName }}
							</div>
						</el-col>
						<el-col :span="4">
							<div class="grid-content">操作人员</div>
						</el-col>
						<el-col :span="8">
							<div class="grid-content">
								{{ dialogData2.user }}
							</div>
						</el-col>
					</el-row>
					<!-- 设备状态、设备权属 -->
					<el-row>
						<el-col :span="4">
							<div class="grid-content">设备状态</div>
						</el-col>
						<el-col :span="8">
							<div class="grid-content" v-if="dialogData2.status == 1">好</div>
							<div class="grid-content" v-if="dialogData2.status == 2">
								一般
							</div>
							<div class="grid-content" v-if="dialogData2.status == 3">差</div>
						</el-col>
						<el-col :span="4">
							<div class="grid-content">设备权属</div>
						</el-col>
						<el-col :span="8">
							<div class="grid-content" v-if="dialogData2.sbquanshu == 1">
								自有
							</div>
							<div class="grid-content" v-if="dialogData2.sbquanshu == 2">
								租赁
							</div>
							<div class="grid-content" v-if="dialogData2.sbquanshu == 3">
								其它
							</div>
						</el-col>
					</el-row>
					<!-- 附件名称 -->
					<el-row>
						<el-col :span="4">
							<div class="grid-content">附件名称</div>
						</el-col>
						<el-col :span="20">
							<div class="grid-content">
								{{ dialogData2.wenjianname }}&nbsp;&nbsp;
								<!-- <span
									v-if="dialogData2.wenjianname"
									style="cursor: pointer; color: #0bf6ff"
									@click="() => (dialogpic = true)"
									>预览
								</span> -->
								<span
									v-if="dialogData2.wenjianname"
									style="cursor: pointer; color: #0bf6ff"
									@click="previewPDF(dialogData2.wenjianurl)"
									>预览
								</span>
							</div>
						</el-col>
					</el-row>
				</div>
			</div>
		</el-dialog>

		<el-dialog
			width="70%"
			top="5%"
			:visible.sync="dialogpic"
			class="touzixiangqing"
			style="padding: 0px"
		>
			<div>
				<img
					:src="dialogData2.wenjianurl"
					style="width: 100%; padding-top: 25px"
				/>
			</div>
		</el-dialog>
	</div>
</template>

<script>
import { getgdDetails, getSiteInfo } from "@/api/processAPI"
import {
	getJXSum,
	getJXDetail,
	getAllSec,
	downloadJXtongji,
	getJXType,
	tanChuang,
	downloadJXInfo,
	tanChuang2,
} from "@/api/anhuan.js"
import EchartsBar from "@/views/anhuan/components/equipmentBar.vue"

export default {
	name: "",
	components: { EchartsBar },
	data() {
		return {
			time: "",
			tableData: [],
			// 四个项目
			options: [],
			id: "",
			currentProjectName: "", // 当前选择的项目名称

			proIds: [], // 项目id的数组 [1078,1232,1275,1234]
			//统计数量
			sumlist: [],

			dimensionsList1: [
				"product",
				"其它大型设备",
				"轨道运行设备",
				"大型起重吊装及制运架设备",
			],

			barList1: [],
			serieslist1: [
				{
					type: "bar",
					stack: "total",
					barWidth: 40,
					itemStyle: { normal: { borderRadius: [2, 2, 0, 0] } },
				},
				{
					type: "bar",
					stack: "total",
					barWidth: 40,
					itemStyle: { normal: { borderRadius: [2, 2, 0, 0] } },
				},
				{
					type: "bar",
					stack: "total",
					barWidth: 40,
					itemStyle: { normal: { borderRadius: [2, 2, 0, 0] } },
				},
			],
			// 表格分页
			current: 1,
			size: 10,
			totalCount: 0,

			/* 弹窗 ======================================== */
			visible: false,
			dialogData: [],
			sectionArray: [], // 弹窗增加标段sectionArray的切换查询
			sectionId: "",
			ProjectArray: [], // 弹窗增加项目ProjectArray的切换查询
			projectId: "",

			// +弹窗，增加设备状态statusArray的切换查询
			statusArray: [
				// {
				// 	status: "",
				// 	statusName: "全部",
				// },
				{
					status: 1,
					statusName: "好",
				},
				{
					status: 2,
					statusName: "一般",
				},
				{
					status: 3,
					statusName: "差",
				},
			],
			status: "",
			// 弹窗，增加设备权属于quanshuArray的切换查询
			quanshuArray: [
				// {
				// 	quanshu: "",
				// 	quanshuName: "全部",
				// },
				{
					quanshu: 1,
					quanshuName: "自有",
				},
				{
					quanshu: 2,
					quanshuName: "租赁",
				},
				{
					quanshu: 3,
					quanshuName: "其它",
				},
			],
			quanshu: "",
			// 弹窗，设备类型typeArray的切换查询
			typeArray: [],
			value: "", // 选中的设备类型
			// 弹窗，全部查询条件
			formData: {
				proIds: [], // 项目id
				secIds: [], // 标段id
				type: "", // 设备类型
				status: "", // 设备状态
				sbQuanShu: "", // 设备权属
				page: 1, // 当前页
				size: 10, // 每页条数
			},
			// 弹窗，分页
			page: 1, // 当前页码
			limit: 10, // 每页数
			total: 0, // 总条数
			isBackground: true, // 是否有背景色

			/* 弹窗-2 ======================================== */
			visible2: false,
			formData2: {
				proIds: [], // 项目id
			},
			dialogData2: {},

			/* 弹窗-3 ======================================== */
			dialogpic: false,
		}
	},
	mounted() {
		this.options = JSON.parse(window.localStorage.getItem("ids"))

		// 当前选择的项目名称
		this.currentProjectName = JSON.parse(
			window.localStorage.getItem("ids")
		)[0].name
		// =========================
		let allids = this.$store.state.app.ids
		for (let i = 0; i < allids.length; i++) {
			this.proIds.push(allids[i].id)
		}

		this.options.unshift({
			id: this.proIds,
			name: "全部",
		})
		this.id = this.options[0].id
		console.log("[ this.options ]-项目", this.options)

		let sumId = this.options[0].id
		this.getJXSum(sumId) // 详情，总设备统计(图和表的总览数量) hhtlxxh/jx/getJXSum POST
		this.getJXDetail(sumId) // 详情，上面柱状图
		this.getAllSec(sumId) // 详情，下面表格

		// 弹窗增加项目ProjectArray的切换查询
		let projectIds = this.$store.state.app.ids
		// this.ProjectArray = this.options.map((item) => {  // 有全部
		this.ProjectArray = projectIds.map((item) => {
			return {
				projectId: item.id,
				projectName: item.name,
			}
		})
	},
	methods: {
		/* 大型机械设备详情页，设备统计 ======================================== */
		// 详情页，切换项目
		selectValue(id) {
			let sumId = ""
			if (id.length != 4) {
				console.log("[ id ] >", id)
				let a = []
				a.push(id)

				sumId = a
				console.log("[ this.id ]-517", this.id)
			} else {
				sumId = this.id
			}
			this.sumlist = []
			this.tableData = []
			this.barList1 = []
			this.serieslist1 = [
				{
					type: "bar",
					stack: "total",
					barWidth: 40,
					itemStyle: { normal: { borderRadius: [2, 2, 0, 0] } },
				},
				{
					type: "bar",
					stack: "total",
					barWidth: 40,
					itemStyle: { normal: { borderRadius: [2, 2, 0, 0] } },
				},
				{
					type: "bar",
					stack: "total",
					barWidth: 40,
					itemStyle: { normal: { borderRadius: [2, 2, 0, 0] } },
				},
			]
			this.getJXSum(sumId)
			this.getJXDetail(sumId)
			this.getAllSec(sumId)

			// this.queryDetailsData()
		},
		// 详情页，总设备统计(图和表的总览数量) hhtlxxh/jx/getJXSum POST
		getJXSum(sumId) {
			getJXSum({ proIds: sumId }).then((res) => {
				if (res.code == 0) {
					// console.log("[ 总设备统计-res.data ]", res.data)
					this.sumlist = [
						{
							name: "设备总数",
							sum: res.data.sum,
						},
						{
							name: "大型起重吊装及制运架设备",
							sum: res.data.sumDa,
						},
						{
							name: "轨道运行设备",
							sum: res.data.sumGui,
						},
						{
							name: "其它大型设备",
							sum: res.data.sumQi,
						},
					]
				}
			})
		},
		// 详情页，上面柱状图 hhtlxxh/jx/getJXDetail POST
		getJXDetail(sumId) {
			getJXDetail({ proIds: sumId }).then((res) => {
				// console.log("[ 上面柱状图-res.data ]", res.data)
				if (res.code == 0) {
					res.data.forEach((element) => {
						let list = []
						for (let i = 0; i < res.data.length; i++) {
							if (sumId.length == 4) {
								list[0] = element.projectName
							} else {
								list[0] = element.sectionName
							}
							// sum 6  sumDa 0 sumGui 2  sumQi 4
							list[3] = Number(element.sumDa)
							list[2] = Number(element.sumGui)
							list[1] = Number(element.sumQi)
							list[4] = Number(element.sum)
						}
						this.barList1.push(list)
					})
				}
			})
		},
		// 详情页，下面的table表格 hhtlxxh/jx/getAllSec
		getAllSec(sumId) {
			getAllSec({ proIds: sumId }).then((res) => {
				// console.log("[ 下面的table表格-res.data ]", res.data)
				if (res.code == 0) {
					let a1 = {
						sum: this.sumlist[0].sum,
						sumDa: this.sumlist[1].sum,
						sumGui: this.sumlist[2].sum,
						sumQi: this.sumlist[3].sum,
					}
					res.data.unshift(a1)
					this.tableData = res.data

					this.totalCount = res.data.length - 1
				}
			})
		},
		// 详情页，表格合并单元格，合并第一行的第一列和第二列
		// objectSpanMethod({ rowIndex, columnIndex }) {
		// 	if (rowIndex === 0) {
		// 		if (columnIndex === 1) {
		// 			return [1, 2]
		// 		} else if (columnIndex === 2) {
		// 			return [0, 0]
		// 		}
		// 	}
		// },
		objectSpanMethod({ rowIndex, columnIndex }) {
			if (this.current == 1) {
				if (rowIndex === 0) {
					if (columnIndex === 0) {
						return [0, 0]
					} else if (columnIndex === 1) {
						return [1, 3]
					} else if (columnIndex === 2) {
						return [0, 0]
					}
				}
			}
		},

		// 详情页，分页-前端分页
		handleSizeChange(val) {
			// console.log(`每页 ${val} 条`)
			this.size = val
		},
		handleCurrentChange(val) {
			// console.log(`当前页: ${val}`)
			this.current = val
		},
		indexChange(index) {
			return (this.current - 1) * 10 + index
		},
		// 详情页，导出
		async exportTable() {
			// exportTable() {
			let a = this.options[0].id
			let res = await downloadJXtongji({ proIds: a })
			this.downloadCallback(res, "设备统计.xls")

			/* 	this.$confirm("导出的是全部项目的数据", "提示", {
				confirmButtonText: "确定",
				cancelButtonText: "取消",
				type: "warning",
			})
				.then(() => {
					downloadJXtongji({ proIds: this.options[0].id }).then((res) => {
						this.downloadCallback(res, "设备统计.xls")
					})
				})
				.catch(() => {
					this.$message({
						type: "info",
						message: "已取消",
					})
				}) */
		},
		downloadCallback(res, fileName) {
			const content = res
			const blob = new Blob([content])
			if ("download" in document.createElement("a")) {
				// 非IE下载
				const elink = document.createElement("a")
				elink.download = fileName
				elink.style.display = "none"
				elink.href = URL.createObjectURL(blob)
				document.body.appendChild(elink)
				elink.click()
				URL.revokeObjectURL(elink.href) // 释放URL 对象
				document.body.removeChild(elink)
			} else {
				// IE10+下载
				navigator.msSaveBlob(blob, fileName)
			}
		},
		/* +++点击标段名称，弹窗 设备详情 ======================================== */
		detailsDialog(row) {
			// console.log("[ 点击标段名称row ] >", row, row.sectionId)
			this.visible = true
			this.projectId = row.projectId // 点击的项目id
			// 根据点击的项目ID，获取标段列表
			let nodeId = row.projectId
			getgdDetails({ nodeId }).then((res) => {
				if (res.code == 0) {
					this.sectionArray = res.data.map((item) => {
						return {
							sectionName: item.name,
							sectionId: item.id,
						}
					})
				}
			})
			this.sectionId = Number(row.sectionId) // 点击的标段id

			this.formData.proIds = [this.projectId]
			this.formData.secIds = [this.sectionId]
			this.formData.type = ""
			this.formData.status = ""
			this.formData.sbQuanShu = ""

			this.queryTypeArray() // +++弹窗，获取设备类型typeArray的数据
			this.queryDialogData() // +++弹窗，获取数据
		},
		// 弹窗，从后端获取设备类型typeArray的数据
		queryTypeArray() {
			getJXType({}).then((res) => {
				if (res.code == 0) {
					/* let a1 = {
						value: "",
						label: "全部",
					}
					res.data.unshift(a1) */
					this.typeArray = res.data
				}
			})
		},
		// 弹窗，获取数据
		queryDialogData() {
			this.dialogData = []
			// 第一页
			this.$set(this.formData, "page", 1)
			this.page = 1
			tanChuang({ ...this.formData }).then((res) => {
				// console.log("[ res ]-弹窗数据", res)
				if (res.code == 0) {
					this.dialogData = res.data.records
					this.total = res.data.total
				}
			})
		},
		// 弹窗，切换项目获取标段
		selectProject(event) {
			this.sectionArray = []
			this.sectionId = ""
			let nodeId = event
			getgdDetails({ nodeId }).then((res) => {
				if (res.code == 0) {
					this.sectionArray = res.data.map((item) => {
						return {
							sectionName: item.name,
							sectionId: item.id,
						}
					})
					// this.sectionId = Number(this.sectionArray[0].sectionId) // 默认选中第一个标段
				}
			})
		},

		//弹窗，筛选数据
		changeDialogData() {
			this.formData.proIds = [this.projectId]

			this.formData.type = this.value[1]
			this.formData.status = this.status
			this.formData.sbQuanShu = this.quanshu

			// this.formData.secIds = [this.sectionId]
			let a = Number(this.sectionId)
			this.formData.secIds = a !== 0 ? [a] : [] // 标段可不选

			this.queryDialogData()
		},
		// 弹窗，导出
		async exportDialogData() {
			let a = this.options[0].id
			let b = {
				proIds: a,
				secIds: [],
				type: "",
				status: "",
				sbQuanShu: "",
				page: "",
				size: "",
			}
			let res = await downloadJXInfo({ ...b })
			console.log("[ res ] >", res)
			this.downloadCallback2(res, "设备详情.xls")
		},
		downloadCallback2(res, fileName) {
			const content = res
			const blob = new Blob([content])
			if ("download" in document.createElement("a")) {
				// 非IE下载
				const elink = document.createElement("a")
				elink.download = fileName
				elink.style.display = "none"
				elink.href = URL.createObjectURL(blob)
				document.body.appendChild(elink)
				elink.click()
				URL.revokeObjectURL(elink.href) // 释放URL 对象
				document.body.removeChild(elink)
			} else {
				// IE10+下载
				navigator.msSaveBlob(blob, fileName)
			}
		},
		// 弹窗，分页-后端-每页条数
		pageSizeChangeHandle(val) {
			this.page = 1
			this.limit = val
		},
		// 弹窗，分页-后端-当前页
		pageCurrentChangeHandle(val) {
			console.log("[ val ]-当前页", val)
			this.page = val
			this.$set(this.formData, "page", val)
			// this.queryDialogData() // +++弹窗，获取数据
			this.dialogData = []
			tanChuang({ ...this.formData }).then((res) => {
				// console.log("[ res ]-弹窗数据", res)
				if (res.code == 0) {
					this.dialogData = res.data.records
					this.total = res.data.total
				}
			})
		},
		// 弹窗1，分页后端做的，序号前端要计算
		indexDialog(index) {
			return (this.page - 1) * 10 + index + 1
		},

		/* 弹窗2，点击弹窗1的详情============================================================= */
		detailsDialog2(row) {
			console.log("[ row ]-点击详情", row, row.id)
			this.visible2 = true
			this.formData2.proIds = [row.id]
			tanChuang2({ ...this.formData2 }).then((res) => {
				if (res.code == 0) {
					this.dialogData2 = res.data[0]

					// this.dialogData2.wenjianname = "123"
					// this.dialogData2.wenjianurl =
					// 	"https://static.runoob.com/images/demo/demo2.jpg"
				}
			})
		},
		// 弹窗2，预览
		previewPDF(wenjianurl) {
			const width = 800
			const height = 600
			const left = (window.innerWidth - width) / 2
			const top = (window.innerHeight - height) / 2
			const options = `width=${width},height=${height},left=${left},top=${top}`

			window.open(wenjianurl, "_blank", options)
		},
	},
}
</script>

<style lang="scss" scoped>
// scoped
::v-deep .el-select {
	width: 181px;
	.el-input {
		height: 28px;
		.el-input__inner {
			width: 100% !important;
			color: #cadbff;
			background-color: #004289 !important;
			border: 1px solid #0c58aa;
		}
	}

	.el-select-dropdown {
		border: 1px solid #0c58aa;
		background-color: #004289 !important;
		opacity: 0.9;
		margin-top: 10px !important;
	}
	.el-select-dropdown__item {
		// 未选中的
		color: #cadbff;
	}
	.el-select-dropdown__item.selected {
		// 选中的
		color: #4cf5ff;
	}
	.el-select-dropdown__item.hover {
		// 鼠标移上去的
		background-color: #cadbff;
		opacity: 0.5;
		// color: #004289;
		color: #0000ff;
	}
}

// 修改 级联选择器el-cascader的样式

.siteDetail {
	padding: 60px 32px 0 32px;
	background: url(@/assets/images/page-bg.jpg);
	background-size: 100% 100%;
	.nav {
		margin-left: 15px;
		display: flex;
		align-items: center;
		height: 52px;

		font-size: 14px;
		span:nth-child(1),
		span:nth-child(2) {
			color: #cadbff;
			// margin-right: 20px;
		}
		span:nth-child(3) {
			color: rgba(#cadbff, 0.6);
		}
		span:nth-child(1):hover {
			color: #0bf6ff;
			cursor: pointer;
		}
		.el-icon-arrow-right {
			margin: 0 6px;
		}
	}

	.content {
		// margin: 15px;
		margin: 0 15px;

		.time {
			display: flex;
			justify-content: space-between;
			div:nth-child(1) {
				// margin-top: -20px;
				margin-top: -25px;
				margin-bottom: -8px;
				p {
					font-size: 20px;
					color: #fff;
					font-style: normal;
					font-family: Helvetica Neue, Helvetica, Arial, PingFang SC,
						Hiragino Sans GB, Heiti SC, Microsoft YaHei, WenQuanYi Micro Hei,
						sans-serif;
					i {
						display: inline-block;
						width: 2px;
						height: 8px;
						background-color: #49edf7;
						margin-right: 10px;
						margin-bottom: 2px;
					}
				}
			}
			div:nth-child(2) {
				font-weight: 400;
				.spanLeft {
					margin-left: 15px;
				}
			}
		}

		/* 列表：table表格 */
		.table {
			// padding: 25px 0;
			padding-top: 10px;
			.daochu {
				margin-bottom: 10px;
				float: right;
			}
		}

		/* 表格可点击：列表table表格-标段名称、弹窗-附件 */
		// 点击标段名称
		.spanName {
			cursor: pointer;
			text-decoration: underline;
			// 下划线距离文字的距离
			text-underline-offset: 2px;
		}
		.spanName:hover {
			color: #0bf6ff;
			font-weight: 600;
			// 下划线变粗
			text-decoration-thickness: 2px;
		}

		/* 图表：barChart柱状图 */
		.barChart {
			width: 100%;
			// barChart柱状图
			// padding-top: 30px;
			padding-top: 20px;
			// height: 280px;
			height: 280px;

			border: 1px solid #1a8dfa;
			border-radius: 6px;
			box-shadow: inset 0px 0px 20px -1px #3ec5fc;
			background: linear-gradient(
				0deg,
				rgba(2, 18, 66, 0.2) 0%,
				rgba(9, 66, 213, 0.2) 106%
			);

			// 统计数量
			.sumlist_container {
				// position: absolute;

				margin-top: -10px;

				box-shadow: inset 0px 0px 1px -1px #3ec5fc;
				background: linear-gradient(
					to right,
					rgba(4, 46, 100, 0.4),
					rgba(1, 28, 72, 0.1)
				);
				// border: 1px solid #1a8dfa;
				width: 40%;
				margin-left: 20px;
				padding: 6px 0;

				.sumlist {
					display: flex;
					align-items: center;

					font-weight: 400;
					// opacity: 0.6;
					font-family: PingFangSC-Regular;
					font-size: 14px;
					// padding-left: 20px;

					// .item_name {
					// }
					.item_sum {
						margin-left: 5px;
						margin-right: 15px;
					}
				}
			}
		}

		// 弹窗
		.dialog {
			.select {
				display: block;
				display: flex;
				justify-content: flex-end;
				align-items: center;
				margin-bottom: 15px;
				color: #cadbff;
				span:nth-child(2),
				span:nth-child(4),
				span:nth-child(6),
				span:nth-child(8),
				span:nth-child(10) {
					margin-right: 20px;
				}
			}
		}

		::v-deep .el-pagination {
			color: #cadbff;
			margin-top: 10px;
			button:disabled {
				background-color: transparent !important;
			}
			span {
				color: #cadbff;
			}
			.btn-prev,
			.btn-next {
				background-color: transparent !important;
				color: #cadbff;
			}
			.el-pager li {
				background-color: transparent !important;
			}
			.el-pager .more:before {
				color: #cadbff;
			}
			.el-input__inner {
				background-color: transparent !important;
				border: 1px solid #cadbff !important;
				color: #cadbff !important;
			}
		}
	}
	// 全页面的弹窗样式
	.touzixiangqing {
		// ::v-deep .el-dialog {
		background: transparent;
		// background-color: rgba(#004289, 0.9);
		color: #cadbff;
		/* 表头 */
		p.title {
			font-style: normal;
			color: #ffffff;
			font-family: Helvetica Neue, Helvetica, Arial, PingFang SC,
				Hiragino Sans GB, Heiti SC, Microsoft YaHei, WenQuanYi Micro Hei,
				sans-serif;
			font-size: 22px;
			font-weight: 500;
			i {
				display: inline-block;
				width: 2px;
				height: 8px;
				background-color: #49edf7;
				margin-right: 10px;
				margin-bottom: 2px;
			}
		}
		::v-deep .el-dialog {
			background-color: rgba(#004289, 0.9);
		}
		::v-deep .el-dialog__header {
			padding: 0;
			.el-dialog__title {
				font-style: normal;
				color: #ffffff;
				font-family: Helvetica Neue, Helvetica, Arial, PingFang SC,
					Hiragino Sans GB, Heiti SC, Microsoft YaHei, WenQuanYi Micro Hei,
					sans-serif;
				font-size: 22px;
				font-weight: 500;
			}
			.el-dialog__headerbtn {
				top: 0;
				right: 0;
				width: 40px;
				height: 37px;
				background-color: #003771;
				border-radius: 0px 0px 0px 16px;
			}
			.el-icon-close:before {
				color: #fff;
			}
		}
		::v-deep .el-dialog__body {
			padding: 10px 30px 20px;
			background: url(@/assets/images/toplinelight.png) left top no-repeat;
			background-size: (100% 5px);
		}
	}

	::v-deep .el-row {
		color: #cadbff;
		border-bottom: 1px solid #0062be;
		border-left: 1px solid #0062be;

		.grid-content {
			padding: 10px;
			border-right: 1px solid #0062be;
			border-left: 1px solid #0062be;
			margin-left: -1px;
			padding-left: 10px;
			min-height: 42px;
		}
	}
}
</style>
