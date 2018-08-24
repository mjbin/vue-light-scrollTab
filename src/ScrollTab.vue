<template>
  <div class="scroll-tab">
    <ul class="tab-list" ref="tabs" :style='containStyle'>
      <li class="tab-item" :ref="'tab_' + i" v-for="(item, i) in tabs" :key='i' @click="actionTab(item, i)" :class="{active: value.key === item.key}" :style="[defaultTabStyle, value.key === item.key && tabStyle]">{{item.label}}</li>
      <li class="line" :style="lineStyle" v-if="value.key || value.key === 0"></li>
    </ul>
  </div>
</template>
<script>
export default {
  props: {
    value: {
      type: Object,
      default: {
        key: 0,
      },
    },
    tabs: Array,
    color: {
      type: String,
      default: '#f53b10',
    },
    bgColor: {
      type: String,
      default: '#fff',
    },
    tabsStyle: {
      type: Object,
      default: () => ({}),
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
        background: '#f53b10',
        left: 0,
      },
      duration: 0,
      animation: null,
    };
  },
  computed: {
    containStyle() {
      let defaultStyle = {
        background: this.bgColor,
      };
      if (Object.keys(this.tabsStyle).length > 0) {
        defaultStyle = { ...defaultStyle, ...this.tabsStyle };
      }
      return defaultStyle;
    },
    defaultTabStyle() {
      const sty = {};
      const len = this.tabs.length;
      if (len <= 4) {
        sty.flex = '1 auto';
        sty.whiteSpace = 'nowrap';
      }
      return sty;
    },
  },
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
      const tabsMiddle = document.body.offsetWidth / 2; // 容器中间
      const tabStart = (currentIndexTab.offsetLeft - tabs.scrollLeft)
        + (currentIndexTab.offsetWidth / 2);

      this.duration = (tabStart - tabsMiddle) / scrollDuration;
      const step = () => {
        if (this.animation) {
          window.cancelAnimationFrame(this.animation);
        }

        tabs.scrollLeft += this.duration;
        this.currentOffset = tabs.scrollLeft;
        count += 1;
        if (count < scrollDuration) {
          this.animation = window.requestAnimationFrame(step);
        } else {
          this.ready = true;
        }
      };
      this.animation = window.requestAnimationFrame(step);
      this.lineSlider(); // 下滑线动效
    },
    lineSlider() {
      this.lineStyle.width = `${this.refCurrentTab.offsetWidth}px`;
      this.lineStyle.left = `${((this.refCurrentTab.offsetLeft)
          / document.body.offsetWidth).toFixed(2) * 100}%`;
    },
    emitValue(value) {
      this.$emit('input', value);
    },
    nextTickCb(list) {
      this.$nextTick(() => {
        if (this.value.key || this.value.key === 0) {
          const tab = list[this.value.key];
          if (tab) this.current = tab;
        }
      });
    },
  },
  mounted() {
    this.nextTickCb(this.tabs);
    this.refTabs = this.$refs.tabs;
    this.tabStyle.color = this.color || '#f23d22';
    this.lineStyle.background = this.color || '#f23d22';
    for (let i = 0; i < this.refTabs.childNodes.length; i += 1) {
      const item = this.refTabs.childNodes[i];
      if (item.className === 'tab-item') {
        this.refTabsWidth += item.clientWidth;
      }
    }
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
        if (findIndex !== -1 && this.$refs[`tab_${findIndex}`]) {
          const tab = this.$refs[`tab_${findIndex}`][0];
          this.refCurrentTab = tab;
        }
      }
      if (this.refCurrentTab) {
        this.ready = true;
        this.scrollToActiveTab();
      }
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
    -webkit-overflow-scrolling: touch;
    -webkit-scroll-snap-type: mandatory;
    .tab-item {
      list-style-type: none;
      flex-shrink: 0;
      text-align: center;
      padding: 0 0.1rem;
      margin: 0 0.2rem;
      &.active {
        color: #f53b10;
      }
    }
    .line {
      position: absolute;
      content: '';
      left: 0;
      bottom: 0;
      height: 3px;
      width: 50px;
      transition: all 0.3s;
      color: #f53b10;
    }
  }
}
</style>
