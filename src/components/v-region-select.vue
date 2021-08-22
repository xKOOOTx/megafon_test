<template>
  <div class="u-select" @click="areOptionsVisible = !areOptionsVisible">
    <p class="u-select__name">{{ regionSelected }}</p>
    <div
      class="u-select__options"
      v-if="areOptionsVisible"
    >
      <p
        class="u-select__options_name"
        v-for="option in options[0]"
        :key="option.name"
        @click="selectRegionOption(option)"
      >
        {{ option.name }}
      </p>
    </div>
  </div>
</template>

<script>
export default {
  name: 'v-region-select',
  data () {
    return {
      areOptionsVisible: false
    }
  },
  props: {
    options: {
      type: Array,
      default () {
        return []
      }
    },
    regionSelected: {
      type: String,
      default: ''
    }
  },
  methods: {
    selectRegionOption (option) {
      this.$emit('select', option)
    }
  }
}
</script>

<style scoped lang="scss">
// inherits style from v-branch-select
</style>
