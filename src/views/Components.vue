<template>
  <div>
    <br />
    <avatar-upload
      :h="'100px'"
      :w="'100px'"
      :fontSize="'28px'"
      :imageUrl="''"
      @returnPictureFile="returnPictureFile"
    />
    <br />
    <file-upload
      :url="'http://localhost:8002/multiSource/uploadFile'"
      :param="'multipartFile'"
      :w="'400px'"
      @returnResult="returnResult"
      @returnErr="returnErr"
      @returnCancel="returnCancel"
    />
  </div>
</template>

<script lang="ts">
import { defineComponent } from "vue";
import AvatarUpload from "@/components/AvatarUpload.vue";
import FileUpload from "@/components/FileUpload.vue";
import { AxiosResponse } from "axios";
export default defineComponent({
  components: { AvatarUpload, FileUpload },
  setup() {
    const returnPictureFile = (val: File | undefined) => {
      console.log(val);
    };

    const returnResult = (val: AxiosResponse) => {
      console.log(val);
    };
    const returnErr = () => {
      console.log("出错了");
    };

    const returnCancel = () => {
      console.log("取消上传");
    };

    return {
      returnPictureFile,
      returnResult,
      returnErr,
      returnCancel,
    };
  },
});
</script>
