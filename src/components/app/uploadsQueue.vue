<template>
  <div class="uploads-queue">
    <div v-for="(upload, index) in $props.queue" :key="index" class="upload">
      <div class="icon">
        <i class="material-icons">insert_drive_file</i>
      </div>
      <div class="information">
        <div class="info">{{ upload.name }}</div>
        <div class="info size">{{ upload.size }}</div>
        <div class="progress">
          <div class="progress-bar">
            <div
              class="progress-bar-inner"
              :style="{ width: `${upload.percent}%` }"
            />
          </div>
          <div class="percent">{{ upload.percent }}%</div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: ["queue"]
};
</script>

<style scoped>
.uploads-queue {
  display: inline-block;
}
.upload {
  color: white;
  background: #26778a;
  margin: 10px;
  margin-left: 10px;
  display: flex;
  padding: 10px;
}
.icon {
  display: flex;
  justify-content: center;
  align-items: center;
  margin-right: 5px;
  flex-shrink: 0;
}
.icon .material-icons {
  font-size: 40px;
}
.info {
  word-wrap: break-word;
  word-break: break-word;
  white-space: pre-wrap;
  font-size: 14px;
  overflow: hidden;
  max-width: 100%;
  color: white;
  overflow-wrap: anywhere;
  margin-top: 3px;
}
.size {
  font-size: 15px;
  color: #d5e3e6;
}
.progress {
  display: flex;
  align-content: center;
  align-items: center;
}
.progress-bar {
  width: 100px;
  height: 10px;
  background: rgba(0, 0, 0, 0.513);
  margin-right: 10px;
}
.progress-bar-inner {
  height: 10px;
  background: rgb(255, 255, 255);
  animation: dimProgress;
  animation-duration: 2s;
  animation-iteration-count: infinite;
  animation-direction: normal;
  animation-fill-mode: forwards;
}
@keyframes dimProgress {
  0% {
    background: rgb(255, 255, 255);
  }
  50% {
    background: rgba(253, 253, 253, 0.705);
  }
  100% {
    background: rgb(255, 255, 255);
  }
}
</style>
