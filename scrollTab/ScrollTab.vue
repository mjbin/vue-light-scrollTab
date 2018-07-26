<template>
  <div class="scroll-tab">
    <ul class="tab-list" ref="tabs">
      <li class="tab-item" :ref="'tab_' + i" v-for="(item, i) in tabs" :key='i' @click="actionTab(item, i)" :class="{active: value.key === item.key}" :style="value.key === item.key && tabStyle">{{item.label}}</li>
      <li class="line" :style="lineStyle" v-if="value.key || value.key === 0"></li>
    </ul>
  </div>
</template>
<script>
export default {
  props: {
    value: {
      type: Object,
      default: null,
    },
    tabs: Array,
    color: {
      type: String,
      default: 'red',
    },
  },
  data() {
    return {
      ready: true, // 待动画完成可生效
      current: null,
      currentOffset: 0,
      refTabs: null,
      refCurrentTab: null,
      refTabsWidth: 0,
      tabStyle: {
        color: '',
      },
      lineStyle: {
        width: 0,
        background: 'red',
        left: 0,
      },
    };
  },
  computed: {},
  methods: {
    actionTab(item, i) {
      if (this.ready) {
        this.refCurrentTab = this.$refs[`tab_${i}`][0]; // eslint-disable-line
        this.$emit('input', item);
        this.$emit('onAction', item);
      }
    },
    scrollToActiveTab() {
      if (!this.ready) {
        return;
      }
      this.ready = false;
      const currentIndexTab = this.refCurrentTab;
      let count = 0;
      const scrollDuration = 15;
      const tabs = this.refTabs;
      const tabsMiddle = tabs.offsetWidth / 2;
      const tabStart = (currentIndexTab.offsetLeft - tabs.scrollLeft)
        + (currentIndexTab.offsetWidth / 2);
      const step = () => {
        const duration = (tabStart - tabsMiddle) / scrollDuration;
        tabs.scrollLeft += duration;
        this.currentOffset = tabs.scrollLeft;
        count += 1;
        if (count < scrollDuration) {
          window.requestAnimationFrame(step);
        } else {
          this.ready = true;
        }
      };
      window.requestAnimationFrame(step);
      this.lineSlider(); // 下滑线动效
    },
    lineSlider() {
      this.lineStyle.width = `${this.refCurrentTab.offsetWidth}px`;
      this.lineStyle.left = `${((this.refCurrentTab.offsetLeft)
          / this.refTabs.offsetWidth).toFixed(2) * 100}%`;
    },
    emitValue(value) {
      this.$emit('input', value);
    },
    nextTickCb(list) {
      this.$nextTick(() => {
        if (this.value.key || this.value.key === 0) {
          const tab = list[this.value.key];
          if (tab) this.current = tab;
          // if (tab) this.$emit('input', tab);
        }
      });
    },
  },
  mounted() {
    this.nextTickCb(this.tabs);
    this.refTabs = this.$refs.tabs;
    this.tabStyle.color = this.color;
    this.lineStyle.background = this.color;
    this.refTabs.childNodes.forEach((i) => {
      if (i.className === 'tab-item') {
        this.refTabsWidth += i.clientWidth;
      }
    });
  },
  watch: {
    tabs(val) {
      if (val.length > 0) {
        this.nextTickCb(val);
      }
    },
    value(val) {
      this.current = val;
    },
    current(val) {
      if (val.key || val.key === 0) {
        const findIndex = this.tabs.findIndex(i => i.key === val.key);
        this.refCurrentTab = findIndex !== -1 && this.$refs[`tab_${findIndex}`][0]; // eslint-disable-line
      }
      this.scrollToActiveTab();
    },
  },
};
</script>

<style lang="less" scoped>
.scroll-tab {
  background: #fff;
  width: 100%;
  overflow: hidden;
  position: relative;
  .tab-list {
    display: flex;
    margin: 0;
    padding: 0;
    width: 100%;
    height: 1rem;
    line-height: 1rem;
    line-height: 1rem;
    position: relative;
    overflow: hidden;
    overflow-x: scroll;
    .tab-item {
      list-style-type: none;
      background: #fff;
      flex-shrink: 0;
      justify-content: center;
      align-items: center;
      text-align: center;
      padding: 0 0.5rem;
      &.active {
        color: red;
      }
    }
    .line {
      position: absolute;
      content: '';
      left: 0;
      bottom: 0;
      height: 3px;
      width: 50px;
      background: #000;
      transition: all 0.3s;
    }
  }
}
</style>