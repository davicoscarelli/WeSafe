<template>
  <q-page>
     <q-scroll-area ref="chatArea" :class="{ 'chat-area': !showOptions, 'chat-area-min': showOptions }" > 
        <q-chat-message
          :key="message.id" v-for="message in messages"
          :text="[message.text]"
          :sent="message.author == 'bot' ?  false : true"
          :bg-color="message.author == 'bot' ? 'secondary' : '#e0e0e0'"
          :text-color="message.author == 'bot' ?  'white' : 'black'"
          class="q-px-md q-pt-sm"
          :class="{ 'message-out': message.author === 'you', 'message-in': message.author !== 'you' }"
        >
        <p v-if="message.text2">{{message.text2}}</p>

        <q-img
          class="rounded-borders"
          @click="openVideo(message.videoLink)"
          v-if="message.video"
          :src="message.video"
        />

        <p v-if="message.text3">{{message.text3}}</p>
        
        
        
        <div class="col flex flex-center" v-if="message.id == 0">
          For example:
          <q-btn rounded color="white" no-caps text-color="primary" style="width: 80%" class="q-ma-xs" label="Choking" @click="buttonSend('Choking')"/>
          <!-- <q-btn rounded color="white" no-caps text-color="primary" style="width: 80%" class="q-ma-xs" label="Bleeding" @click="buttonSend('Bleeding')"/>
          <q-btn rounded color="white" no-caps text-color="primary" style="width: 80%" class="q-ma-xs" label="Drowning" @click="buttonSend('Drowning')"/>
          <q-btn rounded color="white" no-caps text-color="primary" style="width: 80%" class="q-ma-xs" label="Fractures" @click="buttonSend('Fractures')"/> -->
        </div>
          <q-btn v-if="message.warning" style="width: 100%" icon="phone" flat color="white"  :class="{ 'bg-red': message.text === 'Call the emergency!' ,  'bg-orange': message.text !== 'Call the emergency!'}" label="Call 911" href="tel:911" />
         
        </q-chat-message>
        

        <q-chat-message
          v-if="loading"
          bg-color="secondary"
          text-color="white"
          class="q-px-md q-pt-sm message-in"
        >
        <q-spinner-dots size="2rem"/>
        </q-chat-message>
      </q-scroll-area>

      <q-footer elevated>
        <transition name="slide-fade">
          <div  class="row bg-secondary q-pa-sm justify-center " style="height: 50px" v-show="showOptions">
            <q-btn :key="option" v-for="option in options[optionsType]" outline rounded color="white" style="width: 30%" no-caps dense text-color="white"  class="q-mx-sm" :label="option" @click="optionSend(option)"/>
          </div>
          </transition>
        <q-toolbar>
          <q-form @submit="sendMessage('out')" class="full-width ">
            <div class="row">
            <q-input
              ref="newMessage"
              :bg-color="returnColor"
              rounded
              v-model="youMessage"
              placeholder="Message"
              class=" col-11 q-pr-sm"
              dense
              outlined
            >
              <template>
                <q-btn round dense flat type="submit" icon="send" color="primary" />
              </template>
            </q-input>
            <div class="col-1 flex flex-center">
              <q-btn v-if="!recording" round dense flat @click="speechToText" icon="mic" :color="$q.dark.isActive ? 'black' : 'white'"  />
              <q-spinner-bars v-if="recording" :color="$q.dark.isActive ? 'black' : 'white'" size="2em"/>
            </div>
            </div>
          </q-form>
        </q-toolbar>
      </q-footer>
  </q-page>
</template>

<script>
const sdk = require("microsoft-cognitiveservices-speech-sdk");
import { openURL } from 'quasar'



