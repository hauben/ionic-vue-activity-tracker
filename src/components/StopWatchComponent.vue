<template>
   <ion-grid class="container ion-text-center">  

      <ion-row>  
        <ion-col @click="showDeleteDialog" class="timer">{{ time }}</ion-col>
      </ion-row>

      <ion-row class="ion-margin-bottom">
        <ion-col class="startedText"> 
          <p v-if="running">Started at: {{ startedAt }}</p>
        </ion-col>
      </ion-row>

      <ion-row>  
        <ion-col> 
          <ion-button class="ion-margin-end"  :disabled="!activityIsSelected || running" @click="startClock()">Start</ion-button>
          <ion-button :disabled="!running" @click="stopClock()">Stop</ion-button>
        </ion-col>
      </ion-row>

    </ion-grid>  
</template>
  
<script lang="ts">

/* Stop Watch Implementation based on an example Vue component from Raphael Octau 
   https://codepen.io/raphael_octau */

import { IonRow, IonGrid, IonCol, IonButton, alertController } from '@ionic/vue';
import { defineComponent, ref } from 'vue';

export default defineComponent({

    components: { IonGrid, IonRow, IonCol, IonButton },

    setup(props, context) {
      let time             = ref('00:00:00.000')
      let timeBegan:any    = null;
      let timeStopped:any  = null;
      let stoppedDuration  = 0
      let started:any      = null;
      let running          = ref(false);
      let startedAt        = ref("");

      let activityIsSelected = ref(false);

      const showDeleteDialog = async () => {   // hidden dialog if pressed on the timer - usefull for a full reset
        const alert = await alertController.create({
          header: 'DELETE ALL ACTIVITES?',
          buttons: [
            {
              text: 'Cancel',
              role: 'cancel',
              handler: () => {
                console.log("canceled");
              },
            },
            {
              text: 'OK',
              role: 'confirm',
              handler: () => {
                context.emit('deleteRequestReceived');
              },
            },
          ],
        });

        await alert.present();

      };
      
      function startClock() {
            if (running.value) return;
            
            let date = new Date();
            startedAt.value = `${date.getHours().toString().padStart(2, '0')}:${date.getMinutes().toString().padStart(2, '0')}`

            if (timeBegan === null) {
              reset();
              timeBegan = Date.now();
            }

            context.emit("stopwatch_started_event", new Date().getTime());

            if (timeStopped !== null) {
              stoppedDuration += (Date.now() - timeStopped);
            }

            started = setInterval(clockRunning, 10);	
            running.value = true;
      }

      function stopClock() {
          reset();
          timeStopped = new Date().getTime();
          context.emit("stopwatch_stopped_event", timeStopped);
      }

      function reset() {
        running.value = false;
        clearInterval(started);
        started = 0;
        stoppedDuration = 0;
        timeBegan = null;
        timeStopped = null;
        time.value = "00:00:00.000";
        context.emit("stopwatch_reset_event");
      }

      function clockRunning(){
        var currentTime = new Date().getTime()
        , timeElapsed = new Date(currentTime - timeBegan - stoppedDuration)
        , hour = timeElapsed.getUTCHours()
        , min = timeElapsed.getUTCMinutes()
        , sec = timeElapsed.getUTCSeconds()
        , ms = timeElapsed.getUTCMilliseconds();

        time.value = 
          zeroPrefix(hour, 2) + ":" + 
          zeroPrefix(min, 2) + ":" + 
          zeroPrefix(sec, 2) + "." + 
          zeroPrefix(ms, 3);
      }

      function zeroPrefix(num:number, digit:number) {
        var zero = '';
        for(var i = 0; i < digit; i++) {
          zero += '0';
        }
        return (zero + num).slice(-digit);
      }


      function handleActivitySelectedEventFromParent(activity:string) {
        if (activity.length>0) {
          activityIsSelected.value = true;
        }
        else {
          activityIsSelected.value = false;
        }
      }
  
    return { 
        time,
        running,
        startedAt,
        activityIsSelected,
        startClock,
        stopClock,
        reset,
        handleActivitySelectedEventFromParent,
        showDeleteDialog
    };
}

});
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css?family=Share+Tech+Mono');

.timer {
  font-family: 'Share Tech Mono', sans-serif;
  font-size: 2.5em;
  color: white;
}

ion-button {
  --background: #FF9021;
}

.container {
  padding: 1em;
}

p {
  margin-top:0px;
  margin-bottom:0px;
}

.startedText {
  font-size: 1.2em;
  color: white;
}
</style>
