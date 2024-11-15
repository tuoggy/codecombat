<template>
  <div class="start-page center-div">
    <select
      id="language-select"
      v-model="codeLanguage"
      :disabled="!isNewUser"
    >
      <option
        v-for="lang in avaliableLanguages"
        :key="lang"
        :value="lang"
      >
        {{ lang }}
      </option>
    </select>
    <div
      v-if="isNewUser"
      class="timer-tip"
    >
      <input
        v-model="timer"
        name="timer"
        type="checkbox"
      >
      <label for="timer"> {{ $t('exams.timer_tip') }} </label>
    </div>
    <div class="start center-div">
      <input
        :disabled="!hasPermission || !timer"
        type="button"
        :value="buttonValue"
        @click="localStartExam"
      >
      <div v-if="!hasPermission">
        {{ $t('exams.no_permission') }}
      </div>
    </div>
  </div>
</template>

<script>
import { mapGetters, mapActions } from 'vuex'
export default {
  props: {
    examId: {
      type: String,
      required: true,
    },
  },
  data () {
    return {
      codeLanguage: 'python',
      timer: false,
    }
  },
  computed: {
    ...mapGetters('exams', [
      'getExamById',
      'userExam',
    ]),
    exam () {
      return this.getExamById(this.examId)
    },
    avaliableLanguages () {
      return this.exam?.languages || ['python', 'javascript']
    },
    hasPermission () {
      if (this.exam?._id === '-') {
        return true
      }
      const clans = me.get('clans') || []
      return clans.includes(this.exam?.clan)
    },
    buttonValue () {
      if (this.isNewUser) {
        return $.i18n.t('exams.start_exam')
      } else {
        return $.i18n.t('exams.continue_exam')
      }
    },
    limitedDuration () {
      return this.userExam?.duration || this.exam?.duration
    },
    isNewUser () {
      return !this.userExam
    },
    isOldUser () {
      return this.userExam && !this.userExam.archived
    },
    isOldUserExtra () {
      return this.userExam && this.userExam.archived
    },
  },
  mounted () {
    this.checkingUserExam()
  },
  methods: {
    ...mapActions('exams', [
      'startExam',
    ]),
    async localStartExam () {
      if (this.isNewUser) {
        await this.startExam({ examId: this.examId, codeLanguage: this.codeLanguage })
      } else if (this.isOldUserExtra) {
        await this.startExam({ examId: this.examId, codeLanguage: this.codeLanguage, duration: this.userExam.extraDuration })
      }
      application.router.navigate(window.location.pathname.replace(/start$/, 'progress'), { trigger: true })
    },
    checkingUserExam () {
      if (this.isNewUser) {
        return
      }
      this.timer = true // default value for old users
      if (this.isOldUserExtra) {
        return
      }
      const startDate = new Date(this.userExam.startDate)
      const duration = (new Date().getTime() - startDate.getTime()) / (1000 * 60)
      if (duration >= this.limitedDuration) {
        application.router.navigate(window.location.pathname.replace(/start$/, 'end'), { trigger: true })
      }
    },
  },
}

</script>

<style scoped lang="scss">
</style>