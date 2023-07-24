<template>
  <ion-page id="page">

    <ion-content>                        
        <ActivityComponent ref="activityView" @activitySelected="activitiySelected_Received"/>

        <StopWatchComponent class="watch" ref="stopWatch" @stopwatch_started_event="stopWatch_Start_Received" 
                            @stopwatch_stopped_event="stopWatch_Stop_Received"
                            @deleteRequestReceived="deleteRequestReceived"/>
     
        <TableViewComponent ref="tableView"/>
    </ion-content>

  </ion-page>
</template>

<script lang="ts">
import { IonPage, IonContent} from '@ionic/vue';

import { defineComponent, ref } from 'vue';

import ActivityComponent from '@/components/ActivityComponent.vue';
import StopWatchComponent from '@/components/StopWatchComponent.vue';
import TableViewComponent from '@/components/TableViewComponent.vue';

export default defineComponent({
  components: {  IonPage, IonContent, TableViewComponent, ActivityComponent, StopWatchComponent },

  setup() {

      const tableView = ref();
      const stopWatch = ref();
      const activityView = ref();

      let startTime                = "";
      let stopTime                 = "";
      let currentSelectedActivity  = ""

      function stopWatch_Stop_Received(time:string) {
        stopTime = time;

        tableView.value.handleActivityStoppedEventFromParent(currentSelectedActivity, startTime, stopTime);
      }

      function stopWatch_Start_Received(time:string) {
        startTime = time;
      }

      function activitiySelected_Received(activity:string) {
        currentSelectedActivity = activity;
        
        // inform other components about selection
        tableView.value.handleActivitySelectedEventFromParent(currentSelectedActivity);
        stopWatch.value.handleActivitySelectedEventFromParent(currentSelectedActivity);
      }

      function deleteRequestReceived() {
        /* In form component about deletion */
         activityView.value.deleteRequestReceivedFromParent();
         tableView.value.deleteRequestReceivedFromParent();
      }

      return { 
            tableView,
            stopWatch,
            activityView,
            stopWatch_Stop_Received,
            stopWatch_Start_Received,
            activitiySelected_Received,
            deleteRequestReceived
        };
      }
});
</script>

<style scoped>
#page {
  margin-top:2.5em;
  padding: 1em;
  background-color: #121212;
}

.watch {
 background-color: #121212;
}

ion-content::part(scroll) {
  background-color: #121212
}
</style>