export default {
  
  name: 'PageIndex',
  data: () => ({
    botMessage: {},
    loading: false,
    youMessage: '',
    recording: false,
    history: [],
    messages: [],
    options: [['Yes', 'No'], ['Me', 'Someone else']],
    showOptions: false,
    optionsType: null
  }),
  computed: {
    user(){
      return { name: 'Davi', age: 1}
    },
    returnColor() {
      return this.$q.dark.isActive ? 'black' : 'grey-2'
    }
  },
  created(){
    
  },
  mounted(){
    this.botMessage = {text:'Welcome to WeSafe! Whats your emergency?'}
    this.synthesizeSpeech(this.botMessage)

  },
  methods: {
    openVideo(url){
      openURL(url)
    },

    speechToText(){
      try {
        const speechConfig = sdk.SpeechConfig.fromSubscription("80336464dd984e3489ab38cbb895823e", "eastus");
        let audioConfig = sdk.AudioConfig.fromDefaultMicrophoneInput();
        let recognizer = new sdk.SpeechRecognizer(speechConfig, audioConfig);
        const phraseList = sdk.PhraseListGrammar.fromRecognizer(recognizer);
        phraseList.addPhrase(['choking','no','yes','clear','me','other','someone','else']);
        
        console.log('Speak into your microphone.');
        this.recording = true
        recognizer.recognizeOnceAsync(result => {
            this.youMessage = result.text
            console.log(`RECOGNIZED: Text=${result.text}`);
            this.recording = false
            this.sendMessage('out')
            this.showOptions = false 
            return
        });
        
      } catch (error) {
        console.log("recognition", error)
      }
    },

    
    async synthesizeSpeech(message) {
      let text = ''

      if (message.text2) text = message.text + ' ' + message.text2 + ' ' + message.text3
      else text = message.text
            
    
      this.loading = true

      const speechConfig = sdk.SpeechConfig.fromSubscription("80336464dd984e3489ab38cbb895823e", "eastus");

      speechConfig.speechSynthesisOutputFormat = sdk.SpeechSynthesisOutputFormat.Riff24Khz16BitMonoPcm;

      const synthesizer = new sdk.SpeechSynthesizer(speechConfig, undefined);
      synthesizer.speakTextAsync(
          text,
          result => {
              const audioData = result.audioData;
              this.loading = false
              console.log(text)
              
              console.log(`Audio data byte size: ${audioData.byteLength}.`)
              
              
              synthesizer.close();
              if (this.optionsType != null) this.showOptions = true

               
              console.log(text.length)
              this.sendMessage('in')
              let messageDisplay = this.$refs.chatArea
              messageDisplay.setScrollPosition(messageDisplay.$el.scrollHeight + 10000, 1);
            

              
          },
          error => {
              console.log(error);
              synthesizer.close();
          });
    },

    optionSend(message){
      this.youMessage = message
      this.sendMessage('out')
      this.showOptions = false 

    },
    buttonSend(message){
      this.youMessage = message
      this.sendMessage('out')
      },

    async bot(){
      let message = {text: this.botMessage, optionsType: this.optionsType}
      const input = JSON.stringify(this.history)
      console.log(input, "inputtt")

      if (this.history[0] == 0){
        if (input == '[0]') { message = {text: "Is the victim you or someone else?", optionsType: 1}}
        else if (input == '[0,0]') { message = {text: "Is the victim having difficulty breathing?", optionsType: 0}}
        else if (input == '[0,0,0]') {message = {text: "Is the victim over 8 years old?", optionsType: 0}}
        else if (input == '[0,0,1]') { message = {text: "Is the victim conscious?", optionsType: 0}}
        else if (input == '[0,0,1,0]') { message = {text: "Is the victim over 8 years old?", optionsType: 0}}
        else if (input == '[0,0,1,1]') { message = {text: "Is the victim over 8 years old?", optionsType: 0}}
        else if (input == '[0,0,0,1]') { message = {text: "Recline the victim and let the person cough until what was obstructing the airway comes out or can be removed.", text2: 'Keep the victim calm so the situation does not get worse.', text3: 'In case of aggravation call emergency!', warning: true, optionsType: null, }}
        else if (input == '[0,0,0,0]') { message = {text: "Hold the victim with one hand, with the fingers positioned on the jaw and supporting the victim's body on his forearm. Tilt the victim's head down and support the weight on the leg. Perform five moderate slaps on the victim's back with the other hand flat.", text2: 'Then turn the victim to the front and proceed with five chest thrusts. Repeat the process until the victim breathes.', text3: 'In case of aggravation call emergency!', video: './answer2.gif', videoLink: 'http://www.youtube.com/watch?v=gHZdBY-CkGw&t=1m23s', warning: true, optionsType: null}}
        else if (input == '[0,0,1,0,0]') { message = {text: "Call the emergency!", text2: "Then locate the area over the heart to begin chest compressions – between the breasts and on the lower third of the sternum. Stand or kneel directly over the patient's chest. Lock your elbows and use only your upper bodyweight to supply the force for the chest compressions, and count as you perform them. Perform 30 chest compressions.", text3: "Perform 30 chest compressions. Look inside their mouth. See if any obstructions came loose from the chest compressions. If you see something, sweep it out using your finger. If you don't, breathe into the mouth If you see an object, sweep it out and try two more rescue breaths.", video: './answer6.gif', videoLink: 'https://www.youtube.com/watch?v=_K7Dwy6b2wQ', warning: true, optionsType: null}}
        else if (input == '[0,0,1,0,1]') { message = {text: "Call the emergency!", text2: "Then lower the person on his or her back onto the floor, arms to the side. Clear the airway. If a blockage is visible at the back of the throat or high in the throat, reach a finger into the mouth and sweep out the cause of the blockage. Don't try a finger sweep if you can't see the object. Be careful not to push the food or object deeper into the airway.", text3: "Begin CPR if the object remains lodged and the person doesn't respond after you take the above measures. The chest compressions used in CPR may dislodge the object. Remember to recheck the mouth periodically.", video: './answer5.gif', videoLink: 'https://www.youtube.com/watch?v=5kmsKNvKAvU', warning: true, optionsType: null}}
        else if (input == '[0,0,1,1,0]') { message = {text: "Hold the victim with one hand, with the fingers positioned on the jaw and supporting the victim's body on his forearm. Tilt the victim's head down and support the weight on the leg. Perform five moderate slaps on the victim's back with the other hand flat.", text2: 'Then turn the victim to the front and proceed with five chest thrusts. Repeat the process until the victim breathes.', text3: 'In case of aggravation call emergency!', video: './answer2.gif', videoLink: 'http://www.youtube.com/watch?v=gHZdBY-CkGw&t=1m23s', warning: true, optionsType: null}}
        else if (input == '[0,0,1,1,1]') { message = {text: "Is the victim Pregnant or Obese?", optionsType: 0}}
        else if (input == '[0,0,1,1,1,0]') {message = {text: "Stand or kneel behind the person and wrap your arms around his or her waist. Make a fist with one hand. Place the thumb side of your fist against the person's belly, just above the belly button but well below the breastbone. Grasp your fist with the other hand. Give a quick upward thrust into the belly. This may cause the object to pop out.", text2: " You may need to use more force for a large person and less for a child or small adult. Repeat thrusts until the object pops out or the person faints.", text3: 'In case of aggravation call emergency!', video: './answer3.gif', videoLink: 'https://www.youtube.com/watch?v=DE45ks9miIw', warning: true, optionsType: null}}
        else if (input == '[0,0,1,1,1,1]') {message = {text: "Stand or kneel behind the person and wrap your arms around his or her chest. Make a fist with one hand. Place the thumb side of your fist at the base of the breastbone, just above the joining of the lowest ribs. Grasp your fist with the other hand. Press hard into the chest, with a quick thrust.", text2: " Repeat thrusts until the object pops out or the person faints.", text3: 'In case of aggravation call emergency!', video: './answer4.gif', videoLink: 'https://www.youtube.com/watch?v=DHFnxAq1-KM&t=1m50s', warning: true, optionsType: null}}

        else if (input === '[0,1]') { message = {text: "Are you having trouble breathing?", optionsType: 0}}
        else if (input == '[0,1,0]') { 
          if (this.user.age.length == 0){
            message = {text: "Are you over 8 years old?", optionsType: 0}
          }else {
            if (this.user.age > 8){
              this.history.push(1)
              message = {text: "Recline yourself and cough until what was obstructing your airway comes out or can be removed.", text2: 'Keep calm so the situation does not get worse.', text3: ' In case of aggravation call emergency!', warning: true, optionsType: null }
            }
            else{
              this.history.push(0)
              message = {text: "Ask for help and tell the person to follow the next steps:", text2: "Recline the victim and let the person cough until what was obstructing the airway comes out or can be removed.", text3: 'Keep the victim calm so the situation does not get worse. In case of aggravation call emergency!', warning: true, optionsType: null, }

            }
          }
        }
        else if (input == '[0,1,1]') { 
          if (this.user.age.length == 0){
            message = {text: "Are you over 8 years old?", optionsType: 0}
          }else {
            if (this.user.age > 8){
              this.history.push(1)
              message = {text: "Lean over a table edge, chair, or railing. Quickly thrust your upper belly area (upper abdomen) against the edge. You can also do this: Make a fist with one hand and put the thumb side between your belly button and rib cage. Place your other hand on top of that. Push as hard as you can in a quick motion straight into your abdomen.", text2: "If you are pregnant, you should place your hands higher than usual, under the breast bone. Another alternative maneuver is to slam your back into a wall while coughing.", text3: 'In case of aggravation call emergency!', video: './answer3_2.gif', videoLink: 'https://www.youtube.com/watch?v=ljL9JcK6RnM', warning: true, optionsType: null}
            }
            else{
              this.history.push(0)
              message = {text: "Are you alone?", optionsType: 0}

            }
          }
        }
        else if (input === '[0,1,0,0]') {message = {text: "Ask for help and tell the person to follow the next steps:", text2: "Recline the victim and let the person cough until what was obstructing the airway comes out or can be removed.", text3: 'Keep the victim calm so the situation does not get worse. In case of aggravation call emergency!', warning: true, optionsType: null, }}
        else if (input === '[0,1,0,1]') { message = {text: "Recline yourself and cough until what was obstructing your airway comes out or can be removed.", text2: 'Keep calm so the situation does not get worse. In case of aggravation call emergency!', warning: true, optionsType: null }}
        else if (input === '[0,1,1,1]') {message = {text: "Lean over a table edge, chair, or railing. Quickly thrust your upper belly area (upper abdomen) against the edge. You can also do this: Make a fist with one hand and put the thumb side between your belly button and rib cage. Place your other hand on top of that. Push as hard as you can in a quick motion straight into your abdomen.", text2: "If you are pregnant, you should place your hands higher than usual, under the breast bone. Another alternative maneuver is to slam your back into a wall while coughing.", text3: 'In case of aggravation call emergency!', video: './answer3_2.gif', videoLink: 'https://www.youtube.com/watch?v=ljL9JcK6RnM', warning: true, optionsType: null}}
        else if (input === '[0,1,1,0,0]') {message = {text: "Ask for help and tell the person to follow the next steps:", text2: "Recline the victim and let the person cough until what was obstructing the airway comes out or can be removed.", text3: 'Keep the victim calm so the situation does not get worse. In case of aggravation call emergency!', warning: true, optionsType: null, }}
        else if (input === '[0,1,1,0,1]') {message = {text: "Call the emergency!", warning: true, optionsType: null}}
    
      }else if (input[0] == 1){
          message = {text: 'other.1', optionsType: 0}
      }
      this.botMessage = message
      this.optionsType = message.optionsType
      await this.synthesizeSpeech(message)

            
    },

    async createHistory(msg){
      let message = msg.toLowerCase()

      console.log("Hmessage",message)

      if ((message === 'no' || this.findWord('no', message)) && this.history.length != 0){
        this.history.push(0)
      }
      else if ((message === 'yep' || message === 'yes' || message === 'yeah' || this.findWord('yes', message) || this.findWord('yep', message) || this.findWord('yeah', message)) && this.history.length != 0){
        this.history.push(1)
      }

      else if ((message === 'choking' || this.findWord('choking', message)) && this.history.length == 0){
        this.history.push(0)
      }
      else if (message === 'clear' || this.findWord('clear', message)){
        this.clearAllMessages()
      }
      else if ((message === 'me' || this.findWord('me', message)) && this.history.length != 0){
        this.history.push(1)
      }
      else if ((message === 'other' || this.findWord('other', message) || message.includes('someone else')) && this.history.length != 0){
        this.history.push(0)
      }
      else{
        this.botMessage = "Sorry, I didn't understand. Can you repeat?"
      }
      
    },
    findWord(word, str) {
      let test = str.match(/[\w-']+|[^\w\s]+/g).some(function(w){return w === word})
      return test
    },

    sendMessage(direction) {
      if (!this.youMessage && !this.botMessage) {
        return
      }
      
      if (direction === 'out') {
        this.messages.push({text: this.youMessage, author: 'you', id: this.messages.length})
        this.createHistory(this.youMessage)
        this.youMessage = ''
        this.bot()
      } else if (direction === 'in') {
        console.log(this.botMessage, "bot")
        this.messages.push({...this.botMessage, author: 'bot', id: this.messages.length})
        this.botMessage = ''
      } 
      let messageDisplay = this.$refs.chatArea
      messageDisplay.setScrollPosition(messageDisplay.$el.scrollHeight + 10000, 1);
      
    },

    clearAllMessages() {
      this.messages = []
      this.history = []
      this.botMessage = 'Welcome to WeSafe! Whats your emergency?'
      this.synthesizeSpeech(this.botMessage)
    }
  }
}
</script>

<style>
  .slide-fade-enter-active {
    transition: all .3s ease;
  }
  .slide-fade-leave-active {
    transition: all .2s ease;
  }
  .slide-fade-enter, .slide-fade-leave-to
  /* .slide-fade-leave-active below version 2.1.8 */ {
    transform: translateY(10px);
    opacity: 0;
  }

  .headline {
    text-align: center;
    font-weight: 100;
  }
  .chat-area-min{
    height: calc(90vh - 100px);

  }
  .chat-area {
  /*   border: 1px solid #ccc; */
    height: calc(100vh - 100px);
    /* box-shadow: 2px 2px 5px 2px rgba(0, 0, 0, 0.3) */
  }
 
  .message-out {
    padding-left: 15%;
  }
  .message-in {
    padding-right: 15%;
  }
  .chat-inputs {
    display: flex;
    justify-content: space-between;
  }
  #person1-input {
    padding: .5em;
  }
  #person2-input {
    padding: .5em;  
  }

</style>
