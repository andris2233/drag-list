<template>
  <li
    ref="item"
    class="list__item"
  >
    <slot name="item" :item="item" />
  </li>
</template>

<script>
export default {
  name: 'VDragListItem',
  props: {
    item: {
      type: Object,
      default: () => ({}),
    },
  },
  data() {
    return {
      width: 0,
    };
  },
  mounted() {
    this.width = this.$refs.item.clientWidth;
  },
};
</script>

<style lang="scss" scoped>
  .list
  }
</style>
