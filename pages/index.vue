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
            <b-form-checkbox v-model="overallToggled" switch @change="updateProgress">
              Overall
            </b-form-checkbox>
          </div>
          <div v-for="topic in progressData" class="row mb-1">
            <div class="ml-auto">{{ topic.title }}</div>
            <div class="col-sm-8 pt-1">
              <b-progress :key="topic.title" :max=100 height="2rem">
                 <b-progress-bar v-for="progress in topic.data" :value="progress.value"
                                :variant="progress.variant"></b-progress-bar>
              </b-progress>
            </div>
          </div>
          <div class="d-flex justify-content-center mt-5">
            <div class="ml-auto"></div>
            <div class="col-sm-8 auto">
              <Legend :legend="legend.legend"/>
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

function progressBar(topic, overallToggled) {
  let rendered = 0
  let progressBar = []
  let max = 100
  if (overallToggled === false) {
    max = topic.primary_colour + topic.secondary_colour + topic.white_colour
    if (max === 0) {
      progressBar.push({"variant": "olm-grey", "value": 100})
      return progressBar
    }
  }
  progressBar.push({"variant": "olm-primary", "value": (topic.primary_colour / max) * 100})
  rendered += topic.primary_colour
  progressBar.push({"variant": "olm-secondary", "value": (topic.secondary_colour / max) * 100})
  rendered += topic.secondary_colour
  progressBar.push({"variant": "olm-white", "value": (topic.white_colour / max) * 100})
  rendered += topic.white_colour
  progressBar.push({"variant": "olm-grey", "value": max - rendered})
  return progressBar
}

export default {
  components: {Legend},
  data() {
    return {
      overallToggled: true,
      weekData: [],
      legend: [],
      progressData: []
    }
  },
  async fetch() {
    this.weekData = await this.$axios.$get('user/current')
    this.legend = await this.$axios.$get('legend/my-progress')

    let progressData = []
    for (const topic of this.weekData.topics) {
      progressData.push({"title": topic.title, "data": progressBar(topic, this.overallToggled)})
    }
    this.progressData = progressData
  },
  methods: {
    async updateWeek(newWeek) {
      this.weekData = await this.$axios.$get(`/user/week/${newWeek}`)
      await this.updateProgress()
    },
    async updateProgress() {
      let newProgress = []
      for (let i = 0; i < this.weekData.topics.length; i++) {
        newProgress[i] = {
          "title": this.weekData.topics[i].title,
          "data": progressBar(this.weekData.topics[i], this.overallToggled)
        }
      }
      this.progressData = newProgress
    }
  }
}
</script>
