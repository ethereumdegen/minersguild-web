<template>

<PrimaryLayout>
 
  

   <div class="section  bg-white border-b-2 border-black">
     <div class="py-16 w-container">
        
       <div class="  px-2 ">
          <div class="text-lg font-bold mb-4"> Give to the Guild  </div>

          <div class="text-sm   mb-8"> Add 0xBTC to the Guild pool, distributing it to members equally  </div>
           
          <div  class=" " v-if="!connectedToWeb3">
              <NotConnectedToWeb3 />
          </div>

          <div  class=" px-4" v-if=" connectedToWeb3">
 
   
 
           <div class="mb-4 inline-block ">

           


             <div class="flex flex-row">
              <label   class="block text-md font-medium font-bold text-gray-800  ">Give Amount</label>
                  <div class="flex-grow"></div>
                <label   class="block text-xs font-xs text-blue-500  ">Balance:   {{tokenBalanceFormatted}}</label>

            </div>
              <div class="flex flex-row">
              <div class="w-f ">
                    <input type="number"   v-model="formInputs.currencyAmountFormatted"  class="text-gray-900 border-2 border-black font-bold px-4 text-xl focus:ring-indigo-500 focus:border-indigo-500 block w-full py-4 pl-7 pr-12   border-gray-300 rounded-md" placeholder="0.00">
                </div> 
                 
              </div>


                <div class="flex flex-row">
              <div class="bg-blue-200 p-4 w-full mt-4 ">
                    You will receive no guild shares for this contribution.
                </div> 
                 
              </div>
           
           
            </div>

 
 

          </div>

           <div class="py-4" v-if=" connectedToWeb3 && !submitComplete">
              
 
                 <div class="  p-4">
                     <div @click="giveClicked" class="select-none bg-blue-700 p-2 inline-block rounded hover:bg-blue-900 border-gray-800 border-2 cursor-pointer text-white" style=" text-shadow: 1px 1px #222;"> Give </div>
                </div> 

          </div>



          
       </div>
     </div>
   </div>


    

 

</PrimaryLayout>
</template>


<script>



import NotConnectedToWeb3 from './components/NotConnectedToWeb3.vue'

import Web3Plug from '../js/web3-plug.js'  
 import MathHelper from '../js/math-helper.js'  
 

import PrimaryLayout from './PrimaryLayout.vue';
  
import TabsBar from './components/TabsBar.vue';
import GenericTable from './components/GenericTable.vue';
import GenericDropdown from './components/GenericDropdown.vue';
  
const GuildContractABI = require('../contracts/MinersGuild.json')

 

var balanceInterval

export default {
  name: 'Stake',
  props: [],
  components: {PrimaryLayout, TabsBar, GenericTable, GenericDropdown,NotConnectedToWeb3},
  data() {
    return {
      web3Plug: new Web3Plug() , 
  
      formInputs:{},

      tokenBalanceFormatted: null,
       
      connectedToWeb3: false ,
      submitComplete:false
    }
  },

  created(){

 
    this.web3Plug.getPlugEventEmitter().on('stateChanged', async function(connectionState) {
        console.log('stateChanged',connectionState);
         
        this.activeAccountAddress = connectionState.activeAccountAddress
        this.activeNetworkId = connectionState.activeNetworkId
        this.connectedToWeb3 = this.web3Plug.connectedToWeb3()
        
            await this.fetchBalance()
         
      }.bind(this));
   this.web3Plug.getPlugEventEmitter().on('error', function(errormessage) {
        console.error('error',errormessage);
         
        this.web3error = errormessage
       
      }.bind(this));

      this.web3Plug.reconnectWeb()
    
 

  },
   mounted: async function () {

    
     
    balanceInterval = setInterval(this.fetchBalance,8000)
    
  }, 
  beforeDestroy(){
    clearInterval(balanceInterval)
  },
  methods: {

 
    async giveClicked(){
      console.log('start give ')


      let accountAddress = this.web3Plug.getActiveAccountAddress()

      let chainId = this.web3Plug.getActiveNetId()
      let guildContractAddress = this.web3Plug.getContractDataForNetworkID(chainId)['minersguild'].address

       let tokenContractAddress = this.web3Plug.getContractDataForNetworkID(chainId)['0xbitcoin'].address

      let currencyDecimals  = 8 
      let currencyAmountRaw = MathHelper.formattedAmountToRaw(this.formInputs.currencyAmountFormatted,currencyDecimals) 
 
      let tokenContract = this.web3Plug.getTokenContract( tokenContractAddress );

    

      let response = await tokenContract.methods.transfer( guildContractAddress, currencyAmountRaw ).send({from:  accountAddress })
    },


    async fetchBalance(){
       let chainId = this.web3Plug.getActiveNetId()
      let accountAddress = this.web3Plug.getActiveAccountAddress()

     
      let tokenContractAddress = this.web3Plug.getContractDataForNetworkID(chainId)['0xbitcoin'].address


      let tokenBalanceRaw = await  this.web3Plug.getTokenBalance(tokenContractAddress, accountAddress)

      console.log('tokenBalanceRaw', tokenBalanceRaw )

      this.tokenBalanceFormatted = parseFloat(   MathHelper.rawAmountToFormatted(tokenBalanceRaw  , 8  ) )
    }
          
  }
}
</script>
