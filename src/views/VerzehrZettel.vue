<template>
  <div class="verzehrZettel">
    <div class="header">
      <img alt="T&P" class="noPrint" src="../assets/logo.png" />
      <div>
        Verzehrzettel für <input class="dateInput" type="text" :value="getDate()" />
      </div>
    </div>
    <VerzehrZettelForm msg="Welcome to Your Vue.js + TypeScript App" />
  </div>
</template>

<script>
import { defineComponent } from "vue";
import VerzehrZettelForm from "@/components/VerzehrZettelForm.vue"; // @ is an alias to /src

export default defineComponent({
  name: "VerzehrZettel",
  components: {
    VerzehrZettelForm,
  },
  setup() {
    const getDate = () => {

      const currentDate = new Date();

      if(currentDate.getHours() < 8) {
        currentDate.setDate(currentDate.getDate() - 1);
      }

      const options = {
        weekday: "long",
        year: "numeric",
        month: "long",
        day: "numeric",
      };

      return currentDate.toLocaleString("de-DE", options);
    }
    return {getDate}
  }
});
</script>

<style scoped>
  .verzehrZettel {
    display: flex;
    flex-direction: column;
  }
  .header {
    display: flex;
    align-items: center;
    justify-content: space-between;
  }
  .dateInput {
    min-width: 220px;
  }


@media print {
  .dateInput {
    border: none;
    font-weight: bold;
  }
}
</style>


