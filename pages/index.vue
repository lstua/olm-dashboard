<template>
  <b-row no-gutters>
    <b-col>
      <div>
        <b-col>
          <b-nav justified>
            <b-nav-item :disabled="weekData.previous === null" @click="updateWeek(weekData.previous)">< Previous
            </b-nav-item>
            <b-nav-text><strong>Week {{ weekData.week }}</strong></b-nav-text>
            <b-nav-item :disabled="weekData.next === null" @click="updateWeek(weekData.next)">Next ></b-nav-item>
          </b-nav>
          <b-row>
            <b-col class="col-sm-2"></b-col>
            <b-col class="d-flex justify-content-center align-items-center mb-2">
              <div class="mr-2">To Date</div>
              <b-form-checkbox v-model="overallToggled" switch @change="updateProgress">
                Overall
              </b-form-checkbox>
            </b-col>
            <b-col class="col-sm-2">
              <b-button id="info" class="mb-2 float-right" pill size="xs" variant="outline-primary"> ?</b-button>
              <b-popover placement="rightbottom" target="info" triggers="hover">
                <template #title>Info</template>
                On this page you can:
                <ul>
                  <li> Navigate between weeks</li>
                  <li> Toggle between views of course content covered to date, and covered overall</li>
                  <li> Click on topics for more details</li>
                </ul>
                Or navigate to the Ideal or Me vs Ideal pages!
              </b-popover>
            </b-col>
          </b-row>
          <div v-for="topic in progressData" :key="topic.title" class="row mb-1" @click="detailView(topic.title)">
            <div class="ml-auto">{{ topic.title }}</div>
            <div class="col-sm-8 pt-1">
              <b-progress :key="topic.title" :max=100 height="2rem">
                <b-progress-bar v-for="progress in topic.data" :key="progress.value + progress.variant"
                                v-b-tooltip.hover="progress.value.toFixed(2)"
                                :value="progress.value" :variant="progress.variant"></b-progress-bar>
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
    <b-col style="background-color: #e6e6e6">
      <b-row no-gutters>
        <b-col v-if="selectedTopicData.length !== 0" class="mx-4" style="background-color: #fafafa">
          <b-col class="mb-3">
            <b-row class="mb-3 mt-2" no-gutters>
              <b-col class="col-sm-10 auto pt-1">
                <strong>{{ selectedTopic }}</strong>
              </b-col>
              <b-col class="pt-1">
                Weight
              </b-col>
            </b-row>
            <b-row v-for="assessment in selectedTopicData" :key="assessment.title" class="mb-1" no-gutters>
                <div class="ml-auto mr-3">{{ assessment.title }}</div>
                <b-col class="col-sm-6 auto pt-1">
                  <b-progress :key="assessment.title" :max=100 height="2rem">
                    <b-progress-bar v-for="progress in assessment.data" :key="progress.value + progress.variant"
                                    v-b-tooltip.hover="progress.value.toFixed(2)"
                                    :value="progress.value" :variant="progress.variant"></b-progress-bar>
                  </b-progress>
                </b-col>
                <b-col class="col-sm-2">
                  <div class="ml-3">{{ assessment.weight }}</div>
                </b-col>
            </b-row>
          </b-col>
        </b-col>
      </b-row>
    </b-col>
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
      progressData: [],
      selectedTopic: "",
      selectedTopicData: []
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
    async detailView(topic) {
      const selectedTopic = await this.$axios.$get(`/user/week/${this.weekData.week}/${topic}`)
      let newProgress = []
      for (let i = 0; i < selectedTopic.assessments.length; i++) {
        newProgress[i] = {
          "title": selectedTopic.assessments[i].title,
          "data": progressBar(selectedTopic.assessments[i], this.overallToggled),
          "weight": selectedTopic.assessments[i].weight + "%"
        }
      }
      this.selectedTopic = selectedTopic.title
      this.selectedTopicData = newProgress
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
