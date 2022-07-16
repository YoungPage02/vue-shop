<template>
  <div>
    <!-- 背景 -->
    <div class="bubble">
      <span style="--i:11"></span>
      <span style="--i:15"></span>
      <span style="--i:20"></span>
      <span style="--i:22"></span>
      <span style="--i:18"></span>
      <span style="--i:13"></span>
      <span style="--i:15"></span>
      <span style="--i:16"></span>
      <span style="--i:19"></span>
      <span style="--i:22"></span>
      <span style="--i:15"></span>
      <span style="--i:17"></span>
      <span style="--i:20"></span>
      <span style="--i:14"></span>
      <span style="--i:16"></span>
      <span style="--i:13"></span>
      <span style="--i:11"></span>
      <span style="--i:15"></span>
      <span style="--i:20"></span>
      <span style="--i:22"></span>
      <span style="--i:18"></span>
      <span style="--i:13"></span>
      <span style="--i:15"></span>
      <span style="--i:16"></span>
      <span style="--i:19"></span>
      <span style="--i:22"></span>
      <span style="--i:15"></span>
      <span style="--i:17"></span>
      <span style="--i:20"></span>
      <span style="--i:14"></span>
      <span style="--i:16"></span>
      <span style="--i:13"></span>
    </div>
    <h1>欢迎来到iStudy管理平台</h1>
  </div>
</template>

<script>
export default {

}
</script>

<style lang="less" scoped>
.bubble {
  position: relative;
  top: 0;
  left: 0;
  display: flex;
}
.bubble span {
  position: relative;
  width: 30px;
  height: 30px;
  background-color: #4fc3dc;
  margin: 0 4px;
  border-radius: 50%;
  box-shadow: 0 0 0 10px #4fc3dc44,0 0  50px #4fc3dc,0 0  100px #4fc3dc;
  animation: starAnimation 12s linear infinite;
  animation-duration: calc(100s / var(--i));
}
.bubble span:nth-child(even) {
  background-color: #ff2d75;
  box-shadow: 0 0 0 10px #ff2d7544,0 0  50px #ff2d75,0 0  100px #ff2d75;
}
@keyframes starAnimation {
  0% {
    transform: translateY(100vh) scale(0);
  }
  100% {
    transform: translateY(-10vh) scale(1);
  }
}
</style>
