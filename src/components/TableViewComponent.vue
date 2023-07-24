<template>
  <ion-grid class="container">
      <ion-row>
        <ion-col>
            <ion-list class="bordered">
                <ion-item>
                    <ion-select v-model="currentFilter" class="text" @ionChange="timeRangeSelected($event.detail.value)" interface="popover" placeholder="Select timerange">
                        <ion-select-option v-for="(sortBy, index) in sortings" :key="index">{{sortBy}}</ion-select-option>
                    </ion-select>
                </ion-item>
            </ion-list>
        </ion-col>
      </ion-row>

      <ion-row class="headingRow ion-text-center">
        <ion-col>Total: </ion-col>
        <ion-col>{{sumTotal}}</ion-col>
      </ion-row>
      
      <ion-row class="activityRow" v-for="(item, index) in activitiesFilteredRef" :key="index">
        <ion-col class="tableText ion-padding-top ion-padding-start">{{item.activity}}</ion-col>
        <ion-col class="tableText ion-padding-top ion-padding-start">{{secondsToDateString(item.start)}}</ion-col>
        <ion-col class="tableText ion-padding-top ion-padding-start">{{secondsToDateString(item.end)}}</ion-col>
        <ion-col class="tableTextBig tableText ion-padding-top ion-padding-start">{{calculateDuration(item.start, item.end)}}</ion-col>
        <ion-col class="ion-padding-start ion-padding-top"><ion-checkbox @ionChange="handleSelectChange(index, $event)"></ion-checkbox></ion-col>
      </ion-row>
    
      <ion-row>
        <ion-button class="deleteButton ion-text-center" @click="deletedSelectedActivities" v-if="rowsSelected>0">Delete selected entries</ion-button>
      </ion-row>
  
      <ion-alert
            :is-open="showAlertDialog"
            header="Information"
            message="You may want to select an activity first."
            :buttons="['OK']"
            @didDismiss="setAlertDialogOpen(false)"
    ></ion-alert>
  </ion-grid>    
</template>
  
<script lang="ts">
import { IonAlert, IonItem, IonList, IonRow, IonCol, IonGrid, IonSelect, IonSelectOption, IonButton, IonCheckbox } from '@ionic/vue';
import { Storage } from '@ionic/storage';
import { defineComponent, ref } from 'vue';


import {
    isToday, 
    secondsToHhMmSs, 
    secondsToDateString, 
    calculateDuration,
    isThisMonth,
    isThisYear,
    isYesterday,
    isLastMonth,
    isThisWeek,
    isLastWeek
} from '@/helpers';

interface IActivityRecord {
    activity: string,
    start: number,
    end: number
}

