<template>
  <view class="content">
    <range :data="[{}, {}, {}]"></range>
  </view>
</template>

<script>
export default {
  data() {
    return {};
  },
  onLoad() {},
  methods: {},
  components: {
    Range: () => import("@c/Range")
  }
};
</script>

<style lang="scss" scoped>
.content {
	padding: 50px 50px;
}
</style>
