<template>

  <core-modal :title="$tr('deleteLearnerGroup')"
    @cancel="close">
    <p>{{ $tr('areYouSure', { groupName: groupName }) }}</p>
    <p>{{ $tr('learnersWillBecome') }} <strong>{{ $tr('ungrouped') }}</strong>.</p>
    <k-button :text="$tr('cancel')"
      :raised="false"
      @click="close" />
    <k-button :text="$tr('deleteGroup')"
      :primary="true"
      @click="deleteGroup(groupId)" />
  </core-modal>

</template>


<script>

  import * as groupActions from '../../state/actions/group';
  import coreModal from 'kolibri.coreVue.components.coreModal';
  import kTextbox from 'kolibri.coreVue.components.kTextbox';
  import kButton from 'kolibri.coreVue.components.kButton';
  export default {
    name: 'deleteGroupModal',
    $trs: {
      deleteLearnerGroup: 'Delete Learner Group',
      areYouSure: "Are you sure you want to delete '{ groupName }'?",
      learnersWillBecome: 'Learners within this group will become',
      ungrouped: 'Ungrouped',
      cancel: 'Cancel',
      deleteGroup: 'Delete Group',
    },
    components: {
      coreModal,
      kTextbox,
      kButton,
    },
    props: {
      groupName: {
        type: String,
        required: true,
      },
      groupId: {
        type: String,
        required: true,
      },
    },
    methods: {
      close() {
        this.displayModal(false);
      },
    },
    vuex: {
      actions: {
        displayModal: groupActions.displayModal,
        deleteGroup: groupActions.deleteGroup,
      },
    },
  };

</script>


<style lang="stylus" scoped></style>
