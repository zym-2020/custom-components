<template>
  <div class="file-upload">
    <el-upload
      ref="upload"
      action="#"
      :auto-upload="false"
      :limit="1"
      :on-exceed="handleExceed"
      :on-change="changeHandle"
      :on-remove="removeHandle"
    >
      <template #trigger>
        <el-button type="primary" class="select-btn">选择文件</el-button>
      </template>
      <el-button type="success" @click="submitUpload"> 上传文件 </el-button>
    </el-upload>
    <el-progress
      :percentage="percentage"
      :status="status"
      v-if="progressFlag"
    />
  </div>
</template>

<script lang="ts">
/**
 * 单文件上传，非分片上传
 * @h：组件height
 * @w：组件width
 * @url：上传地址
 * @param：针对后端@RequestParam MultipartFile接收参数的参数名，不知道怎么把这个参数名忽略掉
 */
import { defineComponent, computed, ref } from "vue";
import { UploadInstance, UploadRawFile, UploadFile } from "element-plus";
import axios, { CancelTokenSource } from "axios";
export default defineComponent({
  props: {
    h: {
      type: String,
    },
    w: {
      type: String,
    },
    url: {
      type: String,
    },
    param: {
      type: String,
    },
  },
  emits: ["returnResult", "returnErr", "returnCancel"],
  setup(props, context) {
    const upload = ref<UploadInstance>();
    const progressFlag = ref(false);
    const percentage = ref(0);
    const status = ref("");
    const height = computed(() => {
      if (props.h === undefined) return "100%";
      return props.h;
    });
    const width = computed(() => {
      if (props.w === undefined) return "100%";
      return props.w;
    });

    let file: File | undefined = undefined;
    let uploadFlag = false;
    const changeHandle = (uploadFile: UploadFile) => {
      if (uploadFile.status === "ready" && uploadFile.raw) {
        file = uploadFile.raw;
      }
    };
    const handleExceed = (files: File[]) => {
      upload.value?.clearFiles();
      upload.value?.handleStart(files[0] as UploadRawFile);
    };
    const removeHandle = () => {
      if (uploadFlag && source) {
        source.cancel("取消上传");
      }
      progressFlag.value = false;
      status.value = "";
      percentage.value = 0;
      file = undefined;
    };

    const CancelToken = axios.CancelToken;
    let source: CancelTokenSource | undefined = undefined;
    const submitUpload = async () => {
      progressFlag.value = true;
      if (props.url && props.param && file !== undefined) {
        const formData = new FormData();
        formData.append(props.param, file);
        uploadFlag = true;
        source = CancelToken.source();
        axios
          .post(props.url, formData, {
            onUploadProgress: (e) => {
              if (e.progress) {
                percentage.value = Math.floor(e.progress * 100);
              }
            },
            cancelToken: source.token,
          })
          .then((data) => {
            if (data.status === 200) status.value = "success";
            context.emit("returnResult", data);
          })
          .catch((thrown) => {
            if (axios.isCancel(thrown)) {
              context.emit("returnCancel");
            } else {
              status.value = "exception";
              percentage.value = 33;
              context.emit("returnErr");
            }
          })
          .finally(() => {
            uploadFlag = false;
          });
      }
    };

    return {
      progressFlag,
      height,
      width,
      percentage,
      status,
      submitUpload,
      handleExceed,
      changeHandle,
      removeHandle,
      upload,
    };
  },
});
</script>

<style lang="scss" scoped>
.file-upload {
  height: v-bind(height);
  width: v-bind(width);
  .el-upload {
    .select-btn {
      margin-right: 20px;
    }
    /deep/ .el-upload-list__item-info {
      /*让长段文本不换行*/
      white-space: nowrap;
      /*设置文本超出元素宽度部分隐藏*/
      overflow-x: hidden;
      /*设置文本超出部分用省略号显示*/
      text-overflow: ellipsis;
    }
  }
}
</style>

