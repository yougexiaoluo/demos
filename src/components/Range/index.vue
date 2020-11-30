<template>
  <div class="slider_container">
    <!-- 
      1. 主容器
      2. 滑块轨道
      3. 滑块
      4. 数据展示（滑块顶部/最右侧）
     -->
    <!-- 1. 主容器 -->
    <div
      class="slider_item"
      :style="{ background: bgc, height: height + 'rpx' }"
      ref="sliderItemRef"
      v-for="(item, idx) in list"
      :key="idx"
    >
      <!-- 2. 滑块轨道 -->
      <div class="track" :style="{ width: item.value + '%' }"></div>
      <!-- 3. 滑块 -->
      <div
        class="slider"
        :style="{ left: item.x2 - 5 + 'px' }"
        @touchstart="handleStart($event, idx)"
        @touchmove="handleMove($event, idx)"
        @touchend="handleEnd($event, idx)"
      >
        <!-- 4. 数据展示 -->
        <div class="dt_wrap">{{ item.value }}</div>
      </div>
    </div>
  </div>
</template>
<script>
export default {
  name: 'Range',
  props: {
    //
    data: {
      type: Array,
      default: () => {
        /* [{
          name: 'data1',
          value: 10,
          disabled: false
        }] */
        return [];
      },
    },
    // 主容器背景色
    bgc: {
      type: String,
      default: '#cccccc',
    },
    height: {
      type: Number,
      default: 10,
    },
    // 滑块轨道背景色
    activeBgc: String,
    maxValue: {
      type: Number,
      default: 100,
    },
  },
  data() {
    return {
      minW: 0,
      list: [],
      prevX2: 0,
      touchStart: 0,
    };
  },
  computed: {
    // 获取容器最大宽度
    maxW() {
      let result = 0;
      uni
        .createSelectorQuery()
        .select('.slider_item')
        .boundingClientRect((res) => {
          result = res.width;
        })
        .exec();

      return result;
    },
  },
  watch: {
    data: {
      handler(n) {
        this.list = n.map((item) => {
          item = {
            x1: 0,
            x2: 0,
            value: 0,
            prevX2: 0,
            ...item,
          };
          return item;
        });
      },
      deep: true,
      immediate: true,
    },
  },
  created() {},
  methods: {
    handleStart(e, idx) {
      const cX = e.changedTouches[0].clientX;
      this.touchStart = cX;
      if (this.checkMaxValue()) {
        this.$set(this.list, idx, {
          ...this.list[idx],
          x1: cX,
          prevX2: this.list[idx].x2,
        });
      }
    },
    handleMove(e, idx) {
      if (this.checkDirection(e)) {
        const item = this.list[idx];
        const cX = e.changedTouches[0].clientX;
        item.x2 = item.prevX2 ? cX + item.prevX2 - item.x1 : cX - item.x1;
        item.value = Math.floor((item.x2 / this.maxW) * 100);
        if (item.x2 >= this.maxW) {
          item.x2 = this.maxW;
          item.value = 100;
        } else if (item.x2 <= this.minW) {
          item.x2 = this.minW;
          item.value = this.minW;
        }
        this.$set(this.list, idx, item);
      }
    },
    handleEnd(e, idx) {
      this.touchStart = 0;
    },
    // 查询是否 >= maxValue
    checkMaxValue() {
      let total = this.list.reduce((prev, cur) => {
        prev += cur.value;
        return prev;
      }, 0);

      if (total >= this.maxValue) {
        this.$emit('arrive-target-value', this.list);
        return;
      }
      return true;
    },
    // 判断滑动方向
    checkDirection(e) {
      let distanceX = e.changedTouches[0].clientX - this.touchStart;
      if (distanceX > 0) {
        // 右滑
        return this.checkMaxValue();
      } else {
        // 左滑
        return this.checkMaxValue() ? true : !this.checkMaxValue();
      }
    },
  },
};
</script>
<style lang="scss" scoped>
.slider_container {
  width: 100%;
  .slider_item {
    width: 100%;
    border-radius: 5rpx;
    position: relative;
    margin: 60px 0;
    .track {
      width: 0%;
      height: 100%;
      background: #91d5ff;
      border-radius: 5rpx;
    }
    .slider {
      width: 20px;
      height: 20px;
      border-radius: 10px;
      border: 2px solid #1890ff;
      background: #fff;
      box-shadow: 0 0 5px 0px #f5f5f5;
      position: absolute;
      top: 50%;
      left: -10px;
      transform: translateY(-10px);
      cursor: pointer;
      .dt_wrap {
        position: absolute;
        top: -200%;
        left: 50%;
        transform: translate(-50%);
        background: rgba(0, 0, 0, 0.8);
        color: #fff;
        font-size: 12px;
        border-radius: 3px;
        padding: 8rpx 14rpx;
        &::before {
          content: '';
          width: 0;
          height: 0;
          position: absolute;
          bottom: -60%;
          left: 50%;
          transform: translate(-50%, -50%);
          border: 5px solid transparent;
          border-top-color: rgba(0, 0, 0, 0.8);
        }
      }
    }
  }
}
</style>
