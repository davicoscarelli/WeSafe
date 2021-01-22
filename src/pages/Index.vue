<template>
  <q-page class="">
     <q-scroll-area ref="chatArea" class="chat-area">
        <q-chat-message
          :key="message.id" v-for="message in messages"
          :text="[message.body]"
          :sent="message.author == 'bot' ?  false : true"
          :bg-color="message.author == 'bot' ? 'secondary' : '#e0e0e0'"
          :text-color="message.author == 'bot' ?  'white' : 'black'"
          class="q-px-md q-pt-sm"
          :class="{ 'message-out': message.author === 'you', 'message-in': message.author !== 'you' }"
        >
        <div class="col flex flex-center" v-if="message.id == 0">
          <q-btn rounded color="white" no-caps text-color="primary" style="width: 80%" class="q-ma-xs" label="Choking" @click="buttonSend('Choking')"/>
          <q-btn rounded color="white" no-caps text-color="primary" style="width: 80%" class="q-ma-xs" label="Bleeding" @click="buttonSend('Bleeding')"/>
          <q-btn rounded color="white" no-caps text-color="primary" style="width: 80%" class="q-ma-xs" label="Drowning" @click="buttonSend('Drowning')"/>
          <q-btn rounded color="white" no-caps text-color="primary" style="width: 80%" class="q-ma-xs" label="Fractures" @click="buttonSend('Fractures')"/>
        </div>
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
        <q-toolbar>
          <q-form @submit="sendMessage('out')" class="full-width ">
            <div class="row">
            <q-input
              ref="newMessage"
              bg-color="white"
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
              <q-btn v-if="!recording" round dense flat @click="speechToText" icon="mic" color="white"  />
              <q-spinner-bars v-if="recording" color="white" size="2em"/>
            </div>
            </div>
          </q-form>
        </q-toolbar>
      </q-footer>
  </q-page>
</template>

<script>
const sdk = require("microsoft-cognitiveservices-speech-sdk");
import VueRecord from '@codekraft-studio/vue-record'
import Vue from 'vue'

Vue.use(VueRecord)

