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
              <!-- <q-btn v-if="!recording" round dense flat @click="speechToText" icon="mic" color="white"  /> -->
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


export default {
  
  name: 'PageIndex',
  data: () => ({
    botMessage: '',
    loading: false,
    youMessage: '',
    audioContext: null,
    recording: false,
    history: [],
    messages: [],
    options: [['Yes', 'No'], ['Me', 'Someone else']],
    showOptions: false,
    optionsType: null
  }),
  computed: {
    user(){
      return { name: 'Davi', age: 7}
    }
  },
  created(){
    
  },
  mounted(){
    this.botMessage = 'Welcome to WeSafe! Whats your emergency?'
    // this.audioContext = new AudioContext()
    this.synthesizeSpeech(this.botMessage)

  },
  methods: {

    // speechToText(){
    //   try {
  
    //     const speechConfig = sdk.SpeechConfig.fromSubscription("80336464dd984e3489ab38cbb895823e", "eastus");
    //     let audioConfig = sdk.AudioConfig.fromDefaultMicrophoneInput();
    //     let recognizer = new sdk.SpeechRecognizer(speechConfig, audioConfig);
    //     const phraseList = sdk.PhraseListGrammar.fromRecognizer(recognizer);
    //     phraseList.addPhrase(['choking','no','yes','clear','me','other','someone','else']);
        
    //     console.log('Speak into your microphone.');
    //     this.recording = true
    //     recognizer.recognizeOnceAsync(result => {
    //         this.youMessage = result.text
    //         console.log(`RECOGNIZED: Text=${result.text}`);
    //         this.recording = false
    //         this.sendMessage('out')
    //         this.showOptions = false 
    //         return
    //     });
        
    //   } catch (error) {
    //     console.log("recognition", error)
    //   }
    // },

    
    async synthesizeSpeech(text) {
      
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
              if (this.optionsType != null) this.showOptions = true

              this.sendMessage('in')

              
          },
          error => {
              console.log(error);
              synthesizer.close();
          });
    },

    // postMessage(){

    // },

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
        else if (input == '[0,0,0,1]') { message = {text: "ANSWER1", optionsType: null}}
        else if (input == '[0,0,0,0]') { message = {text: "ANSWER2", optionsType: null}}
        else if (input == '[0,0,1,0,0]') { message = {text: "ANSWER6", optionsType: null}}
        else if (input == '[0,0,1,0,1]') { message = {text: "ANSWER5", optionsType: null}}
        else if (input == '[0,0,1,1,0]') { message = {text: "ANSWER2", optionsType: null}}
        else if (input == '[0,0,1,1,1]') { message = {text: "Is the victim Pregnant or Obese?", optionsType: 0}}
        else if (input == '[0,0,1,1,1,0]') {message = {text: "ANSWER3", optionsType: null}}
        else if (input == '[0,0,1,1,1,1]') {message = {text: "ANSWER4", optionsType: null}}

        else if (input === '[0,1]') { message = {text: "Are you having trouble breathing?", optionsType: 0}}
        else if (input == '[0,1,0]') { 
          if (this.user.age.length == 0){
            message = {text: "Are you over 8 years old?", optionsType: 0}
          }else {
            if (this.user.age > 8){
              this.history.push(1)
              message = {text: "ANSWER1", optionsType: null}
            }
            else{
              this.history.push(0)
              message = {text: "ANSWER2+", optionsType: null}

            }
          }
        }
        else if (input == '[0,1,1]') { 
          if (this.user.age.length == 0){
            message = {text: "Are you over 8 years old?", optionsType: 0}
          }else {
            if (this.user.age > 8){
              this.history.push(1)
              message = {text: "ANSWER3.2", optionsType: null}
            }
            else{
              this.history.push(0)
              message = {text: "Are you alone?", optionsType: 0}

            }
          }
        }
        else if (input === '[0,1,0,0]') {message = {text: "ANSWER2+", optionsType: null}}
        else if (input === '[0,1,0,1]') {message = {text: "ANSWER1", optionsType: null}}
        else if (input === '[0,1,1,1]') {message = {text: "ANSWER3.2", optionsType: null}}
        else if (input === '[0,1,1,0,0]') {message = {text: "ANSWER2+", optionsType: null}}
        else if (input === '[0,1,1,0,1]') {message = {text: "CALL 911", optionsType: null}}
    
      }else if (input[0] == 1){
          message = {text: 'other.1', optionsType: 0}
      }
      this.botMessage = message.text
      this.optionsType = message.optionsType
      await this.synthesizeSpeech(message.text)

            
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
        this.messages.push({body: this.youMessage, author: 'you', id: this.messages.length})
        this.createHistory(this.youMessage)
        this.youMessage = ''
        this.bot()
      } else if (direction === 'in') {
        console.log(this.botMessage, "bot")
        this.messages.push({body: this.botMessage, author: 'bot', id: this.messages.length})
        this.botMessage = ''
      } 
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
    color: white;
  }
  .chat-area {
  /*   border: 1px solid #ccc; */
    height: calc(90vh - 100px);
    background: white;
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
