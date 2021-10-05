<template>
  <transition-group
    ref="list"
    name="list"
    tag="ul"
    class="list"
  >
    <li
      v-for="(item, index) of items"
      :ref="`item${item.id}`"
      :key="item.id"
      class="list__item"
      :list-rect="clientRect"
      :class="{ 'list__item_dragged': isOnDrag && item === draggedElement }"
      @mousedown.stop="onStartDrag($event, item, index)"
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
        offsetX: 0,
        width: 0,
      },
      draggedElement: null,
      isDownDirection: true,
      draggedIndex: -1,
      canDrag: true,
    };
  },
  computed: {
    itemPosition() {
      return {
        top: `${this.position.y}px`,
        left: `${this.position.x}px`,
        transition: this.isOnDrag ? 'none' : 'all 0.1s',
        width: `${this.position.width}px`,
      };
    },
  },
  methods: {
    changePosition(ev) {
      const { position } = this;
      const rect = ev.currentTarget.getBoundingClientRect();

      const offsetY = ev.clientY - rect.top;
      const offsetX = ev.clientX - rect.left;

      position.offsetY = offsetY;
      position.offsetX = offsetX;

      position.width = ev.currentTarget.clientWidth;

      position.x = ev.pageX - offsetX;
      position.y = ev.pageY - offsetY;
    },

    onStartDrag(ev, item, index) {
      if (!this.canDrag) return;
      this.changePosition(ev);
      this.draggedElement = item;
      this.isOnDrag = true;
      this.draggedIndex = index;
    },

    onEndDrag() {
      if (this.isOnDrag) {
        this.canDrag = false;
        this.isOnDrag = false;

        const rect = this.$refs[`item${this.draggedElement.id}`][0].getBoundingClientRect();
        this.position.y = rect.top;
        this.position.x = rect.left;

        setTimeout(() => {
          this.canDrag = true;
          this.draggedIndex = -1;
          this.draggedElement = null;
        }, 300);
      }
    },

    switch() {
      let heightSum = 0;

      if (this.isDownDirection) {
        if (this.draggedIndex === this.items.length - 1) return;

        for (let i = 0; i <= this.draggedIndex; i += 1) {
          heightSum += this.$refs[`item${this.items[i].id}`][0].clientHeight;
        }

        if (this.position.y > heightSum) {
          this.switchElements(this.items[this.draggedIndex + 1].id);
        }
      } else {
        if (this.draggedIndex === 0) return;

        for (let i = 0; i <= this.draggedIndex - 1; i += 1) {
          heightSum += this.$refs[`item${this.items[i].id}`][0].clientHeight;
        }

        if (this.position.y < heightSum) {
          this.switchElements(this.items[this.draggedIndex - 1].id);
        }
      }
    },

    switchElements(id) {
      if (this.draggedElement.id === id) return;
      const draggedIdx = this.items.findIndex((item) => item.id === this.draggedElement.id);
      const switchedIdx = this.items.findIndex((item) => item.id === id);
      const el = this.items[draggedIdx];
      this.items[draggedIdx] = this.items[switchedIdx];
      this.items[switchedIdx] = el;
      this.draggedIndex += draggedIdx < switchedIdx ? 1 : -1;
    },

    onMoveItem(ev) {
      if (this.isOnDrag) {
        const newVerticalPosition = ev.y - this.position.offsetY;
        this.isDownDirection = newVerticalPosition > this.position.y;
        this.position.y = newVerticalPosition;
        this.position.x = ev.x - this.position.offsetX;
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
    width: 300px;
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
        cursor: move;
        transition: none;
        outline: 1px solid aqua;
        box-sizing: border-box;
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
