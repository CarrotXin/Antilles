<style lang="css">
	.dashboard-monitor {

	}
	.margin-right-20 {
		margin-right: 20px;
	}
	.margin-bottom-20 {
		margin-bottom: 20px;
	}
	.dashboard-backgroud-color {
		background: #fff;
	}
	.dashboard-chart-height {
		height: 240px;
	}
	.dashboard-card {
		padding: 10px;

	}
	.dislplay-flex {
		display: flex;
	}
	.dislplay-flex-column {
		display: flex;
		flex-direction: column;
	}
	.float-right {
		float: right;
	}
	.overflow-hidden {
		overflow: hidden;
	}
	.dashboard-chart-select {
		width: 180px;
	}
	.dashboard-jobChart{
		/*max-height: 528px;*/
	}
	.dashboard-more {
		color: #5FB4F9;
	}
</style>
<template lang="html">
  <div class="dashboard-monitor dislplay-flex-column height--100">
		<el-row class="">
			<node-hardware :init-data='initOverData'></node-hardware>
		</el-row>
		<el-row class="dislplay-flex height--100" style="flex-wrap: wrap;">
			<el-col :xs="24" :sm="24" :md="12" :lg="6" class=" dislplay-flex-column" >
				<div class="dashboard-card height--100">
					<div class="p-20 dashboard-backgroud-color height--100">
						<el-row class="">
							<el-radio-group size='small' class="float-right" v-model="jobListDefalut" @change='onJobListChange'>
								<el-radio-button label="running">{{$t('Dashboard.JobList.Status.running')}}</el-radio-button>
								<el-radio-button label="waiting">{{$t('Dashboard.JobList.Status.waiting')}}</el-radio-button>
							</el-radio-group>
							<span>{{ $t('Dashboard.Job.List.Title') }}</span>
						</el-row>
						<job-list :init-data='initJobListData' :status='jobListDefalut'></job-list>
					</div>
				</div>
			</el-col>
			<el-col :xs="24" :sm="24" :md="12" :lg="12" class=" dislplay-flex-column" >
				<div class="dashboard-card height--100">
					<div v-if='initJobChartData' class="p-20 dashboard-backgroud-color dashboard-jobChart dislplay-flex-column  height--100">
						<div class="">
							<el-select class="dashboard-chart-select float-right"
								v-model="selectedTimeDefalut"
								@change='onJobTimeChange'>
								<el-option
									v-for="item in selectTimeOptions"
									:key="item.value"
									:label="$t('Dashboard.JobChart.TimeSelect.'+ item.label)"
									:value="item.value">
								</el-option>
							</el-select>
								<el-select class="dashboard-chart-select float-right"
									v-model="jobQueueDefalut"
									@change='onJobQueueChange'
									style="margin-right:10px;">
									<el-option
										v-for="item in initJobQueueData"
										:key="item.value"
										:label="item.label=='all'?$t('Dashboard.JobChart.Queue.all'):item.label"
										:value="item.value">
									</el-option>
								</el-select>
								<span>{{ $t('Dashboard.Job.Chart.Title')}}</span>
						</div>
						<job-chart :init-data='initJobChartData'></job-chart>
					</div>
					<div v-else v-loading='true' class="p-20 dashboard-backgroud-color dashboard-jobChart dislplay-flex-column  height--100"></div>
				</div>
			</el-col>
			<el-col :xs="24" :sm="24" :md="24" :lg="6" class="dashboard-card height--100" style="min-height:530px;">
				<div v-loading='!initTemplateData' class="p-20 dashboard-backgroud-color height--100" style="overflow: hidden; min-height:510px;">
					<dashboard-template v-if='initTemplateData' :init-data='initTemplateData'></dashboard-template>
				</div>
			</el-col>
		</el-row>
	</div>
</template>

<script>
import NodeHardware from './dashboard/dashboard-node-hardware'
import NodeStatus from './../widget/node-panel/node-status-chart'
import JobList from './dashboard/job-list'
import JobChart from './dashboard/job-chart'
import DashboardTemplate from './dashboard/dashboard-job-template'
import DashboardService from './../service/dashboard-monitor'
import Format from './../common/format'
export default {
	data() {
		return {
			role: this.$store.state.auth.access,
			initOverData: null,
			initJobListData: true,
			initJobChartData: null,
			initTemplateData: true,
			jobQueueDefalut: 'all',
			initJobQueueData: [{label: 'all',value: 'all'}],
			selectedTimeDefalut: 3600,
			selectTimeOptions:[
				{label: 'hour',value: 3600},
				{label: 'day',value: 3600*24},
				{label: 'week',value: 3600*24*7},
				{label: 'month',value: 3600*24*30}
			],
			jobListDefalut: 'running',
			refreshTimeout: null,
			refreshInterval: 10000
		};
	},
	components: {
		'node-hardware': NodeHardware,
		'job-list': JobList,
		'job-chart': JobChart,
		'dashboard-template': DashboardTemplate
	},
	mounted() {
		DashboardService.getJobChartQueue().then((res) => {
			res.forEach((item) => {
				this.initJobQueueData.push({
					label: item.name,
					value: item.name
				}) ;
			})
		});
		this.init();
	},

	beforeDestroy() {
    clearTimeout(this.refreshTimeout);
  },
	methods:{
		init() {
			DashboardService.getDashboardOverview().then((res) => {
				this.initOverData = res;
				if(!this.refreshTimeout) {
					this.refresh()
				}
			});
			DashboardService.getUserLatestTemplate().then((res) => {
				this.initTemplateData = res;
				if(!this.refreshTimeout) {
					this.refresh()
				}
			});
			this.getJoblist();
			this.getJobChart();
		},
		getJobChart() {
			DashboardService.getDashboardJobChart(this.selectedTimeDefalut, this.jobQueueDefalut, this.role).then((res) => {
				this.initJobChartData = res;
				if(!this.refreshTimeout) {
					this.refresh()
				}
			});
		},
		getJoblist() {
			var status = this.jobListDefalut.toLowerCase();
			DashboardService.getDashboardJobList(10, status, this.role).then((res) => {
				this.initJobListData = res;
				if(!this.refreshTimeout) {
					this.refresh()
				}
			});
		},
		onJobListChange(val){
			this.getJoblist();
    },
		onJobQueueChange(val) {
			this.getJobChart();
		},
		onJobTimeChange(val) {
			this.getJobChart();
		},
		refresh() {
			this.refreshTimeout = setTimeout(() => {
				this.refreshTimeout = null;
				this.init();
			}, this.refreshInterval)
		}
	}
}
</script>