export default {
  
  name: 'PageIndex',
  data: () => ({
    botMessage: '',
    loading: false,
    youMessage: '',
    recording: false,
    history: [],
    messages: []
  }),
  computed: {
    user(){
      return { name: 'Davi', age: 7}
    }
  },
  mounted(){
    this.botMessage = 'Welcome to WeSafe! Whats your emergency?'
    this.synthesizeSpeech(this.botMessage)

  },
  methods: {

    speechToText(){
      try {
  
        const speechConfig = sdk.SpeechConfig.fromSubscription("80336464dd984e3489ab38cbb895823e", "eastus");
        let audioConfig = sdk.AudioConfig.fromDefaultMicrophoneInput();
        let recognizer = new sdk.SpeechRecognizer(speechConfig, audioConfig);
        
        console.log('Speak into your microphone.');
        this.recording = true
        recognizer.recognizeOnceAsync(result => {
            this.youMessage = result.text
            console.log(`RECOGNIZED: Text=${result.text}`);
            this.recording = false
            this.sendMessage('out')
            return
        });
        
      } catch (error) {
        console.log("recognition", error)
      }
    },

    
    async synthesizeSpeech(text) {
      let audioContext = new AudioContext()
      let playSoundBuffer = ''
      this.loading = true

      const speechConfig = sdk.SpeechConfig.fromSubscription("80336464dd984e3489ab38cbb895823e", "eastus");

      // Set the output format
      speechConfig.speechSynthesisOutputFormat = sdk.SpeechSynthesisOutputFormat.Riff24Khz16BitMonoPcm;

      const synthesizer = new sdk.SpeechSynthesizer(speechConfig, undefined);
      synthesizer.speakTextAsync(
          text,
          result => {
              // Interact with the audio ArrayBuffer data
              const audioData = result.audioData;
              this.loading = false
              console.log(text)
              
              console.log(`Audio data byte size: ${audioData.byteLength}.`)
              

              synthesizer.close();
              this.sendMessage('in')
              
          },
          error => {
              console.log(error);
              synthesizer.close();
          });
    },

    postMessage(){

    },

    buttonSend(message){
      this.youMessage = message
      this.sendMessage('out')
    },

    async bot(){
      let text = this.botMessage
      const input = JSON.stringify(this.history)
      console.log(input, "inputtt")

      if (this.history[0] == 0){
        if (input == '[0]') { text = "Is the victim you or someone else?"}
        else if (input == '[0,0]') { text = "Is the victim having difficulty breathing?"}
        else if (input == '[0,0,0]') {text = "Is the victim over 8 years old?"}
        else if (input == '[0,0,1]') {text = "Is the victim conscious?"}
        else if (input == '[0,0,1,0]') {text = "Is the victim over 8 years old?"}
        else if (input == '[0,0,1,1]') {text = "Is the victim over 8 years old?"}
        else if (input == '[0,0,0,1]') {text = "ANSWER1"}
        else if (input == '[0,0,0,0]') {text = "ANSWER2"}
        else if (input == '[0,0,1,0,0]') {text = "ANSWER6"}
        else if (input == '[0,0,1,0,1]') {text = "ANSWER5"}
        else if (input == '[0,0,1,1,0]') {text = "ANSWER2"}
        else if (input == '[0,0,1,1,1]') {text = "Is the victim Pregnant or Obese?"}
        else if (input == '[0,0,1,1,1,0]') {text = "ANSWER3"}
        else if (input == '[0,0,1,1,1,1]') {text = "ANSWER4"}

        else if (input === '[0,1]') {text = "Are you having trouble breathing?"}
        else if (input == '[0,1,0]') { 
          if (this.user.age.length == 0){
            text = "Are you over 8 years old?"
          }else {
            if (this.user.age > 8){
              this.history.push(1)
              text = "ANSWER1"
            }
            else{
              this.history.push(0)
              text = "ANSWER2+"

            }
          }
        }
        else if (input == '[0,1,1]') { 
          if (this.user.age.length == 0){
            text = "Are you over 8 years old?"
          }else {
            if (this.user.age > 8){
              this.history.push(1)
              text = "ANSWER3.2"
            }
            else{
              this.history.push(0)
              text = "Are you alone?"

            }
          }
        }
        else if (input === '[0,1,0,0]') {text = "ANSWER2+"}
        else if (input === '[0,1,0,1]') {text = "ANSWER1"}
        else if (input === '[0,1,1,1]') {text = "ANSWER3.2"}
        else if (input === '[0,1,1,0,0]') {text = "ANSWER2+"}
        else if (input === '[0,1,1,0,1]') {text = "CALL 911"}
    
      }else if (input[0] == 1){
          text = 'other.1'
      }
      this.botMessage = text

      await this.synthesizeSpeech(text)

            
    },

    async createHistory(msg){
      let message = msg.toLowerCase()

      console.log("Hmessage",message)

      if ((message === 'no' || message.includes('yes')) && this.history.length != 0){
        this.history.push(0)
      }
      else if ((message === 'yes' || message.includes('yes')) && this.history.length != 0){
        this.history.push(1)
      }

      else if ((message === 'choking' || message.includes('choking')) && this.history.length == 0){
        this.history.push(0)
      }
      else if (message === 'clear' || message.includes('clear')){
        this.clearAllMessages()
      }

      else if ((message === 'me' || message.includes('me')) && this.history.length != 0){
        this.history.push(1)
      }
      else if ((message === 'other' || message.includes('other')) && this.history.length != 0){
        this.history.push(0)
      }
      else{
        this.botMessage = "Sorry, I didn't understand. Can you repeat?"
      }
      
    },

    sendMessage(direction) {
      if (!this.youMessage && !this.botMessage) {
        return
      }
      if (direction === 'out') {
        this.messages.push({body: this.youMessage, author: 'you', id: this.messages.length})
        this.createHistory(this.youMessage)
        this.youMessage = ''
        this.bot()
      } else if (direction === 'in') {
        console.log(this.botMessage, "bot")
        this.messages.push({body: this.botMessage, author: 'bot', id: this.messages.length})
        this.botMessage = ''
      } 
      this.$refs.newMessage.focus();
      let messageDisplay = this.$refs.chatArea
      messageDisplay.setScrollPosition(messageDisplay.$el.scrollHeight + 1000, 1);
      
    },

    clearAllMessages() {
      this.messages = [
      {
        id: 0,
        body: "Welcome to WeSafe! Whats your emergency?",
        author: 'bot'
      },
    ]
      this.history = []
      this.synthesizeSpeech(this.messages[0].body)
    }
  }
}
</script>

<style>

  .headline {
    text-align: center;
    font-weight: 100;
    color: white;
  }
  .chat-area {
  /*   border: 1px solid #ccc; */
    height: calc(100vh - 100px);
    background: white;
    box-shadow: 2px 2px 5px 2px rgba(0, 0, 0, 0.3)
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
