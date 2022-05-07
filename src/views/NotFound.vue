<template>

<PrimaryLayout>

   

   


      <div class=" section  mb-4">
        <div class="autospacing w-container">
          <div> File not found! </div>
          <router-link to="/"> -> Go Home</router-link>

          </div>
      </div>
 

 

</PrimaryLayout>
</template>


<script>



import Web3Plug from '../js/web3-plug.js' 


import PrimaryLayout from './PrimaryLayout.vue';
  
 



export default {
  name: 'NotFound',
  props: [],
  components: {PrimaryLayout},
  data() {
    return {
      web3Plug: new Web3Plug() ,
      bidPacketData: {} 
    }
  },
  created: function () {
    this.web3Plug.reconnectWeb()
    this.web3Plug.getPlugEventEmitter().on('stateChanged', function(connectionState) {
        console.log('stateChanged',connectionState);
         
        this.activeAccountAddress = connectionState.activeAccountAddress
        this.activeNetworkId = connectionState.activeNetworkId
         
      }.bind(this));
   this.web3Plug.getPlugEventEmitter().on('error', function(errormessage) {
        console.error('error',errormessage);
         
        this.web3error = errormessage
        
      }.bind(this));
   
      this.fetchPacketData(this.$route.params.signature)
  }, 
  methods: {
      
  }
}
</script>
