<template>
  <Modal
    v-model="show"
    :width="580">
    <div slot="header">
      <div class="header-tab">
        <ButtonGroup
          shape="circle"
          size="large">
          <Button
            :type="switcher === 'job' ? 'primary': 'default'"
            @click="getRunningJobs">任务管理器</Button>
          <Button
            :type="switcher === 'session' ? 'primary': 'default'"
            @click="getEngines">引擎管理器</Button>
          <Button
            :type="switcher === 'queue' ? 'primary': 'default'"
            @click="getQueue">队列管理器</Button>
        </ButtonGroup>
      </div>
    </div>
    <div class="resource-simple-content">
      <job
        ref="job"
        v-if="switcher === 'job'"
        @close-modal="close"
        @change-loading="changeLoading"
        @change-job-disabled="changeJobDisabled"/>
      <engine
        ref="engine"
        v-else-if="switcher === 'session'"
        @disabled="engineDisabledChange"
        @change-loading="changeLoading"/>
      <queue
        ref="queue"
        v-else></queue>
    </div>
    <template slot="footer">
      <div
        class="resource-simple-footer legendShow">
        <point
          v-if="pointList.length"
          :point-list="pointList"></point>
        <div>
          <Button
            type="default"
            @click="rest">
            <Icon
              size="18"
              type="ios-repeat"></Icon>
            刷新
          </Button>
          <Button
            type="default"
            v-if="switcher === 'job'"
            :disabled="isJobBtnDisabled"
            @click="openKillModal">
            结束任务
          </Button>
          <Button
            :disabled="engineDisable"
            type="default"
            v-if="switcher === 'session'"
            @click="openKillModal">
            结束引擎
          </Button>
        </div>
      </div>
    </template>
    <detele-modal
      ref="killModal"
      :loading="loading"
      @delete="killJob"></detele-modal>
  </Modal>
</template>
<script>
import deteleModal from '@js/component/deleteDialog';
import module from './index';
import job from './job.vue';
import engine from './engine.vue';
import queue from './queue.vue';
import point from './point.vue';

export default {
    components: {
        job,
        engine,
        queue,
        point,
        deteleModal,
    },
    mixins: [module.mixin],
    data() {
        return {
            dataList: [],
            show: false,
            switcher: 'job',
            pointList: [],
            loading: false,
            isJobBtnDisabled: true,
            engineDisable: true,
        };
    },
    methods: {
        getRunningJobs() {
            this.switcher = 'job';
            this.getPointList();
            this.$nextTick(() => {
                this.$refs.job.getJobList();
            });
        },
        getEngines() {
            this.switcher = 'session';
            this.getPointList();
            this.$nextTick(() => {
                this.$refs.engine.getEngineData();
            });
        },
        getQueue() {
            this.switcher = 'queue';
            this.getPointList();
            this.$nextTick(() => {
                this.$refs.queue.getQueueList();
            });
        },
        open() {
            this.show = true;
            if (this.switcher === 'job') {
                this.isJobBtnDisabled = true;
                this.getRunningJobs();
            } else if (this.switcher === 'session') {
                this.getEngines();
            } else {
                this.getQueue();
            }
        },
        close() {
            this.show = false;
        },
        killSuccess(type) {
            this.getRunningJobs();
        },
        rest() {
            this.open();
        },
        getPointList() {
            let list = null;
            if (this.switcher === 'queue') {
                list = [{
                    type: 'idle',
                    label: '空闲',
                }, {
                    type: 'busy',
                    label: '繁忙',
                }];
            } else if (this.switcher === 'job') {
                list = [{
                    type: 'init',
                    label: '排队中',
                }, {
                    type: 'schedule',
                    label: '资源申请中',
                }, {
                    type: 'running',
                    label: '运行',
                }];
            } else {
                list = [{
                    type: 'busy',
                    label: '繁忙',
                }, {
                    type: 'idle',
                    label: '空闲',
                }, {
                    type: 'starting',
                    label: '启动',
                }];
            }
            this.pointList = list;
        },
        openKillModal() {
            const type = this.switcher === 'session' ? '引擎' : '任务';
            this.$refs.killModal.open({ type, name: '' });
        },
        killJob() {
            if (this.switcher === 'session') {
                this.$refs.engine.killJob();
            } else {
                this.$refs.job.killJob();
            }
        },
        changeLoading(val) {
            this.loading = val;
        },
        changeJobDisabled(val) {
            this.isJobBtnDisabled = val;
        },
        engineDisabledChange(params) {
            this.engineDisable = params;
        },
    },
};
</script>
<style lang="scss" src="./index.scss">
</style>
