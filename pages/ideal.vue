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
          <div v-if="fetching === false" v-for="topic in progressData" class="row mb-1">
            <div class="ml-auto">{{ topic.title }}</div>
            <div class="col-sm-8 auto pt-1">
              <b-progress :key="topic.title" :max=101 height="2rem">
                <b-progress-bar v-for="progress in topic.data" :value="progress.value" :variant="progress.variant"></b-progress-bar>
              </b-progress>
            </div>
          </div>
        </b-col>
      </div>
    </b-col>
    <b-col class="flex-container-grey container-fluid min-vh-100 d-flex justify-content-center">
      <div>
        <b-col class="flex-container-off-white mx-4" justified style="width: 300px">
          <b-form-group label="Goal"
          >
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
function progressBarHelper(value, goal, progressBar, rendered, variant) {
  console.log("value: " + value + " goal: " + goal + " rendered: " + rendered + " " + variant)
  if (value === 0) {
    return progressBar
  }
  if (value + rendered < goal) {
    progressBar.push({"variant": variant, "value": value})
  } else if (value + rendered >= goal && rendered < goal) {
    progressBar.push({"variant": variant, "value": goal - rendered})
    progressBar.push({"variant": "olm-highlight", "value": 1})
    progressBar.push({"variant": variant, "value": value - goal + rendered})
  }
  return progressBar;
}

function progressBar(topic, goal) {
  let rendered = 0
  let progressBar = []
  console.log(topic)
  console.log(goal)
  progressBar = progressBarHelper(topic.understood, goal, progressBar, rendered, "olm-primary");
  rendered += topic.understood
  progressBar = progressBarHelper(topic.not_understood, goal, progressBar, rendered, "olm-secondary");
  rendered += topic.not_understood
  progressBar = progressBarHelper(topic.can_improve, goal, progressBar, rendered, "olm-off-white");
  rendered += topic.can_improve
  if (rendered < goal) {
    progressBar.push({"variant": "olm-highlight", "value": 1})
  }
  progressBar.push({"variant": "olm-grey", "value": 100 - rendered})
  return progressBar
}
export default {
  data() {
    return {
      fetching: true,
      selectedGoal: null,
      overallToggled: true,
      weekData: [],
      goals: [],
      goalData: [],
      progressData: []
    }
  },
  async fetch() {
    this.weekData = await this.$axios.$get('user/current')
    const goalsData = await this.$axios.$get('goals')
    this.goals = goalsData.goals
    this.selectedGoal = this.goals[0].id
    this.goalData = await this.$axios.$get(`${this.selectedGoal}/week/${this.weekData.week}`)

    const progressData = []
    for (const topic of this.weekData.topics) {
      for (const goal of this.goalData.topics) {
        if (topic.title === goal.title) {
          progressData.push({"title": topic.title, "data": progressBar(topic, goal.understood)})
        }
      }
    }
    this.progressData = progressData
    this.fetching = false
  },
  methods: {
    async updateWeek(newWeek) {
      this.weekData = await this.$axios.$get(`/user/week/${newWeek}`)
    },
    async updateGoal() {
      this.fetching = true
      this.goalData = await this.$axios.$get(`${this.selectedGoal}/week/${this.weekData.week}`)
      const progressData = []
      for (const topic of this.weekData.topics) {
        for (const goal of this.goalData.topics) {
          if (topic.title === goal.title) {
            progressData.push({"title": topic.title, "data": progressBar(topic, goal.understood)})
          }
        }
      }
      this.progressData = progressData
      this.fetching = false
    },

  }
}
</script>
