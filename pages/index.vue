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
                <b-progress-bar :value="topic.primary_colour" variant="olm-primary"></b-progress-bar>
                <b-progress-bar :value="topic.secondary_colour" variant="olm-secondary"></b-progress-bar>
                <b-progress-bar :value="topic.white_colour" variant="olm-white"></b-progress-bar>
                <b-progress-bar :value="topic.grey_colour" variant="olm-grey"></b-progress-bar>
              </b-progress>
            </div>
          </div>
          <div class="d-flex justify-content-center mt-5">
            <div class="ml-auto"></div>
            <div class="col-sm-8 auto">
              <Legend :legend="legend"/>
            </div>
          </div>
        </b-col>
      </div>
    </b-col>
    <b-col class="flex-container-grey container-fluid min-vh-100 d-flex justify-content-center"></b-col>
  </b-row>
</template>

<script>
import Legend from "@/components/Legend";

export default {
  components: {Legend},
  data() {
    return {
      overallToggled: true,
      weekData: [],
      legend: [
        {
          name: "Understood %",
          variant: "olm-primary"
        },
        {
          name: "Not understood",
          variant: "olm-secondary"
        },
        {
          name: "Can improve",
          variant: "olm-white"
        },
        {
          name: "Not yet covered",
          variant: "olm-grey"
        },
        {
          name: "Ideal progress for goal",
          variant: "olm-highlight"
        }
      ]
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
