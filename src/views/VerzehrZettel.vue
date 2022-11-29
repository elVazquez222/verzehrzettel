<template>
  <div class="home">
    <img alt="T&P" class="noPrint" src="../assets/logo.png" />
    <VerzehrZettelForm msg="Welcome to Your Vue.js + TypeScript App" />
  </div>
</template>

<script lang="ts">
import { defineComponent } from "vue";
import VerzehrZettelForm from "@/components/VerzehrZettelForm.vue"; // @ is an alias to /src

export default defineComponent({
  name: "VerzehrZettel",
  components: {
    VerzehrZettelForm,
  },
});
</script>


