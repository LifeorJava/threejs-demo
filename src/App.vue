<template>
  <div id="app" style="height: 100%;width: 100%;">
    <TaskPanel />
  </div>
</template>

<script>
import TaskPanel from './components/TaskPanel.vue';

export default {
  components: {
    TaskPanel
  }
};
</script>

<style>
body { margin: 0; overflow: hidden; background-color: #ffffff; display: flex; justify-content: center; align-items: center; height: 100vh; }
canvas { display: block; }
.panel-container {
  position: relative;
  width: 70%; /* 调整容器宽度 */
  height: 100%; /* 调整容器高度 */
}
.panel {
  position: absolute;
  width: 300px; /* 调整卡片宽度 */
  height: 400px; /* 调整卡片高度 */
  background-color: rgba(19, 15, 243, 0.5);
  border-radius: 10px;
  box-shadow: 0 4px 8px rgba(255, 20, 51, 0.5);
  padding: 10px;
  overflow: hidden;
  text-align: center;
  font-family: Arial, sans-serif;
  color: rgb(79, 231, 241);
  font-size: 25px;
}
.big {
  width: 500px; /* 调整卡片宽度 */
  height: 650px; /* 调整卡片高度 */
  font-size: 35px;
}

.panel p {
  margin: 10px 0;
}
.panel.flipped {
  transform: rotateY(180deg);
}
.close-btn {
  position: absolute;
  top: 10px;
  right: 10px;
  cursor: pointer;
  color: red;
  font-size: 16px;
}
</style>