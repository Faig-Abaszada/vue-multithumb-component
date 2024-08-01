<template>
  <main>
    <br>
    <br>
    <br>
    <br>
    <br>
    <p>{{data}}</p>
    <br>
    <multi-thumb-slider
        :options="multithumbOptions"
        v-model:display="data.display"
        v-model:video="data.video"
        v-model:social="data.social"
        v-model:search="data.search"
        :totalBudget="data.totalBudget"
        :error="{ has: false, messages: 'sliderErrorMessages'}"
    />
  </main>
</template>
<script>
import MultiThumbSlider from "@/components/MultiThumbSlider.vue";
export default  {
  name: 'HomeView',
  components: {
    MultiThumbSlider,
  },
  data() {
    return {
      data: {
        display: 25,
        video: 25,
        social: 25,
        search: 25,
        totalBudget: 100,
      },
      multithumbOptions:  [
        {
          label: 'Display',
          key: 'display',
          default: 25,
          color: '#4DAE1F',
          disabled: false,
          error: false
        },
        {
          label: 'Video',
          key: 'video',
          default: 25,
          color: '#2CADC9',
          disabled: false,
          error: false
        },
        {
          label: 'Social',
          key: 'social',
          default: 25,
          color: '#2C6BC9',
          disabled: false,
          error: false
        },
        {
          label: 'Search',
          key: 'search',
          default: 25,
          color: '#6844D0',
          disabled: false,
          error: false
        }
      ]
    };
  },
}
</script>