export default defineComponent({
    components: { IonAlert, IonItem, IonList, IonGrid, IonRow, IonCol, IonSelect, IonSelectOption, IonButton, IonCheckbox },

    setup() {
        const KEY_RECORDS = "records";

        const TODAY          = "Today";
        const YESTERDAY      = "Yesterday";
        const MONTH          = "This Month";
        const PREV_MONTH     = "Last Month";
        const YEAR           = "This Year";
        const WEEK           = "This Week";
        const PREV_WEEK      = "Last Week";
        const All            = "All";
        const All_UNFILTERED = "All (unfiltered)"  // All activities independent of an selected one

        let sortings   = ref([TODAY, YESTERDAY, WEEK, PREV_WEEK, MONTH, PREV_MONTH, YEAR, All, All_UNFILTERED]);
      
        const sumTotal   = ref('00:00:00')
        let rowsSelected = ref(0);

        let activitiesFilteredRef = ref<IActivityRecord[]>([]);
        let allActivities: Array<IActivityRecord> = [];

        const ionicStorage:Storage = new Storage();
        let checkedIndices: number[] = [];

        let currentActivity = "";  // current selected activity - received via event from ActivityComponent

        let currentFilter = ref(All_UNFILTERED);  // select ALL_UNFILTERED as default

        const showAlertDialog = ref(false);
        const setAlertDialogOpen = (state: boolean) => (showAlertDialog.value = state);

  
        function sumUpDurationTime(activities: Array<IActivityRecord>) {
            let sum = 0;

            activities.forEach( (activity:IActivityRecord) => {
               sum = sum + (activity.end- activity.start)
            });

            return secondsToHhMmSs(Math.round(sum/1000));
        }
        

        function timeRangeSelected(range:string) { 
            
            if (currentActivity=="" && range!=All_UNFILTERED) {  // show info dialog that an activity must be selected first in all time ranges except ALL_UNFILTERED
                showAlertDialog.value = true;
            }
           
            sumTotal.value = "00:00:00";  // reset total time counter

            activitiesFilteredRef.value = [];  // clear list

            switch (range) {
                case TODAY: /* filter for todays activities */
                    allActivities.forEach( (activity:IActivityRecord) => {
                        if (isToday(activity.start) && (activity.activity == currentActivity)) {
                            activitiesFilteredRef.value.push(activity);
                        }
                    });
                break;

                case MONTH:
                    allActivities.forEach( (activity:IActivityRecord) => {
                        if (isThisMonth(activity.start) && (activity.activity == currentActivity)) {
                            activitiesFilteredRef.value.push(activity);
                        }
                    });
                break;

                case PREV_MONTH:
                    allActivities.forEach( (activity:IActivityRecord) => {
                        if (isLastMonth(activity.start) && (activity.activity == currentActivity)) {
                            activitiesFilteredRef.value.push(activity);
                        }
                    });
                break;

                case YEAR:
                    allActivities.forEach( (activity:IActivityRecord) => {
                        if (isThisYear(activity.start) && (activity.activity == currentActivity)) {
                            activitiesFilteredRef.value.push(activity);
                        }
                    });
                break;

                case YESTERDAY:
                    allActivities.forEach( (activity:IActivityRecord) => {
                        if (isYesterday(activity.start) && (activity.activity == currentActivity)) {
                            activitiesFilteredRef.value.push(activity);
                        }
                    });
                break;    

                case WEEK: {
                    allActivities.forEach( (activity:IActivityRecord) => {
                        if ( isThisWeek(activity.start) && (activity.activity == currentActivity) ) {
                            activitiesFilteredRef.value.push(activity);
                        }
                    });
                   }
                break;    

                case PREV_WEEK: {
                    allActivities.forEach( (activity:IActivityRecord) => {
                        if ( isLastWeek(activity.start) && (activity.activity == currentActivity) ) {
                            activitiesFilteredRef.value.push(activity);
                        }
                    });
                }
                break;
                
                case All:          
                    allActivities.forEach( (activity:IActivityRecord) => {
                            if (activity.activity == currentActivity) {
                                activitiesFilteredRef.value.push(activity);
                            }
                    });
                break;

                case All_UNFILTERED:
                    allActivities.forEach( (activity:IActivityRecord) => {
                        activitiesFilteredRef.value.push(activity);
                    });
                break;
            }

            /* sum up activity time according filtered result */ 
            if (activitiesFilteredRef.value.length!=0) {
                        sumTotal.value = sumUpDurationTime(activitiesFilteredRef.value);
            }
        }   


        async function readOutStorageSyncHMI() {
                // read out all stored activities
                const activitiesFromStorage = await ionicStorage.get(KEY_RECORDS);

                if (activitiesFromStorage) {
                    let json = JSON.parse(activitiesFromStorage);

                    json.activities.forEach( (activity:IActivityRecord) => {
                        activitiesFilteredRef.value.push(activity);
                        allActivities.push(activity);
                    });
                }
        }


        async function initStorage() {
            await ionicStorage.create();
            
            readOutStorageSyncHMI();
        }


        function handleSelectChange(rowIndex: number, event:any) {
            let checked = event.detail.checked;

            if (checked) {
                rowsSelected.value++;
                checkedIndices.push(rowIndex);
            }
            else {
                rowsSelected.value--;
                checkedIndices.splice(rowIndex, 1);
            }
            checkedIndices.sort();
        }


        async function deletedSelectedActivities() {

            // delete the selected items from the array
            checkedIndices.sort((a,b) => b - a).forEach(index => allActivities.splice(index, 1));
            
            // empty helper array
            checkedIndices = [];

            // set flag - nothing is selected - Delete
            rowsSelected.value = 0;
          
            let storeMe = {
                activities: allActivities
            }

            // write the remaining activities to storage again
            await ionicStorage.set(KEY_RECORDS, JSON.stringify(storeMe));

            activitiesFilteredRef.value = [];
            allActivities = [];

            readOutStorageSyncHMI();
        }


        initStorage();
       

        function handleActivitySelectedEventFromParent(activity:string) {
            console.log(`selected activity received ${activity}`)

            currentActivity = activity;

            if (currentFilter.value!=All_UNFILTERED) {   // only apply activity selection if NOT in All (unfiltered) mode
                timeRangeSelected(currentFilter.value)
            }
        }


        async function handleActivityStoppedEventFromParent(activity:string, startTime: number, stopTime: number) {
            console.log(`Activity: ${activity} started at: ${startTime} stopped at: ${stopTime}`);

            // write new activity to storage and show in view
            let newActivity = {
                activity: activity,
                start: startTime,
                end: stopTime
            };
          
           // Only put newActivity into the reactive activitiesFilteredRef in case if the time filter is set to something like
           // ALL (unfiltered), Today, This Week, This Month, This Year or All
           if ( [All_UNFILTERED, TODAY, WEEK, MONTH, YEAR, All].indexOf(currentFilter.value) > -1) {       
                activitiesFilteredRef.value.push(newActivity);
           }

           allActivities.push(newActivity);  // store new activity recording 

           let storeMe = {
            activities: allActivities
           }

           await ionicStorage.set(KEY_RECORDS, JSON.stringify(storeMe));
        }


        function deleteRequestReceivedFromParent() {
            activitiesFilteredRef.value = [];
            allActivities = [];
        }


        return { 
            sortings,
            activitiesFilteredRef,
            rowsSelected,
            sumTotal,
            currentFilter,
            showAlertDialog,
            handleActivitySelectedEventFromParent,
            handleActivityStoppedEventFromParent,
            secondsToDateString,
            calculateDuration,
            handleSelectChange,
            deletedSelectedActivities,
            timeRangeSelected,
            deleteRequestReceivedFromParent,
            isThisWeek,
            setAlertDialogOpen
        };
    }
});
</script>

<style scoped>
.headingRow {
    font-size: 1.2em;
    margin-top: 0.5em;
    border: 1px solid rgb(167, 167, 167);
    border-top-right-radius: 5px;
    border-top-left-radius: 5px;
    background-color: #1F1F1F;
    color: white;
}

.bordered {
  border-radius: 5px;
  background-color: #1F1F1F;
  padding-top: 0px;
  padding-bottom: 0px;
}

.activityRow {
    border: 1px solid rgb(167, 167, 167);
    color: white;
}

ion-button, ion-checkbox {
  --background: #FF9021;
}

ion-checkbox {
    --background-checked: #FF9021;
    --border-color-checked: #ffffff;
    --size: 1.5em;
}

.deleteButton {
    margin-top: 1em;
}

.text, .tableText {
  font-size: 1.0em;
  color: white;
}



ion-select::part(icon) {
  opacity: 1;
  margin-left:1em;
  color: #FF9021;
}

.container {
    background-color: #121212;
}

ion-item {
  --background: #1F1F1F;
}

.tableTextBig {
    font-size: 1.2em;
    font-weight: bold;
}
</style>

