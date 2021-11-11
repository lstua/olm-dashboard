<template>
  <b-table-simple borderless class="text-xsmall" responsive small style="width: 200px">
    <b-tbody>
      <b-tr v-for="i of legend" :key="i.name" v-b-tooltip.hover="i.info">
        <b-td :class="'bg-olm-' + i.variant" style="width: 30px"></b-td>
        <b-td style="font-size: 14px">{{ i.name }}</b-td>
      </b-tr>
    </b-tbody>
  </b-table-simple>
</template>

<script>
export default {
  name: 'Legend',
  props: {
    legend: {}
  }
}
</script>
