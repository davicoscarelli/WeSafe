
<template>
  <q-page class="flex flex-center">
    <img
      alt="Quasar logo"
      src="~assets/quasar-logo-full.svg"
    >
  </q-page>
</template>

<script>

import { DirectLine } from 'botframework-directlinejs';


export default {
  data: () => ({

  }),
  created() {
    this.send()
    this.hear()
      
  },
  methods: {
    send(){
      const directLine = new DirectLine({
          secret: 'r-S2FB1ly_A.8n_nn2bKBafpoAa-SRftSNp1ScFf35g2ur_zbRfvRNA',
          token: 'r-S2FB1ly_A.afSgtuyKBph-ZlZm5zT_ocNHfoDHZls062iHMTBCIBI',
      });

      directLine.postActivity({
          from: { id: 'myUserId', name: 'myUserName' }, 
          type: 'message',
          text: 'ralado'
      }).subscribe(
          id => {
            console.log("Posted activity, assigned ID ", id)
            directLine.activity$
                .subscribe(
                    activity => console.log("received activity ", activity)
                );
            
          },
          error => console.log("Error posting activity", error)
      );
    },
    hear(){
      const directLine = new DirectLine({
          secret: 'r-S2FB1ly_A.8n_nn2bKBafpoAa-SRftSNp1ScFf35g2ur_zbRfvRNA',
          token: 'r-S2FB1ly_A.afSgtuyKBph-ZlZm5zT_ocNHfoDHZls062iHMTBCIBI',
      });

      directLine.activity$
        .subscribe(
            activity => console.log("received activity ", activity)
        );
    }
    
  }
}
</script>


