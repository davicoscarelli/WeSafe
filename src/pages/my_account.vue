<style lang="sass" scoped>

.plan-title
  font-size: 16px
  font-weight: 900

.title
  font-size: 18px
.info_text
  font-size: 14px
.psg-round
  border-radius: 20px

.name-user
    font-weight: 900
    font-size: 24px
    margin: 10px 0


</style>

<template>
  <q-page >
    <div class="row q-pa-xl">
      <div class=" col-12">
        <div class="row q-col-gutter-md">
          <q-form
            class="col-12 q-mb-xl q-pa-md"
            @submit="updateUser()"
          >
            <div class="row q-col-gutter-md">

              <div class="col-12 q-pt-lg">
                <h5 class="title text-bold text-primary text-left q-ma-none">
                  Personal Info
                </h5>
              </div>
              
              <div class="col-12">
                <q-input
                  id="name"
                  rounded
                  outlined
                  maxlength="60"
                  dense
                  @input="namechanged = true"
                  v-model="form.name"
                  placeholder="Name"
                />
              </div>

              <div class="col-12 col-md-6">
                <q-input
                  class="q-mt-sm"
                  id="email"
                  rounded
                  outlined
                  dense
                  v-model="form.email"
                  placeholder="Email"
                />
              </div>

              <div class="col-12 col-md-6">
                <q-input
                  class="q-mt-sm"
                  id="phone"
                  rounded
                  outlined
                  dense
                  mask="(##) # ####-####"
                  v-model="form.phone"
                  placeholder="Contact Phone"
                />
              </div>
              
              <div class="col-12 col-md-6">
                <q-input
                  class="q-mt-sm"
                  outlined
                  rounded
                  dense
                  v-model="form.birthdate"
                  mask="##/##/####"
                  placeholder="Birthdate"
                >
                  <template v-slot:append>
                    <q-icon name="event" class="cursor-pointer">
                      <q-popup-proxy
                        ref="qDateProxy"
                        transition-show="scale"
                        transition-hide="scale"
                      >
                        <q-date
                          v-model="form.birthdate"
                          @input="() => $refs.qDateProxy.hide()"
                          mask="DD/MM/YYYY"
                        />
                      </q-popup-proxy>
                    </q-icon>
                  </template>
                </q-input>
              </div>

              <div class="col-12 q-pt-xl">
                <h5 class="title text-bold text-primary text-left q-ma-none">
                  Health Information
                </h5>
              </div>

              <div class="col-12 col-md-8">
                <q-input
                  id="allergies"
                  rounded
                  maxlength="60"
                  outlined
                  dense
                  v-model="form.allergies"
                  placeholder="Allergies"
                />
              </div>

              

              <div class="col-12 col-md-1 text-center" >
                <q-btn
                  type="submit"
                  :loading="loading"
                  class=" q-my-md q-px-md text-bold"
                  no-caps
                  unelevated
                  rounded
                  color="primary"
                  label="Update"
                  size="16px"
                />
              </div>
            </div>
          </q-form>
        </div>
      </div>
    </div>


  </q-page>
</template>

<script>

const defaultForm = {
  name: '',
  email: '',
  phone: '',
  birthdate: '',
  allergies: ''
}

export default {
  name: 'minha-conta',
  data: () => ({
    form: { ...defaultForm },
    loading: false,

  }),

  methods: {

    async updateUser() {
      localStorage.user = JSON.stringify(this.form)
      this.getUserData()
      window.location.reload()
      
    },

    async getUserData() {
      let user = {}
      if (localStorage.user) user = JSON.parse(localStorage.user)
      Object.assign(this.form, user)
      this.$q.loading.hide()
      
  }
  },
  
  beforeMount() {
    this.$q.loading.show({
          spinnerColor: 'white',
          thickness:"10",
          spinnerSize: 140,
          backgroundColor: 'primary',
          messageColor: 'white'
        })
    this.timer = setTimeout(() => {
      this.getUserData()
    }, 500)
    
  },
}
</script>
