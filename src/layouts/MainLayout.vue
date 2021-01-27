<template>
  <q-layout view="lHh Lpr lFf">
    <q-header elevated>
      <q-toolbar>
        <q-btn
          flat
          dense
          round
          :class="titleColor"
          icon="menu"
          aria-label="Menu"
          @click="leftDrawerOpen = !leftDrawerOpen"
        />

        <q-toolbar-title :class="titleColor">
          WeSafe
        </q-toolbar-title>

        <q-space />
        <div>
          <UserAvatar></UserAvatar>
        </div>

        
      </q-toolbar>
    </q-header>

    <q-drawer
      v-model="leftDrawerOpen"
      show-if-above
      bordered
      :content-class="bgColor"
    >
      <q-list>
        <q-item-label
          header
          :class="titleColor2"
        >
          Imagine Cup 2021
        </q-item-label>
        <EssentialLink
          v-for="link in essentialLinks"
          :key="link.title"
          v-bind="link"
        />
      </q-list>
    </q-drawer>

    <q-page-container>
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script>
import EssentialLink from 'components/EssentialLink.vue'

const linksData = [
  {
    title: 'Home',
    caption: '',
    icon: 'home',
    link: '/'
  },
  {
    title: 'My Account',
    caption: '',
    icon: 'account_circle',
    link: '/my-account'
  },
  
];

export default {
  name: 'MainLayout',
  components: { 
    EssentialLink,
    UserAvatar: () => import('components/user-avatar')
   },
  data: () => ({
    leftDrawerOpen: false,
      essentialLinks: linksData
  }),
  computed: {
    titleColor(){
      return this.$q.dark.isActive ? 'text-black' : 'text-white'
    },
    titleColor2(){
      return this.$q.dark.isActive ? 'text-white' : 'text-black'
    },
    bgColor(){
      return this.$q.dark.isActive ? 'bg-primary' : 'bg-grey-1'
    }
  },
  
}
</script>
