<template>
  <q-page class="">
     <q-scroll-area ref="chatArea" class="chat-area">
      <q-chat-message
        :key="message.id" v-for="message in messages"
        :text="[message.body]"
        :sent="message.author == 'bot' ?  false : true"
        :bg-color="message.author == 'bot' &&  'primary'"
        :text-color="message.author == 'bot' &&  'white'"
        class="q-px-md q-pt-sm"
      />
      <!-- <p :key="message.id" v-for="message in messages" class="message q-ma-md">
        {{ message.body }}
      </p> -->
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
    bobMessage: '',
    youMessage: '',
    messages: [
      {
        id: 0,
        body: "Welcome to WeSafe! What's your emergency?",
        author: 'bot'
      },
    ]
  }),
  methods: {
    sendMessage(direction) {
      if (!this.youMessage && !this.bobMessage) {
        return
      }
      if (direction === 'out') {
        this.messages.push({body: this.youMessage, author: 'you'})
        this.youMessage = ''
      } else if (direction === 'in') {
        this.messages.push({body: this.bobMessage, author: 'bot'})
        this.bobMessage = ''
      } else {
        alert('something went wrong')
      }
      
        let messageDisplay = this.$refs.chatArea
        messageDisplay.setScrollPosition(messageDisplay.$el.scrollHeight, 1);
      
    },
    clearAllMessages() {
      this.messages = []
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
  .message {
    width: 100%;
    border-radius: 10px;
    padding: .5em;
  /*   margin-bottom: .5em; */
    font-size: 1em;
  }
  .message-out {
    background: #407FFF;
    color: white;
  }
  .message-in {
    background: #F1F0F0;
    color: black;
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
