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
                On this page you can choose a goal on the right to see how you are tracking!
                <br><br>
                This page shows you the ideal student progress in each topic for the goal you have selected.
              </b-popover>
            </b-col>
          </b-row>
          <div v-for="topic in progressData" v-if="fetching === false" :key="topic.title" class="row mb-1">
            <div class="ml-auto">{{ topic.title }}</div>
            <div class="col-sm-8 auto pt-1">
              <b-progress :key="topic.title" :max=101 height="2rem">
                <b-progress-bar v-for="progress in topic.data"
                                v-b-tooltip.hover="progress.hover.toFixed(2)" :value="progress.value"
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

function progressBarHelper(value, goal, progressBar, rendered, variant, max) {
  if (value + rendered < goal) {
    progressBar.push({"variant": variant, "value": (value / max) * 100, "hover": (value / max) * 100})
  } else if (value + rendered >= goal && rendered < goal) {
    progressBar.push({"variant": variant, "value": ((goal - rendered) / max) * 100, "hover": (value / max) * 100})
    progressBar.push({"variant": "olm-highlight", "value": 1, "hover": goal})
    progressBar.push({
      "variant": variant,
      "value": ((value - goal + rendered) / max) * 100,
      "hover": (value / max) * 100
    })
  } else {
    progressBar.push({"variant": variant, "value": (value / max) * 100, "hover": (value / max) * 100})
  }
  return progressBar;
}

function progressBar(topic, goal, overallToggled) {
  let rendered = 0
  let progressBar = []
  let max = 100
  if (overallToggled === false) {
    max = topic.primary_colour + topic.secondary_colour + topic.white_colour
    if (max === 0) {
      progressBar.push({"variant": "olm-grey", "value": 100, "hover": 100})
      return progressBar
    }
  }
  progressBar = progressBarHelper(topic.primary_colour, goal, progressBar, rendered, "olm-primary", max)
  rendered += topic.primary_colour
  progressBar = progressBarHelper(topic.secondary_colour, goal, progressBar, rendered, "olm-secondary", max)
  rendered += topic.secondary_colour
  progressBar = progressBarHelper(topic.white_colour, goal, progressBar, rendered, "olm-white", max)
  rendered += topic.white_colour
  if (rendered < goal) {
    progressBar.push({"variant": "olm-highlight", "value": 1, "hover": rendered})
  }
  progressBar.push({"variant": "olm-grey", "value": max - rendered, "hover": max - rendered})
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
    this.legend = await this.$axios.$get('legend/ideal')

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
