<template lang="pug">
  .operate-wrapper
    .like
      img(src="../../../assets/way/xin.png")
      span {{countItem.likeCount}}
    .comment
      img(src="../../../assets/way/common.png")
      span {{countItem.commentsCount}}
    .on-the-way
      img(src="../../../assets/way/byway.png")
      span 顺路么
</template>

<script>

export default {
  name: 'Operate',
  props: {
    name: {
      type: String,
      default: '认识顺路的人'
    },
    countItem: {
      type: Object,
      require: true
    }
  },
  methods: {
    openApp () {
      window.location.href = 'appPkgUrl'
    }
  }
}
</script>

<style lang="less" scoped>
.operate-wrapper {
  box-sizing: border-box;
  height: 40px;
  padding: 0 28px;
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: space-between;
  .like, .comment, .on-the-way {
    display: flex;
    justify-content: center;
    align-items: center;
    color: #808080;
    font-size: 12px;
    img{
      width: 15px;
      height: 15px;
      margin-right: 4px;
    }
  }
}
.app-bottom-btn {
  border-top: 1px solid #f6f6f6;
  position: fixed;
  bottom: 0;
  left: 0;
  height: 67px;
  width: 100%;
  background: #FFFFFF;
  display: flex;
  justify-content: center;
  align-items: center;
  .btn-wrap {
    width: 167px;
    height: 44px;
    background: #00c584;
    border-radius: 22px;
    font-size: 16px;
    font-weight: bolder;
    display: flex;
    align-items: center;
    justify-content: center;
    color: #ffffff;
  }
}

</style>
