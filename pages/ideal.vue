<template>
  <b-row>
    <b-col>
      <div>
        <b-col>
          <b-nav class="shadow-sm mb-3" justified tabs>
            <b-nav-item>< Previous</b-nav-item>
            <b-nav-text>Week {{ week }}</b-nav-text>
            <b-nav-item>Next ></b-nav-item>
          </b-nav>
        </b-col>
        <div class="d-flex justify-content-center align-items-center">
          <div>To Date</div>
          <b-form-checkbox v-model="checked" switch>
            Overall
          </b-form-checkbox>
        </div>
        <div v-for="topic in topics" class="row mb-1">
          <div class="ml-auto">{{ topic.title }}</div>
          <div class="col-sm-8 pt-1">
            <b-progress :key="topic.title" :max=101 height="2rem">
              <b-progress-bar :value="topic.understood" variant="olm-primary"></b-progress-bar>
              <b-progress-bar :value="topic.not_understood" variant="olm-secondary"></b-progress-bar>
              <b-progress-bar :value="topic.can_improve" variant="olm-off-white"></b-progress-bar>
              <b-progress-bar :value=1 variant="olm-highlight"></b-progress-bar>
              <b-progress-bar :value="topic.not_covered" variant="olm-grey"></b-progress-bar>
            </b-progress>
          </div>
        </div>
      </div>
      <div>

      </div>
    </b-col>
    <b-col>
      Goal toggle
    </b-col>
  </b-row>
</template>

<script>
export default {
  data() {
    return {
      week: 8,
      topics: [
        {
          title: "Engagement",
          understood: 50.00,
          not_understood: 0,
          can_improve: 16.67,
          not_covered: 33.33
        },
        {
          title: "Assumed knowledge",
          understood: 50,
          not_understood: 50,
          can_improve: 0,
          not_covered: 0
        },
        {
          title: "Python programming",
          understood: 50.00,
          not_understood: 50.00,
          can_improve: 0,
          not_covered: 0
        },
        {
          title: "Unix",
          understood: 75,
          not_understood: 25,
          can_improve: 0,
          not_covered: 0
        },
        {
          title: "Data representation",
          understood: 83.33,
          not_understood: 0,
          can_improve: 0,
          not_covered: 16.67
        },
        {
          title: "Computer architecture",
          understood: 50.00,
          not_understood: 16.67,
          can_improve: 0,
          not_covered: 33.33
        },
        {
          title: "Networks and security",
          understood: 37.50,
          not_understood: 12.50,
          can_improve: 0,
          not_covered: 50.00
        },
        {
          title: "Cloud and containers",
          understood: 0,
          not_understood: 0,
          can_improve: 0,
          not_covered: 100
        },
      ]
    }
  }
}
</script>
