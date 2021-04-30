<template>

<div>

   <div class="section  bg-gray-200  border-b-2 border-black px-0 lg:px-1">

     <div class=" ">
        <Navbar 
        v-bind:web3Plug="web3Plug" 
       />
     </div>


   </div>

  

   <div class="section   border-b-2 border-black text-white " style="background:#222;">
     <div class="py-16 w-container">
        
       <div class="w-column overflow-x-auto">

         <div class="text-center">
          <div class="text-lg">  Miners Guild Reserve Collection </div> 
         
         </div>

         <div class="mt-8"> </div> 

          <div v-for="work of collectedWorks" class="m-4 p-4 ">
             
            <img v-bind:src="getUrlForIpfsHash(work.ipfsHash)"  width="300px" /> 
          </div>
          
          
             <div class="text-center">
         
             <a v-bind:href="'https://etherscan.io/address/'+vaultWalletAddress" class="text-xs"> {{vaultWalletAddress}} </a>
         </div>
        </div>
         
         
     </div>
   </div>


    


    
  <Footer/>

</div>
</template>


<script>



import NotConnectedToWeb3 from './components/NotConnectedToWeb3.vue'

import Web3Plug from '../js/web3-plug.js'  

import web3utils from 'web3-utils' 

import Navbar from './components/Navbar.vue';
 
import Footer from './components/Footer.vue';
import TabsBar from './components/TabsBar.vue';
 
import MathHelper from '../js/math-helper.js'

import StarflaskAPIHelper from '../js/starflask-api-helper.js'


const artworkLookupTable= {

  "0x6c10511DdEA5f3ED38A0163224198e37b81525bC": {
    6: 'QmPUXMy83rS783UHozGPPdFBncxLQ1LEvHyTP4ay1aEBqn'
  }

}
 
export default {
  name: 'Collection',
  props: [],
  components: {Navbar, Footer, TabsBar,  NotConnectedToWeb3},
  data() {
    return {
      web3Plug: new Web3Plug() , 
     
    

      collectedWorks:[],
      vaultWalletAddress:"0xA9333994CcDDcd6a1219Bbc5F52b60c49d1C1dB0",
       
       
      connectedToWeb3: false,
      currentBlockNumber: 0
    }
  },

  created(){

 
    this.web3Plug.getPlugEventEmitter().on('stateChanged', async function(connectionState) {
        console.log('stateChanged',connectionState);
         
        this.activeAccountAddress = connectionState.activeAccountAddress
        this.activeNetworkId = connectionState.activeNetworkId
        this.connectedToWeb3 = this.web3Plug.connectedToWeb3()
      
         
         
      }.bind(this));
   this.web3Plug.getPlugEventEmitter().on('error', function(errormessage) {
        console.error('error',errormessage);
         
        this.web3error = errormessage
       
      }.bind(this));

      this.web3Plug.reconnectWeb()
    
 

  },
  mounted: async function () {
    
      await this.fetchCollectedWorks()
     
  }, 
  methods: {
          async fetchCollectedWorks(){

            let apiURI = 'https://api.starflask.com/api/v1/testapikey'
            let inputData = {requestType: 'ERC721_balance_by_owner', input: { publicAddress:this.vaultWalletAddress } } 
            let results = await StarflaskAPIHelper.resolveStarflaskQuery(apiURI ,  inputData   )
           
            let worksArray = results.output 

            this.collectedWorks = [] 

            for(let typeOfWork of worksArray){
            

              for(let tokenId of typeOfWork.tokenIds){

                let ipfsHash = artworkLookupTable[typeOfWork.contractAddress][tokenId]

                this.collectedWorks.push( { tokenId: tokenId, contractAddress: typeOfWork.contractAddress, ipfsHash: ipfsHash  }  )
              }

            }
             console.log(this.collectedWorks)



            
              this.$forceUpdate()
          },
          getUrlForIpfsHash(hash){


            return 'https://cloudflare-ipfs.com/ipfs/'.concat(hash)
          }

         
  }
}
</script>
