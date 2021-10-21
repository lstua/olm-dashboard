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

    function progressBar(topic, goal) {
      let rendered = 0
      const progressBar = []
      if (topic.understood < goal) {
        progressBar.push({"variant": "olm-primary", "value": topic.understood})
        rendered += topic.understood
      }
      else if (topic.understood >= goal) {
        progressBar.push({"variant": "olm-primary", "value": goal})
        progressBar.push({"variant": "olm-highlight", "value": 1})
        progressBar.push({"variant": "olm-primary", "value": topic.understood - goal})
        rendered += topic.understood
      }
      if (topic.not_understood <= goal) {
        progressBar.push({"variant": "olm-secondary", "value": topic.not_understood})
        rendered += topic.not_understood
      }
      else if (topic.not_understood >= goal) {
        progressBar.push({"variant": "olm-secondary", "value": goal - rendered})
        progressBar.push({"variant": "olm-highlight", "value": 1})
        progressBar.push({"variant": "olm-secondary", "value": topic.not_understood - goal + rendered})
        rendered += topic.not_understood
      }
      if (topic.can_improve <= goal) {
        progressBar.push({"variant": "olm-off-white", "value": topic.can_improve})
        rendered += topic.can_improve
      }
      else if (topic.can_improve >= goal) {
        progressBar.push({"variant": "olm-off-white", "value": goal - rendered})
        progressBar.push({"variant": "olm-highlight", "value": 1})
        progressBar.push({"variant": "olm-off-white", "value": topic.can_improve - goal + rendered})
        rendered += topic.can_improve
      }
      if (rendered <= goal) {
        progressBar.push({"variant": "olm-highlight", "value": 1})
      }
      progressBar.push({"variant": "olm-grey", "value": 100 - rendered})

      return progressBar
    }
  },
  methods: {
    async updateWeek(newWeek) {
      this.weekData = await this.$axios.$get(`/user/week/${newWeek}`)
    },
    async updateGoal() {
      this.fetching = true
      this.goalData = await this.$axios.$get(`${this.selectedGoal}/week/${this.weekData.week}`)
      this.fetching = false
    },

  }
}
</script>
