<template>

<div>

   <div class="section bg-gray-200  border-b-2 border-black  lg:px-1 ">

     <div class=" ">
        <Navbar 
        v-bind:web3Plug="web3Plug" 
       />
     </div>


   </div>

  

   <div class="section  bg-white border-b-2 border-black">
     <div class="py-16 w-container">
        
       <div class="  px-2 ">
          <div class="text-lg font-bold mb-4"> Unstake Guild Shares    </div>

           
          <div  class=" " v-if="!connectedToWeb3">
              <NotConnectedToWeb3 />
          </div>

          <div  class=" px-4" v-if=" connectedToWeb3">
 
             


            <div class="mb-4 ">
              <label   class="block text-md font-medium font-bold text-gray-800  ">Shares Balance</label>

              <div class="flex flex-row">
              <div class="w-1/2 ">
                   {{sharesBalance}} Shares
                </div> 
                 
              </div>
           
            </div>

             <div class="mb-4 ">
              <label   class="block text-md font-medium font-bold text-gray-800  ">Max Output for Shares</label>

              <div class="flex flex-row">
              <div class="w-1/2 ">
                   {{expectedOutput}} 0xBTC
                </div> 
                 
              </div>
           
            </div>


              
           <div class="mb-4 block">
             <div class="flex flex-row"> 
                <label   class="  text-md font-medium font-bold text-gray-800  inline ">Shares Withdraw Amount</label>
                <label @click="setMax()"  class="  text-md font-medium  select-none text-blue-400 hover:text-blue-500 cursor-pointer inline mx-4 ">max</label>
                
               </div> 
              <div class="flex flex-col">
              <div class="w-1/2 ">
                    <input type="number"   v-model="formInputs.currencyAmountFormatted" v-on:blur="recalculateOutput()" class="text-gray-900 border-2 border-black font-bold px-4 text-xl focus:ring-indigo-500 focus:border-indigo-500 block w-full py-4 pl-7 pr-12   border-gray-300 rounded-md" placeholder="0.00">
                </div> 
                 
              </div>
           
            </div>

             <div class="mb-4 block" v-if="formInputs.currencyAmountFormatted && formInputs.currencyAmountFormatted>0">
               You will receive: {{  getEstimatedOutput(formInputs.currencyAmountFormatted)  }} 0xBTC
              </div>
 



          </div>

           <div class="py-4" v-if=" connectedToWeb3 && !submitComplete">
             
 
  

                 <div class="  p-4">
                     <div @click="withdrawClicked" class="select-none bg-blue-700 p-2 inline-block rounded hover:bg-blue-900 border-gray-800 border-2 cursor-pointer text-white" style=" text-shadow: 1px 1px #222;"> Withdraw </div>
                </div> 


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
 import MathHelper from '../js/math-helper.js'  
 

import Navbar from './components/Navbar.vue';
 
import Footer from './components/Footer.vue';
import TabsBar from './components/TabsBar.vue';
import GenericTable from './components/GenericTable.vue';
import GenericDropdown from './components/GenericDropdown.vue';
  
const GuildContractABI = require('../contracts/MinersGuild.json')

import web3utils from 'web3-utils'


var balanceInterval

export default {
  name: 'Stake',
  props: [],
  components: {Navbar, Footer, TabsBar, GenericTable, GenericDropdown,NotConnectedToWeb3},
  data() {
    return {
      web3Plug: new Web3Plug() , 
  
      formInputs:{},
      sharesBalance: 0,
      expectedOutput:0,
     
       
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
        
            this.updateBalances()
         
      }.bind(this));
   this.web3Plug.getPlugEventEmitter().on('error', function(errormessage) {
        console.error('error',errormessage);
         
        this.web3error = errormessage
       
      }.bind(this));

      this.web3Plug.reconnectWeb()
    
 

  },
  mounted: function () {

    let chainId = this.web3Plug.getActiveNetId()
     
    this.updateBalances()

    balanceInterval = setInterval(this.updateBalances,8000)
    
  }, 
  methods: {

    async updateBalances(){
        let accountAddress = this.web3Plug.getActiveAccountAddress()

      let chainId = this.web3Plug.getActiveNetId()
        if(!chainId) chainId = 1;

       let stakedTokenContractAddress = this.web3Plug.getContractDataForNetworkID(chainId)['0xbitcoin'].address



      let guildContractAddress = this.web3Plug.getContractDataForNetworkID(chainId)['minersguild'].address

       let tokenContractAddress = this.web3Plug.getContractDataForNetworkID(chainId)['guildreserve'].address

      
      
      let tokenContract = this.web3Plug.getTokenContract( tokenContractAddress );
      let guildContract = this.web3Plug.getCustomContract( GuildContractABI,guildContractAddress );


      if(accountAddress &&  web3utils.isAddress(accountAddress)  ){
        
      


      let currencyDecimals = 8
      let sharesBalanceRaw = await tokenContract.methods.balanceOf( accountAddress ).call()

      let expectedOutputRaw = await guildContract.methods._vaultOutputAmount( sharesBalanceRaw, stakedTokenContractAddress ).call()
 
      this.sharesBalance =  MathHelper.rawAmountToFormatted(sharesBalanceRaw,currencyDecimals) 
      this.expectedOutput =  MathHelper.rawAmountToFormatted(expectedOutputRaw,currencyDecimals) 



      }


    },

    setMax(){
      console.log('set max', this.sharesBalance)
       this.formInputs.currencyAmountFormatted =  this.sharesBalance 

       this.$forceUpdate()
    },


    recalculateOutput(){

      this.$forceUpdate()
    },


      getEstimatedOutput( amountFormatted ){

      let currencyDecimals = 8

      let rawInput =  MathHelper.formattedAmountToRaw(amountFormatted,currencyDecimals) 

      

       let ratio =   this.expectedOutput / this.sharesBalance 

   
 

        let rawOutput = rawInput * ratio

        console.log('rawOutput',rawOutput , this.expectedOutput )
       

        let formattedOutput = MathHelper.rawAmountToFormatted ( rawOutput , 8 )
         if(formattedOutput >  this.expectedOutput){
          formattedOutput =  this.expectedOutput
        }

      return formattedOutput

    },
 
    async withdrawClicked(){
      console.log('start withdraw ')


      let accountAddress = this.web3Plug.getActiveAccountAddress()

      let chainId = this.web3Plug.getActiveNetId()
      let guildContractAddress = this.web3Plug.getContractDataForNetworkID(chainId)['minersguild'].address

       let tokenContractAddress = this.web3Plug.getContractDataForNetworkID(chainId)['0xbitcoin'].address

      let currencyDecimals  = 8 
      let currencyAmountRaw = MathHelper.formattedAmountToRaw(this.formInputs.currencyAmountFormatted,currencyDecimals) 
 
      let tokenContract = this.web3Plug.getTokenContract( tokenContractAddress );
      let guildContract = this.web3Plug.getCustomContract( GuildContractABI,guildContractAddress );
 

      let response = await guildContract.methods.unstakeCurrency(  currencyAmountRaw,  tokenContractAddress ).send({from:  accountAddress })
    }
          
  }
}
</script>
