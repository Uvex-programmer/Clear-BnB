<template>
  <div class="start-container">
    <div id="wrapper">
      <div class="hero-text">WE ARE HERE</div>
      <div class="hero-text">SO YOU CAN</div>
      <div class="hero-text">BE THERE</div>
    </div>
    <CitiesList :objects="objects" :text="'POPULAR LOCATIONS'" />
    <CitiesList :objects="objects" :text="'PARTY LOCATION'" />
    <CitiesList :objects="objects" :text="'CLOSE TO YOU'" />
  </div>
</template>

<script>
import CitiesList from '../components/CitiesList.vue'

export default {
  components: {
    CitiesList,
  },

  data() {
    return {
      objects: '',
    }
  },

  async created() {
    let res = await fetch('/rest/rental-objects?limit=6')
    this.objects = await res.json()
  },
}
</script>

<style scoped>
.start-container {
  padding: 1rem;
  flex: 1;
  display: flex;
  flex-direction: column;
  max-width: 65rem;
  margin: auto;
}

#wrapper {
  font-size: 6rem;
  margin-top: 4rem;
  display: flex;
  flex-direction: column;
  width: 100%;
  margin-bottom: 20rem;
}

#wrapper:focus {
  outline: none;
}

.hero-text {
}

#wrapper div {
  display: flex;
  margin-bottom: 3rem;
  color: #fff;
  letter-spacing: 0.25em;
  text-shadow: rgba(0, 0, 0, 0.1) 0 20px 80px;
}

#wrapper:hover div:nth-child(1) {
  text-shadow: #fff 0 0 2px, #fff 0 4px 5px, #ddd 0 9px 5px, #ccc 0 12px 10px,
    rgba(0, 0, 0, 0.2) 0 14px 5px, rgba(0, 0, 0, 0.1) 0 20px 10px,
    rgba(0, 0, 100, 0.2) 0 15px 80px;
}

#wrapper:hover div:nth-child(3) {
  text-shadow: #fff 0 0 2px, #fff 0 4px 5px, #ddd 0 9px 5px, #ccc 0 12px 10px,
    rgba(0, 0, 0, 0.2) 0 14px 5px, rgba(0, 0, 0, 0.1) 0 20px 10px,
    rgba(0, 0, 100, 0.2) 0 15px 80px;
}

@media screen and (max-width: 400px) {
  #wrapper {
    font-size: 3rem;
  }
}
</style>
