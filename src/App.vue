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
import { postUploadFile } from "@/api/api.js";
import { ElMessage } from "element-plus";
import { ref } from "vue";
import SparkMD5 from "spark-md5";
let percentage = ref(0);

const chunkSize = 5 * 1024 * 1024;

/**
 * @description: 创建文件分片
 * @param {*}
 * @return {*}
 */
const createChunkList = (file, chunkSize) => {
  const fileChunkList = [];
  let cur = 0;
  while (cur < file.size) {
    fileChunkList.push(file.slice(cur, cur + chunkSize));
    cur += chunkSize;
  }
  return fileChunkList;
};
/**
 * @description: 生成文件Hash
 * @param {*}
 * @return {*}
 */
const createMD5 = (fileChunkList) => {
  return new Promise((resolve, reject) => {
    // const slice =
    //   File.prototype.slice ||
    //   File.prototype.mozSlice ||
    //   File.prototype.webkitSlice;
    const chunks = fileChunkList.length;
    let currentChunk = 0;
    let spark = new SparkMD5.ArrayBuffer();
    let fileReader = new FileReader();
    // 读取文件切片
    fileReader.onload = function (e) {
      spark.append(e.target.result);
      currentChunk++;
      if (currentChunk < chunks) {
        loadChunk();
      } else {
        // 读取切片，返回最终文件的Hash值
        resolve(spark.end());
      }
    };

    fileReader.onerror = function (e) {
      reject(e);
    };

    function loadChunk() {
      fileReader.readAsArrayBuffer(fileChunkList[currentChunk]);
    }

    loadChunk();
  });
};
/**
 * @description: 文件上传 Change 事件
 * @param {*}
 * @return {*}
 */
let currentFile = ref(null);

const handleFileChange = async (file) => {
  if (!file) return;
  currentFile.value = file;
};
// /**
//  * @description: 文件上传 Click 事件
//  * @param {*}
//  * @return {*}
//  */
let chunkFormData = ref([]);
let fileHash = ref(null);
const handleUploadFile = async () => {
  if (!currentFile.value) {
    ElMessage.warning("请选择文件");
    return;
  }
  // 文件分片
  let fileChunkList = createChunkList(currentFile.value.raw, chunkSize);
  fileHash.value = await createMD5(currentFile.value.raw, chunkSize);

  let chunkList = fileChunkList.map((file, index) => {
    return {
      file: file,
      chunkHash: fileHash.value + "-" + index,
      fileHash: fileHash.value,
    };
  });
  chunkFormData.value = chunkList.map((chunk) => {
    let formData = new FormData();
    formData.append("chunk", chunk.file);
    formData.append("chunkHash", chunk.chunkHash);
    formData.append("fileHash", chunk.fileHash);
    return {
      formData: formData,
    };
  });

  Promise.all(
    chunkFormData.value.map((data) => {
      return new Promise((resolve, reject) => {
        postUploadFile(data.formData)
          .then((data) => {
            resolve(data);
          })
          .catch((err) => {
            reject(err);
          });
      });
    })
  );
};
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
