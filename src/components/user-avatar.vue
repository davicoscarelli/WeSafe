<style lang="sass" scoped>
  .user-menu-content 
    width: 270px
    max-width: 300px
    font-size: 12px


</style>

<template>
  <div class="row q-gutter-sm" >
  
    <q-avatar class="cursor-pointer">
      <img src="images/avatar.png" />
      <q-menu max-width="320px">
        <div class="q-py-md">
          <div class="user-menu-content">
            <q-list>
              <q-item-label class="q-pb-md q-px-md text-subtitle1">
                {{ user.name }}
              </q-item-label>
              <q-separator />

              <q-item clickable v-ripple @click="changeTheme()">
                <q-item-section avatar>
                  <q-icon
                    :color="iconColor"
                    :name="`mdi-brightness-${$q.dark.isActive ? '7' : '4'}`"
                  />
                </q-item-section>
                <q-item-section>
                  {{ `${$q.dark.isActive ? 'Light Mode' : 'Dark Mode'}` }}
                </q-item-section>
              </q-item>
              <q-item
                active-class="text-positive"
                clickable
                v-ripple
                to="/my-account"
                exact
              >
                <q-item-section avatar>
                  <q-icon :color="iconColor" name="mdi-account" />
                </q-item-section>
                <q-item-section>My Account</q-item-section>
              </q-item>

              <q-item clickable @click="logout" v-ripple>
                <q-item-section avatar>
                  <q-icon :color="iconColor" name="mdi-exit-run" />
                </q-item-section>
                <q-item-section>Logout</q-item-section>
              </q-item>
            </q-list>
          </div>
        </div>
      </q-menu>
    </q-avatar>

  </div>
  
</template>

<script>
import { Notify } from 'quasar'

export default {
  computed: {
    user() {
      let user = {name: "Test User"}
      if (!localStorage.user){
        localStorage.user = JSON.stringify(user)
      }else{
        user = JSON.parse(localStorage.user)
      }

      return user
    },
    iconColor() {
      return this.$q.dark.isActive ? 'white' : 'primary'
    }
  },
  methods:{
    changeTheme() {
      const dark = !this.$q.dark.isActive
      this.$q.dark.set(dark)
      localStorage.setItem('theme', JSON.stringify({ dark }))
    },
    async logout() {
      localStorage.removeItem('user')
      localStorage.removeItem('theme')
      this.showAlert({
        message: `See you soon ${this.user.name}! 😉`,
        icon: 'check',
        color: 'positive'
      })
      window.location.reload()
    },
    showAlert({ message, icon, color }) {
      Notify.create({
        progress: true,
        timeout: 3000,
        message,
        icon,
        color,
        position: 'bottom'
      })
    }
  },
  created(){
    const theme = JSON.parse(localStorage.theme)
    this.$q.dark.set(theme.dark)
    
  }
  
    
}
</script>
