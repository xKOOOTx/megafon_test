<template>
  <div class="u-select" @click="areOptionsVisible = !areOptionsVisible" :branchSelected="branchSelected">
    <p class="u-select__name">{{ branchSelected }}</p>
    <div
      class="u-select__options"
      v-if="areOptionsVisible"
    >
      <p
        class="u-select__options_name"
        v-for="option in options[0]"
        :key="option.value"
        @click="selectBranchOption(option)"
      >
        {{ option.branchName }}
      </p>
    </div>
  </div>
</template>

<script>
export default {
  name: 'v-branch-select',
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
    branchSelected: {
      type: String,
      default: ''
    }
  },
  methods: {
    selectBranchOption (option) {
      this.$emit('select', option)
    }
  }
}
</script>

<style lang="scss">
.u-select__name,
.u-select__options_name,
.submitBtn {
  width: 300px;
  padding: 10px 0;
  font-size: 24px;
  font-weight: 700;
  cursor: pointer;
  color: white;

}
.u-select__name,
.u-select__options_name {
  background: #5B9BD5;
  border: none;
}
.u-select {
  position: relative;
  user-select: none;
  &:not(:last-child) {
    margin-right: 20px;
  }
  &__options {
    position: absolute;
    top: 50px;
    left: 0;
    width: 300px;
    &_name {
      border: 1px solid black;
      transition: background-color .2s ease-in-out;
      &:not(:last-child) {
        border-bottom: none;
      }
      &:hover {
        background: #009cd0;
      }
    }
  }
}
</style>
