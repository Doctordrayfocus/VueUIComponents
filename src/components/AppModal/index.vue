<template>
  <component :is="Teleport" to="body">
    <transition name="fade" appear>
      <div
        :class="`fixed top-0 w-screen h-screen bg-black !bg-opacity-40 flex flex-col overflow-y-hidden items-center mdlg:!justify-center justify-end ${customClass}`"
        style="z-index: 9999999999999999"
        @click="closeModal()"
      >
        <div
          class="relative w-full flex flex-col mdlg:!justify-center justify-end items-center mdlg:!h-[85%] h-full overflow-y-auto"
        >
          <div
            class="w-full flex flex-col h-full mdlg:!justify-center justify-end items-center"
          >
            <slot />
          </div>
        </div>
      </div>
    </transition>
  </component>
</template>
<script lang="ts">
import { Teleport as teleport_, TeleportProps, VNodeProps } from "vue";

const Teleport = teleport_ as {
  new (): {
    $props: VNodeProps & TeleportProps;
  };
};

export default {
  props: {
    canClose: {
      type: Boolean,
      default: true,
    },
    close: {
      type: Function,
      required: true,
    },
    customClass: {
      type: String,
      default: "",
    },
  },
  name: "AppModal",
  setup(props: any) {
    const closeModal = () => {
      if (props.canClose) {
        props.close();
      }
    };

    return {
      closeModal,
      Teleport,
    };
  },
};
</script>
