<template>
  <div class="window-height window-width row justify-center items-center">
    <router-view />
  </div>
</template>

<script>
import { defineComponent } from "vue";

export default defineComponent({
  name: "MainLayout",
});
</script>
