<template lang='pug'>
  b-modal#broken-task-modal(title="Broken Challenge", size='sm', :hide-footer="true", v-if='brokenChallengeTask.challenge')
    .modal-body
      div(v-if='brokenChallengeTask.challenge.broken === "TASK_DELETED" || brokenChallengeTask.challenge.broken === "CHALLENGE_TASK_NOT_FOUND"')
        h2 {{ $t('brokenTask') }}
        div
          button.btn.btn-primary(@click='unlink("keep")') {{ $t('keepIt') }}
          button.btn.btn-danger(@click='removeTask(obj)') {{ $t('removeIt') }}
      div(v-if='brokenChallengeTask.challenge.broken === "CHALLENGE_DELETED"')
        h2 {{ $t('brokenChallenge') }}
        div
          button.btn.btn-primary(@click='unlink("keep-all")') {{ $t('keepThem') }}
          button.btn.btn-danger(@click='unlink("remove-all")') {{ $t('removeThem') }}
      div(v-if='brokenChallengeTask.challenge.broken === "CHALLENGE_CLOSED"')
        h2(v-html="$t('challengeCompleted', {user: brokenChallengeTask.challenge.winner})")
        div
          button.btn.btn-primary(@click='unlink("keep-all")') {{ $t('keepThem') }}
          button.btn.btn-danger(@click='unlink("remove-all")') {{ $t('removeThem') }}
      // @TODO: I ported this over, but do we use it anymore?
      //div(v-if='brokenChallengeTask.challenge.broken === "UNSUBSCRIBED"')
        p {{ $t('unsubChallenge') }}
        p
          a(@click="unlink('keep-all')") {{ $t('keepThem') }}
          | &nbsp;|&nbsp;
          a(@click="unlink('remove-all')") {{ $t('removeThem') }}
</template>

<style scoped>
  .modal-body {
    padding-bottom: 2em;
  }
</style>

<script>
import { mapActions } from 'client/libs/store';
import bModal from 'bootstrap-vue/lib/components/modal';
import notifications from 'client/mixins/notifications';

export default {
  mixins: [notifications],
  props: ['brokenChallengeTask'],
  components: {
    bModal,
  },
  methods: {
    ...mapActions({
      destroyTask: 'tasks:destroy',
      unlinkOneTask: 'tasks:unlinkOneTask',
      unlinkAllTasks: 'tasks:unlinkAllTasks',
    }),
    async unlink (keepOption) {
      if (keepOption.indexOf('-all') !== -1) {
        this.unlinkAllTasks({
          challengeId: this.brokenChallengeTask.challenge.id,
          keep: keepOption,
        });
        await this.$store.dispatch('tasks:fetchUserTasks', {forceLoad: true});
        this.close();
        return;
      }

      this.unlinkOneTask({
        task: this.brokenChallengeTask,
        keep: keepOption,
      });
      this.close();
    },
    removeTask () {
      if (!confirm('Are you sure you want to delete this task?')) return;
      this.destroyTask(this.brokenChallengeTask);
    },
    close () {
      this.$root.$emit('hide::modal', 'broken-task-modal');
    },
  },
};
</script>
