<template>
    <ion-grid class="grid">
     <ion-row>
        <ion-col size="9">  
          <ion-list class="roundedCorners">
              <ion-item>
                <ion-select class="text" @ionChange="activitySelected($event.detail.value)" interface="popover" placeholder="Select Activity">
                  <ion-select-option v-for="(activity, index) in activities" :key="index">{{activity}}</ion-select-option>
                </ion-select>
              </ion-item>
 
          </ion-list>
        </ion-col>

        <ion-col size="3">
          <ion-button expand="block" v-if="someActivityIsSelected" id="clicktrigger" @click="removeActivity(selectedActivity)">Remove</ion-button>
        </ion-col>

      </ion-row>

      <ion-row>
        <ion-col size="9"> 
          <ion-item class="bordered">
              <ion-input class="custom text" v-model="inputValue" @ionInput="validate" placeholder="New activity here..."></ion-input>
              <ion-note  v-if="invalid">{{errorMessage}}</ion-note>
          </ion-item>
        </ion-col>
  
        <ion-col size="3">
          <ion-button expand="block" v-if="!invalid" id="clicktrigger" @click="addActivity()">Add</ion-button>
        </ion-col>
      </ion-row>
    </ion-grid>    

</template>

<script lang="ts">
  import { IonItem, IonList, IonGrid, IonRow, IonCol, IonInput, IonButton, IonSelect, IonSelectOption, IonNote  } from '@ionic/vue';
  import { Storage } from '@ionic/storage';
  import { defineComponent, ref } from 'vue';
  
  export default defineComponent({
  components: {  IonGrid, IonItem, IonList, IonInput, IonRow, IonCol, IonSelect, IonSelectOption, IonButton, IonNote },
 
  setup(props, context) {
    const inputValue = ref('');
    const selectedActivity = ref('');
    const someActivityIsSelected = ref(false);

    const invalid = ref(true);
    const errorMessage = ref('');

    const showValue  = ref(false)
    const KEY_ACTIVITIES = "activities";

    const ionicStorage:Storage = new Storage();

    let activities = ref<string[]>([]);
    
    async function initStorage() {
        await ionicStorage.create();
    }
    
    function activitySelected(activity:string) {
        selectedActivity.value = activity;
        someActivityIsSelected.value = true;

        context.emit('activitySelected', activity);
    }

    async function readStorage() {
      const activitiesFromStorage = await ionicStorage.get(KEY_ACTIVITIES);
      activities.value = [];
      
      if (activitiesFromStorage) {
        let activitiesArray = activitiesFromStorage.split(",");

        activitiesArray.forEach(function(activity:string)  {
          activities.value.push(activity);
        });
      }
    }

    function validate(ev:any) {
        if (activities.value.indexOf(ev.target.value)!=-1) {
          invalid.value = true;  // entered activity already exists
          errorMessage.value = "Activity already exists";
        }
        else if (ev.target.value.length==0) {
           invalid.value = true;  // entered activity already exists
           errorMessage.value = "Field is empty";
        }
        else {
          invalid.value = false;  // entered activity does not exisits
        }
    }

    initStorage();
    readStorage();

    function checkIfActivityExists(activity:string) {
      let found = false;
     
      for (let i=0; i<activities.value.length; i++) {
        if (activities.value[i]==activity) {
          found = true;
          break;
        }
      }
  
      return found;
    }
  
    async function addActivity() {
      showValue.value = true
  
      if (!checkIfActivityExists(inputValue.value)) {
        activities.value.push(inputValue.value);

        // write into storage
        await ionicStorage.set(KEY_ACTIVITIES, activities.value.toString());

        inputValue.value = ""; // empty input field
        errorMessage.value = "Field is empty";
        invalid.value = true;
      }
    }

    async function removeActivity(activity:string) {
        if (activity.length!=0) {
          let index = activities.value.indexOf(activity);

          if (index>-1) {
            activities.value.splice(index, 1);

            someActivityIsSelected.value = false;
            
            // write array back to storage
            await ionicStorage.set(KEY_ACTIVITIES, activities.value.toString());
          }
        }
        else {
          console.log("nothing selected");
        }
    }

    async function deleteRequestReceivedFromParent() {
        await ionicStorage.clear(); 
        readStorage();
    }
  
    return { 
       inputValue, 
       activities,
       showValue,
       selectedActivity,
       someActivityIsSelected,
       invalid,
       errorMessage,
       addActivity,
       checkIfActivityExists,
       activitySelected,
       removeActivity, 
       validate,
       deleteRequestReceivedFromParent
    };
  }
  
  });
  </script>
  
  <style scoped>
.roundedCorners {
  border-radius: 5px;
  background-color: #1F1F1F;
  padding-top: 0px;
  padding-bottom: 0px;
}

ion-button {
  --background: #FF9021;
}

ion-input.custom {
  background-color: #1F1F1F;
}

.bordered {
  border-radius: 5px;
}

.text {
  font-size: 1.0em;
  color: white;
}

ion-item {
  --background: #1F1F1F;
}

ion-select::part(icon) {
  opacity: 1;
  margin-left:1em;
  color: #FF9021;
}

.grid {
  background-color: #121212;
}
</style>
  