<template>

  <div class="task-progress">
    <div class="progress-icon dtc">
      <mat-svg
        v-if="!taskHasFailed"
        category="action"
        name="autorenew"
        class="inprogress"
      />
      <mat-svg
        v-else
        category="alert"
        name="error"
        class="error"
      />
    </div>

    <div class="progress-bar dtc">
      <div class="task-stage">
        {{ stageText }}
      </div>
      <ui-progress-linear
        class="progress-linear"
        :progress="formattedPercentage"
        :type="progressBarType"
        color="primary"
      />
    </div>

    <div class="progress-messages dtc">
      <span class="percentage">{{ progressMessage }}</span>
    </div>

    <div class="buttons dtc">
      <k-button
        v-if="taskHasCompleted || cancellable"
        :text="taskHasCompleted ? $tr('close') : $tr('cancel')"
        :primary="false"
        :raised="false"
        @click="endTask()"
        :disabled="uiBlocked"
      />
    </div>

  </div>

</template>


<script>

  import UiProgressLinear from 'keen-ui/src/UiProgressLinear';
  import kButton from 'kolibri.coreVue.components.kButton';
  import { refreshChannelList } from '../../state/actions/manageContentActions';
  import { cancelTask } from '../../state/actions/taskActions';
  import { TaskTypes, TaskStatuses } from '../../constants';

  const RequiredString = {
    type: String,
    required: true,
  };

  export default {
    name: 'taskProgress',
    components: {
      UiProgressLinear,
      kButton,
    },
    props: {
      type: RequiredString,
      status: RequiredString,
      percentage: {
        type: Number,
        required: true,
      },
      id: RequiredString,
      cancellable: {
        type: Boolean,
        required: true,
      },
    },
    watch: {
      taskHasFailed: () => {
        if (this.taskHasFailed) {
          this.$emit('taskfailed');
        }
      },
    },
    data() {
      return {
        uiBlocked: false,
      };
    },
    computed: {
      TaskStatuses: () => TaskStatuses,
      stageText() {
        if (this.status === TaskStatuses.RUNNING) {
          switch (this.type) {
            case TaskTypes.REMOTE_IMPORT:
            case TaskTypes.LOCAL_IMPORT:
              return this.$tr('importingContent');
            case TaskTypes.LOCAL_EXPORT:
              return this.$tr('exportingContent');
            case TaskTypes.DELETE_CHANNEL:
              return this.$tr('deletingChannel');
            default:
              return '';
          }
        }
        if (this.taskHasFailed) {
          return this.$tr('taskHasFailed');
        }
        if (this.taskHasCompleted) {
          return this.$tr('finished');
        }
        if (this.taskIsPreparing) {
          return this.$tr('preparingTask');
        }
      },
      taskHasFailed() {
        return this.status === TaskStatuses.FAILED;
      },
      taskHasCompleted() {
        return this.status === TaskStatuses.COMPLETED;
      },
      taskIsPreparing() {
        return this.status === TaskStatuses.QUEUED || this.status === TaskStatuses.SCHEDULED;
      },
      formattedPercentage() {
        return this.percentage * 100;
      },
      progressMessage() {
        if (this.percentage > 0) {
          return this.formattedPercentage.toFixed(1) + '%';
        }
        return '';
      },
      progressBarType() {
        return this.taskIsPreparing ? 'indeterminate' : 'determinate';
      },
    },
    methods: {
      endTask() {
        this.uiBlocked = true;
        if (this.taskHasCompleted) {
          this.$emit('taskcomplete');
          this.refreshChannelList();
        }
        this.cancelTask(this.id).then(() => {
          this.uiBlocked = false;
        });
      },
    },
    vuex: {
      actions: {
        cancelTask,
        refreshChannelList,
      },
    },
    $trs: {
      importingContent: 'Importing content…',
      exportingContent: 'Exporting content…',
      finished: 'Finished!',
      preparingTask: 'Preparing…',
      close: 'Close',
      cancel: 'Cancel',
      taskHasFailed: 'Transfer failed. Please try again.',
      deletingChannel: 'Deleting channel…',
    },
  };

</script>


<style lang="stylus" scoped>

  @require '~kolibri.styles.definitions'

  .progress-icon
    text-align: center
    width: 5%
    .inprogress
      fill: $core-status-progress
    .error
      fill: $core-text-error

  .task-progress
    display: table
    width: 100%
    height: 5em
    vertical-align: middle
    padding-left: 1em
    padding-right: 1em

  .task-stage
    margin-bottom: 0.5em

  .progress-bar
    width: 50%
    font-size: 0.75em
    padding-bottom: 10px

  .progress-messages
    padding-left: 1em
    padding-right: 1em

  .percentage
    font-weight: bold

  .buttons
    text-align: right

  .dtc
    display: table-cell
    vertical-align: inherit

</style>
