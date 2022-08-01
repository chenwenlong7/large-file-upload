<template>
  <div class="file-upload-fragment">
    <div class="file-upload-fragment-container">
      <el-upload
        class="fufc-upload"
        action=""
        :on-change="handleFileChange"
        :auto-upload="false"
        :show-file-list="false"
      >
        <template #trigger>
          <el-button class="fufc-upload-file" size="small" type="primary">
            选择文件
          </el-button>
        </template>
        <el-button
          class="fufc-upload-server"
          size="small"
          type="success"
          @click="handleUploadFile"
        >
          上传到服务器
        </el-button>
        <el-button
          class="fufc-upload-stop"
          size="small"
          type="primary"
          @click="stopUpload"
        >
          暂停上传
        </el-button>
        <el-button
          class="fufc-upload-continue"
          size="small"
          type="success"
          @click="continueUpload"
          >继续上传</el-button
        >
      </el-upload>
      <el-progress :percentage="percentage" color="#409eff" />
    </div>
  </div>
</template>
<script setup>
// import { ref } from 'vue'
// let percentage = ref(0)
// /**
//  * @description: 文件上传 Change 事件
//  * @param {*}
//  * @return {*}
//  */
// const handleFileChange = async (file) => {
// }
// /**
//  * @description: 文件上传 Click 事件
//  * @param {*}
//  * @return {*}
//  */
// const handleUploadFile = async () => {
// }
// /**
//  * @description: 暂停上传 Click 事件
//  * @param {*}
//  * @return {*}
//  */
// const stopUpload = () => {
// }
// /**
//  * @description: 继续上传 Click 事件
//  * @param {*}
//  * @return {*}
//  */
// const continueUpload = () => {
// }
</script>

<style scoped lang="scss">
.file-upload-fragment {
  width: 100%;
  height: 100%;
  padding: 10px;
  &-container {
    position: relative;
    margin: 0 auto;
    width: 600px;
    height: 300px;
    top: calc(50% - 150px);
    min-width: 400px;
    .fufc-upload {
      display: flex;
      justify-content: space-between;
      align-items: center;
    }
    .el-progress {
      margin-top: 10px;
      ::v-deep(.el-progress__text) {
        min-width: 0px;
      }
    }
  }
}
</style>
