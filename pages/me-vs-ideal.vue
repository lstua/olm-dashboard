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
          <div class="d-flex justify-content-center align-items-center mb-2">
            <div class="mr-2">To Date</div>
            <b-form-checkbox v-model="overallToggled" switch @change="updateProgress">
              Overall
            </b-form-checkbox>
          </div>
          <div v-for="topic in progressData" v-if="fetching === false" class="row mb-1">
            <div class="ml-auto">{{ topic.title }}</div>
            <div class="col-sm-8 auto pt-1">
              <b-progress :key="topic.title" :max=101 height="2rem">
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
    <b-col class="flex-container-grey container-fluid d-flex justify-content-center">
      <div>
        <b-col class="flex-container-white mx-4" justified style="width: 300px">
          <b-form-group>
            <label style="font-weight: bold">Goal</label>
            <b-form-radio
              v-for="goal in goals"
              :key="goal.id"
              v-model="selectedGoal"
              :value="goal.id"
              button-variant="danger"
              @change="updateGoal">
              {{ goal.name }}
            </b-form-radio>
          </b-form-group>
        </b-col>
      </div>
    </b-col>
  </b-row>
</template>

<script>
import Legend from "@/components/Legend";

function progressBar(topic, goal, overallToggled) {
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
  if (topic.primary_colour >= goal) {
    progressBar.push({"variant": "olm-primary", "value": ((topic.primary_colour - goal) / max) * 100})
    rendered += topic.primary_colour - goal
  } else {
    progressBar.push({"variant": "olm-secondary", "value": ((goal - topic.primary_colour) / max) * 100})
    rendered += goal - topic.primary_colour
  }
  progressBar.push({"variant": "olm-grey", "value": 100 - rendered})
  return progressBar
}

export default {
  components: {Legend},
  data() {
    return {
      fetching: true,
      selectedGoal: null,
      overallToggled: true,
      weekData: [],
      goals: [],
      goalData: [],
      progressData: [],
      legend: []
    }
  },
  async fetch() {
    this.weekData = await this.$axios.$get('user/current')
    const goalsData = await this.$axios.$get('goals')
    this.goals = goalsData.goals
    this.selectedGoal = this.goals[0].id
    this.goalData = await this.$axios.$get(`${this.selectedGoal}/week/${this.weekData.week}`)
    this.legend = await this.$axios.$get('legend/me-vs-ideal')

    let progressData = []
    for (const topic of this.weekData.topics) {
      for (const goal of this.goalData.topics) {
        if (topic.title === goal.title) {
          progressData.push({"title": topic.title, "data": progressBar(topic, goal.goal, this.overallToggled)})
        }
      }
    }
    this.progressData = progressData
    this.fetching = false
  },
  methods: {
    async updateWeek(newWeek) {
      this.weekData = await this.$axios.$get(`/user/week/${newWeek}`)
      await this.updateProgress()
    },
    async updateGoal() {
      this.goalData = await this.$axios.$get(`${this.selectedGoal}/week/${this.weekData.week}`)
      await this.updateProgress()
    },
    async updateProgress() {
      let newProgress = []
      for (let i = 0; i < this.weekData.topics.length; i++) {
        for (const goal of this.goalData.topics) {
          if (this.weekData.topics[i].title === goal.title) {
            newProgress[i] = {
              "title": this.weekData.topics[i].title,
              "data": progressBar(this.weekData.topics[i], goal.goal, this.overallToggled)
            }
          }
        }
      }
      this.progressData = newProgress
    }
  }
}
</script>
