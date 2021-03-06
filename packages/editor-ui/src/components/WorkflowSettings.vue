<template>
	<span>
		<el-dialog class="workflow-settings" :visible="dialogVisible" append-to-body width="50%" title="Workflow Settings" :before-close="closeDialog">
			<div v-loading="isLoading">
				<el-row>
					<el-col :span="10" class="setting-name">
						Error Workflow:
						<el-tooltip class="setting-info" placement="top" effect="light">
							<div slot="content" v-html="helpTexts.errorWorkflow"></div>
							<font-awesome-icon icon="question-circle" />
						</el-tooltip>
					</el-col>
					<el-col :span="14" class="ignore-key-press">
						<el-select v-model="workflowSettings.errorWorkflow" placeholder="Select Workflow" size="small" filterable>
							<el-option
								v-for="item in workflows"
								:key="item.id"
								:label="item.name"
								:value="item.id">
							</el-option>
						</el-select>
					</el-col>
				</el-row>
				<el-row>
					<el-col :span="10" class="setting-name">
						Timezone:
						<el-tooltip class="setting-info" placement="top" effect="light">
							<div slot="content" v-html="helpTexts.timezone"></div>
							<font-awesome-icon icon="question-circle" />
						</el-tooltip>
					</el-col>
					<el-col :span="14" class="ignore-key-press">
						<el-select v-model="workflowSettings.timezone" placeholder="Select Timezone" size="small" filterable>
							<el-option
								v-for="timezone of timezones"
								:key="timezone.key"
								:label="timezone.value"
								:value="timezone.key">
							</el-option>
						</el-select>
					</el-col>
				</el-row>
				<el-row>
					<el-col :span="10" class="setting-name">
						Save Data Error Execution:
						<el-tooltip class="setting-info" placement="top" effect="light">
							<div slot="content" v-html="helpTexts.saveDataErrorExecution"></div>
							<font-awesome-icon icon="question-circle" />
						</el-tooltip>
					</el-col>
					<el-col :span="14" class="ignore-key-press">
						<el-select v-model="workflowSettings.saveDataErrorExecution" placeholder="Select Option" size="small" filterable>
							<el-option
								v-for="option of saveDataErrorExecutionOptions"
								:key="option.key"
								:label="option.value"
								:value="option.key">
							</el-option>
						</el-select>
					</el-col>
				</el-row>
				<el-row>
					<el-col :span="10" class="setting-name">
						Save Data Success Execution:
						<el-tooltip class="setting-info" placement="top" effect="light">
							<div slot="content" v-html="helpTexts.saveDataSuccessExecution"></div>
							<font-awesome-icon icon="question-circle" />
						</el-tooltip>
					</el-col>
					<el-col :span="14" class="ignore-key-press">
						<el-select v-model="workflowSettings.saveDataSuccessExecution" placeholder="Select Option" size="small" filterable>
							<el-option
								v-for="option of saveDataSuccessExecutionOptions"
								:key="option.key"
								:label="option.value"
								:value="option.key">
							</el-option>
						</el-select>
					</el-col>
				</el-row>
				<el-row>
					<el-col :span="10" class="setting-name">
						Save Manual Executions:
						<el-tooltip class="setting-info" placement="top" effect="light">
							<div slot="content" v-html="helpTexts.saveManualExecutions"></div>
							<font-awesome-icon icon="question-circle" />
						</el-tooltip>
					</el-col>
					<el-col :span="14" class="ignore-key-press">
						<el-select v-model="workflowSettings.saveManualExecutions" placeholder="Select Option" size="small" filterable>
							<el-option
								v-for="option of saveManualOptions"
								:key="option.key"
								:label="option.value"
								:value="option.key">
							</el-option>
						</el-select>
					</el-col>
				</el-row>

				<div class="action-buttons">
					<el-button type="success" @click="saveSettings">
						Save
					</el-button>
				</div>
			</div>
		</el-dialog>
	</span>
</template>

<script lang="ts">
import Vue from 'vue';

