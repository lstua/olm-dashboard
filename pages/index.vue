<template>
  <b-row no-gutters>
    <b-col>
      <div>
        <b-col>
          <b-nav justified>
            <b-nav-item :disabled="weekData.previous === null" @click="updateWeek(weekData.previous)">< Previous
            </b-nav-item>
            <b-nav-text>Week {{ weekData.week }}</b-nav-text>
            <b-nav-item :disabled="weekData.next === null" @click="updateWeek(weekData.next)">Next ></b-nav-item>
          </b-nav>
          <div class="d-flex justify-content-center align-items-center mb-2">
            <div class="mr-2">To Date</div>
            <b-form-checkbox v-model="overallToggled" switch>
              Overall
            </b-form-checkbox>
          </div>
          <div v-for="topic in weekData.topics" class="row mb-1">
            <div class="ml-auto">{{ topic.title }}</div>
            <div class="col-sm-8 pt-1">
              <b-progress :key="topic.title" :max=100 height="2rem">
                <b-progress-bar :value="topic.understood" variant="olm-primary"></b-progress-bar>
                <b-progress-bar :value="topic.not_understood" variant="olm-secondary"></b-progress-bar>
                <b-progress-bar :value="topic.can_improve" variant="olm-off-white"></b-progress-bar>
                <b-progress-bar :value="topic.not_covered" variant="olm-grey"></b-progress-bar>
              </b-progress>
            </div>
          </div>
        </b-col>
      </div>
    </b-col>
    <b-col class="flex-container-grey container-fluid min-vh-100 d-flex justify-content-center"></b-col>
  </b-row>
</template>

<script>
export default {
  data() {
    return {
      overallToggled: true,
      weekData: []
    }
  },
  async fetch() {
    this.weekData = await this.$axios.$get('user/current')
  },
  methods: {
    async updateWeek(newWeek) {
      this.weekData = await this.$axios.$get(`/user/week/${newWeek}`)
    }
  }
}
</script>
