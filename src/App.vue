<template>
  <ion-app class="app">
    <ion-header>
      <ion-toolbar class="toolbar">
        <ion-title class="yellow">Activity Tracker</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-router-outlet id="main-content"/>
  </ion-app>
</template>

<script lang="ts">
import { IonApp, IonHeader, IonTitle, IonToolbar, IonRouterOutlet } from '@ionic/vue';
import { defineComponent } from 'vue';

import { ellipsisHorizontal, ellipsisVertical } from 'ionicons/icons';

export default defineComponent({
  name: 'App',
  components: {
    IonApp,
    IonRouterOutlet,
    IonHeader,
    IonTitle,
    IonToolbar
  },
  setup() {
   
    return {
        ellipsisHorizontal, 
        ellipsisVertical
    }
  }
});
</script>


<style scoped>
.yellow {
  color: #FF9021;
  font-size: 1.2em;
  background-color: #1F1F1F;
}

ion-toolbar {
  --background: #1F1F1F;
}

.app {
  background-color: #121212;
}
</style>