import { restApi } from '@/components/mixins/restApi';
import { genericHelpers } from '@/components/mixins/genericHelpers';
import { showMessage } from '@/components/mixins/showMessage';
import {
	IWorkflowDataUpdate,
	IWorkflowSettings,
	IWorkflowShortResponse,
} from '@/Interface';

import mixins from 'vue-typed-mixins';

export default mixins(
	genericHelpers,
	restApi,
	showMessage,
).extend({
	name: 'WorkflowSettings',
	props: [
		'dialogVisible',
	],
	data () {
		return {
			isLoading: true,
			helpTexts: {
				errorWorkflow: 'The workflow to run in case the current one fails.<br />To function correctly that workflow has to contain an "Error Trigger" node!',
				timezone: 'The timezone in which the workflow should run. Gets for example used by "Cron" node.',
				saveDataErrorExecution: 'If data data of executions should be saved in case they failed.',
				saveDataSuccessExecution: 'If data data of executions should be saved in case they succeed.',
				saveManualExecutions: 'If data data of executions should be saved when started manually from the editor.',
			},
			defaultValues: {
				timezone: 'America/New_York',
				saveDataErrorExecution: 'all',
				saveDataSuccessExecution: 'all',
				saveManualExecutions: false,
			},
			saveDataErrorExecutionOptions: [] as Array<{ key: string, value: string }>,
			saveDataSuccessExecutionOptions: [] as Array<{ key: string, value: string }>,
			saveManualOptions: [] as Array<{ key: string | boolean, value: string }>,
			timezones: [] as Array<{ key: string, value: string }>,
			workflowSettings: {} as IWorkflowSettings,
			workflows: [] as IWorkflowShortResponse[],
		};
	},
	watch: {
		dialogVisible (newValue, oldValue) {
			if (newValue) {
				this.openDialog();
			}
		},
	},
	methods: {
		closeDialog () {
			// Handle the close externally as the visible parameter is an external prop
			// and is so not allowed to be changed here.
			this.$emit('closeDialog');
			return false;
		},
		async loadSaveDataErrorExecutionOptions () {
			this.saveDataErrorExecutionOptions.length = 0;
			this.saveDataErrorExecutionOptions.push.apply( // eslint-disable-line no-useless-call
				this.saveDataErrorExecutionOptions, [
					{
						key: 'DEFAULT',
						value: 'Default - ' + (this.defaultValues.saveDataErrorExecution === 'all' ? 'Save' : 'Do not save'),
					},
					{
						key: 'all',
						value: 'Save',
					},
					{
						key: 'none',
						value: 'Do not save',
					},
				]
			);
		},
		async loadSaveDataSuccessExecutionOptions () {
			this.saveDataSuccessExecutionOptions.length = 0;
			this.saveDataSuccessExecutionOptions.push.apply( // eslint-disable-line no-useless-call
				this.saveDataSuccessExecutionOptions, [
					{
						key: 'DEFAULT',
						value: 'Default - ' + (this.defaultValues.saveDataSuccessExecution === 'all' ? 'Save' : 'Do not save'),
					},
					{
						key: 'all',
						value: 'Save',
					},
					{
						key: 'none',
						value: 'Do not save',
					},
				]
			);
		},
		async loadSaveManualOptions () {
			this.saveManualOptions.length = 0;
			this.saveManualOptions.push({
				key: 'DEFAULT',
				value: 'Default - ' + (this.defaultValues.saveManualExecutions === true ? 'Yes' : 'No'),
			});
			this.saveManualOptions.push({
				key: true,
				value: 'Yes',
			});
			this.saveManualOptions.push({
				key: false,
				value: 'No',
			});
		},

		async loadTimezones () {
			if (this.timezones.length !== 0) {
				// Data got already loaded
				return;
			}

			const timezones = await this.restApi().getTimezones();

			let defaultTimezoneValue = timezones[this.defaultValues.timezone] as string | undefined;
			if (defaultTimezoneValue === undefined) {
				defaultTimezoneValue = 'Default Timezone not valid!';
			}

			this.timezones.push({
				key: 'DEFAULT',
				value: `Default - ${defaultTimezoneValue}`,
			});
			for (const timezone of Object.keys(timezones)) {
				this.timezones.push({
					key: timezone,
					value: timezones[timezone] as string,
				});
			}
		},
		async loadWorkflows () {
			const workflows = await this.restApi().getWorkflows();
			workflows.sort((a, b) => {
				if (a.name.toLowerCase() < b.name.toLowerCase()) {
					return -1;
				}
				if (a.name.toLowerCase() > b.name.toLowerCase()) {
					return 1;
				}
				return 0;
			});

			// @ts-ignore
			workflows.unshift({
				id: undefined as unknown as string,
				name: '- No Workflow -',
			});

			Vue.set(this, 'workflows', workflows);
		},
		async openDialog () {
			const workflowId = this.$route.params.name;
			if (this.$route.params.name === undefined) {
				this.$showMessage({
					title: 'No workflow active',
					message: `No workflow active to display settings of.`,
					type: 'error',
					duration: 0,
				});
				this.closeDialog();
				return;
			}

			this.defaultValues.saveDataErrorExecution = this.$store.getters.saveDataErrorExecution;
			this.defaultValues.saveDataSuccessExecution = this.$store.getters.saveDataSuccessExecution;
			this.defaultValues.saveManualExecutions = this.$store.getters.saveManualExecutions;
			this.defaultValues.timezone = this.$store.getters.timezone;

			this.isLoading = true;
			const promises = [];
			promises.push(this.loadWorkflows());
			promises.push(this.loadSaveDataErrorExecutionOptions());
			promises.push(this.loadSaveDataSuccessExecutionOptions());
			promises.push(this.loadSaveManualOptions());
			promises.push(this.loadTimezones());

			try {
				await Promise.all(promises);
			} catch (error) {
				this.$showError(error, 'Problem loading settings', 'The following error occurred loading the data:');
			}

			const workflowSettings = JSON.parse(JSON.stringify(this.$store.getters.workflowSettings));

			if (workflowSettings.timezone === undefined) {
				workflowSettings.timezone = 'DEFAULT';
			}
			if (workflowSettings.saveDataErrorExecution === undefined) {
				workflowSettings.saveDataErrorExecution = 'DEFAULT';
			}
			if (workflowSettings.saveDataSuccessExecution === undefined) {
				workflowSettings.saveDataSuccessExecution = 'DEFAULT';
			}
			if (workflowSettings.saveManualExecutions === undefined) {
				workflowSettings.saveManualExecutions = 'DEFAULT';
			}

			Vue.set(this, 'workflowSettings', workflowSettings);
			this.isLoading = false;
		},
		async saveSettings () {
			// Set that the active state should be changed
			const data: IWorkflowDataUpdate = {
				settings: this.workflowSettings,
			};

			this.isLoading = true;

			try {
				await this.restApi().updateWorkflow(this.$route.params.name, data);
			} catch (error) {
				this.$showError(error, 'Problem saving settings', 'There was a problem saving the settings:');
				this.isLoading = false;
				return;
			}

			// Get the settings without the defaults set for local workflow settings
			const localWorkflowSettings: IWorkflowSettings = {};
			for (const key of Object.keys(this.workflowSettings)) {
				if (this.workflowSettings[key] !== 'DEFAULT') {
					localWorkflowSettings[key] = this.workflowSettings[key];
				}
			}

			this.$store.commit('setWorkflowSettings', localWorkflowSettings);

			this.isLoading = false;

			this.$showMessage({
				title: 'Settings saved',
				message: 'The workflow settings got saved!',
				type: 'success',
			});

			this.closeDialog();
		},
	},
});
</script>

<style scoped lang="scss">
.workflow-settings {
	.el-row {
		padding: 0.25em 0;
	}
}

.action-buttons {
	margin-top: 1em;
	text-align: right;
}

.setting-info {
	display: none;
}

.setting-name {
	line-height: 32px;
}

.setting-name:hover {
	.setting-info {
		display: inline;
	}
}

</style>
