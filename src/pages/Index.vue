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
        />
      </q-scroll-area>

      <q-footer elevated>
        <q-toolbar>
          <q-form @submit="sendMessage('out')" class="full-width">
            <q-input
              ref="newMessage"
              bg-color="white"
              rounded
              v-model="youMessage"
              placeholder="Message"
              class="full-width"
              dense
              outlined
            >
              <template>
                <q-btn round dense flat type="submit" icon="send" color="primary" />
              </template>
            </q-input>
          </q-form>
        </q-toolbar>
      </q-footer>
  </q-page>
</template>

<script>
export default {
  name: 'PageIndex',
  data: () => ({
    botMessage: '',
    youMessage: '',
    history: [],
    messages: [
      {
        id: 0,
        body: "Welcome to WeSafe! What's your emergency?",
        author: 'bot'
      },
    ]
  }),
  computed: {
    user(){
      return { name: 'Davi', age: 1}
    }
  },
  methods: {

    postMessage(){

    },

    bot(){
      let text = ''
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
        else if (input == '[0,0,1,1,1]') {text = "Is the victim Pregnant/Obese?"}
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

      this.sendMessage('in')

      
    },

    async createHistory(message){
      console.log("Hmessage",message)

      if (message === 'no'){
        this.history.push(0)
      }
      else if (message === 'yes'){
        this.history.push(1)
      }

      if (message === 'chocking'){
        this.history.push(0)
      }

      if (message === 'me'){
        this.history.push(1)
      }else if (message === 'other'){
        this.history.push(0)
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
        this.messages.push({body: this.botMessage, author: 'bot', id: this.messages.length})
        this.botMessage = ''
      } 
      this.$refs.newMessage.focus();
      let messageDisplay = this.$refs.chatArea
      messageDisplay.setScrollPosition(messageDisplay.$el.scrollHeight, 1);
      
    },

    clearAllMessages() {
      this.messages = []
      this.history = []
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
