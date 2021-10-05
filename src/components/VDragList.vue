<template>
  <transition-group
    ref="list"
    name="list"
    tag="ul"
    class="list"
  >
    <li
      v-for="item of items"
      :ref="`item${item.id}`"
      :key="item.id"
      class="list__item"
      :list-rect="clientRect"
      :class="{ 'list__item_dragged': isOnDrag && item === draggedElement }"
      @mousedown="onStartDrag($event, item)"
    >
      <span>{{ item.title }}</span>
      <h2 v-if="item.id === 2">{{ item.title }}</h2>
    </li>
    <li
      v-if="draggedElement"
      :key="`${draggedElement.id}-copy`"
      class="list__item list__item_copy"
      :style="itemPosition"
      @mouseup="onEndDrag"
    >
      <span>{{ draggedElement.title }}</span>
      <h2 v-if="draggedElement.id === 2">{{ draggedElement.title }}</h2>
    </li>
  </transition-group>
</template>

<script>
export default {
  name: 'VDragList',
  data() {
    return {
      items: [
        { title: '1', id: 1 },
        { title: '2', id: 2 },
        { title: '3', id: 3 },
      ],
      clientRect: null,
      isOnDrag: false,
      position: {
        x: 0,
        y: 0,
        offsetY: 0,
      },
      foundPosition: {
        x: 0,
        y: 0,
      },
      draggedElement: null,
    };
  },
  computed: {
    itemPosition() {
      return {
        top: `${this.position.y}px`,
        transition: 'none',
      };
    },
  },
  methods: {
    changePosition(ev) {
      const { position, foundPosition } = this;
      // eslint-disable-next-line no-multi-assign
      foundPosition.x = position.x = ev.pageX;
      position.offsetY = ev.offsetY;
      // eslint-disable-next-line no-multi-assign
      foundPosition.y = position.y = ev.pageY - ev.offsetY;
    },

    onStartDrag(ev, item) {
      this.changePosition(ev);
      this.draggedElement = item;
      this.isOnDrag = true;
    },

    onEndDrag() {
      if (this.isOnDrag) {
        this.isOnDrag = false;
        this.position.y = this.foundPosition.y;
        setTimeout(() => {
          this.draggedElement = null;
        }, 200);
      }
    },

    switch() {
      let heightSum = 0;
      let id = null;
      // eslint-disable-next-line no-restricted-syntax
      for (const item of this.items) {
        const currentHeight = this.$refs[`item${item.id}`][0].clientHeight;
        id = item.id;
        if (heightSum + currentHeight < this.position.y) {
          heightSum += currentHeight;
          id = item.id;
        } else {
          this.switchElements(id);
          return;
        }
      }
      if (id) {
        this.switchElements(id);
      }
    },

    switchElements(id) {
      if (this.draggedElement.id === id) return;
      const draggedIdx = this.items.findIndex((item) => item.id === this.draggedElement.id);
      const switchedIdx = this.items.findIndex((item) => item.id === id);
      this.foundPosition.y = this.$refs[`item${id}`][0].getBoundingClientRect().top;
      const el = this.items[draggedIdx];
      this.items[draggedIdx] = this.items[switchedIdx];
      this.items[switchedIdx] = el;
    },

    onMoveItem(ev) {
      if (this.isOnDrag) {
        this.position.y = ev.y - this.position.offsetY;
        this.switch();
      }
    },
  },
  mounted() {
    this.clientRect = this.$refs.list.$el.getBoundingClientRect();
    window.addEventListener('mousemove', this.onMoveItem.bind(this));
  },
};
</script>

<style lang="scss" scoped>
  .list {
    display: flex;
    flex-direction: column;
    list-style: none;
    padding: 0;
    & > .list__item {
      margin-bottom: 10px;
    }

    &__item {
      padding: 15px;
      border-radius: 5px;
      background-color: #333;
      color: #fffff0;
      font-size: 1rem;
      transition: all 0.3s;
      user-select: none;

      &_dragged {
        opacity: 0;
        transition: none;
      }

      &_copy {
        position: fixed;
        z-index: 1000;
        width: 100%;
        cursor: move;
        transition: none;
        outline:  1px solid aqua;
      }
    }
  }
  .list-enter, .list-leave-to {
    transition: all 0.3s;
  }

  .list-leave-active {
    position: absolute;
  }
</style>
