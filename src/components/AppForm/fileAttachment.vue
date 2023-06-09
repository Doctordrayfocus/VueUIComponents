<template>
  <div class="flex w-full flex-col space-y-2">
    <template v-if="isWrapper">
      <div class="flex flex-row relative">
        <input
          type="file"
          style="
            opacity: 0;
            width: 100%;
            height: 100%;
            left: 0;
            overflow: hidden;
            position: absolute;
            z-index: 10;
          "
          :accept="accept"
          :multiple="isMultiple"
          @change="uploadHandler"
        />
        <slot name="content" />
      </div>
    </template>
    <template v-else>
      <div
        class="rounded flex flex-row items-center justify-start relative space-x-2 px-1 py-4 bg-grayBackground border-dashed"
      >
        <input
          type="file"
          style="
            opacity: 0;
            width: 100%;
            height: 100%;
            left: 0;
            overflow: hidden;
            position: absolute;
            z-index: 10;
          "
          :accept="accept"
          :multiple="isMultiple"
          @change="uploadHandler"
        />
        <app-icon :name="`${iconName}`" :customClass="'h-[15px]'" />
        <app-normal-text
          color="text-paragraphTextLight"
          customClass="w-full text-left line-clamp-1"
        >
          {{ selectedFileName != "" ? selectedFileName : placeholder }}
        </app-normal-text>
      </div>
    </template>
  </div>
</template>
<script lang="ts">
import { defineComponent, onMounted, ref, watch } from "vue";
import AppNormalText from "../AppTypography/normalText.vue";
import AppIcon from "../AppIcon/index.vue";

export default defineComponent({
  components: {
    AppNormalText,
    AppIcon,
  },
  props: {
    placeholder: {
      type: String,
      default: "Upload File",
    },
    iconName: {
      type: String,
      default: "upload",
    },
    accept: {
      type: String,
      default: "*",
    },
    modelValue: {
      required: false,
    },
    isWrapper: {
      type: Boolean,
      default: false,
    },
    isMultiple: {
      type: Boolean,
      default: false,
    },
  },
  emits: ["update:modelValue", "update:localFileUrl", "update:base64Data"],
  name: "AppFileAttachment",
  setup(props: any, context: any) {
    const files = ref<FileList>();

    const selectedFileName = ref("");

    const fileListArray = ref<any[]>([]);

    const toDataURL = (url: string, callback: any) => {
      var xhr = new XMLHttpRequest();
      xhr.onload = function () {
        var reader = new FileReader();
        reader.onloadend = function () {
          callback(reader.result);
        };
        reader.readAsDataURL(xhr.response);
      };
      xhr.open("GET", url);
      xhr.responseType = "blob";
      xhr.send();
    };

    const uploadHandler = (e: any) => {
      const input = e.target;

      if (props.isMultiple) {
        files.value = input.files;
      } else {
        files.value = input.files;

        selectedFileName.value = input.files[0].name;

        // create readable url
        const fr = new FileReader();
        if (files.value) {
          fr.readAsDataURL(files.value[0]);
          fr.addEventListener("load", () => {
            context.emit(
              "update:localFileUrl",
              fr.result?.toString() ? fr.result?.toString() : ""
            );
            toDataURL(fr.result?.toString() || "", (dataUrl: any) => {
              context.emit("update:base64Data", dataUrl);
            });
          });
        }
      }
    };

    watch(files, () => {
      if (files.value) {
        fileListArray.value = [];
        for (let index = 0; index < files.value.length; index++) {
          const file = files.value?.item(index);
          fileListArray.value.push(file);
        }
        context.emit(
          "update:modelValue",
          props.isMultiple ? fileListArray.value : fileListArray.value[0]
        );
      }
    });

    watch(props, () => {
      if (props.placeholder) {
        selectedFileName.value = "";
      }
    });

    onMounted(() => {
      if (props.modelValue) {
        fileListArray.value = props.modelValue;
      }
    });

    const removeFile = (index: number) => {
      fileListArray.value = fileListArray.value.filter((file, fileIndex) => {
        return fileIndex != index;
      });
      context.emit(
        "update:modelValue",
        props.isMultiple ? fileListArray.value : fileListArray.value[0]
      );
    };

    return {
      uploadHandler,
      fileListArray,
      removeFile,
      selectedFileName,
    };
  },
});
</script>
