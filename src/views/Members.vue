<template>

<div>

   <div class="section  bg-gray-200  border-b-2 border-black px-0 lg:px-1">

     <div class=" ">
        <Navbar 
        v-bind:web3Plug="web3Plug" 
       />
     </div>


   </div>

  

   <div class="section   border-b-2 border-black text-white" style="background:#222;">
     <div class="py-16 w-container">
        
       <div class="w-column">

         <div class="text-center">
           Total Guild Balance: {{getFormattedGuildBalance()}} 0xBTC

         </div>

         <div class="mt-8"> </div> 

          <div class="text-lg font-bold"> Guild Members  </div>
          
          
            
          <div class="text-xs">
            <ThiccTable 
              v-bind:labelsArray="[' ','shares', 'balance']"
              v-bind:rowsArray="shareRowsArray"
              v-bind:clickedRowCallback="onClickedRow"
            />
          </div>


          <div class="mt-16"> </div> 
           <div class="text-lg font-bold"> Sponsors  </div> 
          
            
          <div class="text-xs">
            <GenericTable 
              v-bind:labelsArray="[' ','donation']"
              v-bind:rowsArray="donationRowsArray"
              v-bind:clickedRowCallback="onClickedRow"
            />
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
import GenericTable from './components/GenericTable.vue';
 
import ThiccTable from './components/ThiccTable.vue';
 

import MathHelper from '../js/math-helper.js'

import StarflaskAPIHelper from '../js/starflask-api-helper.js'
 

export default {
  name: 'Members',
  props: [],
  components: {Navbar, Footer, TabsBar, GenericTable, ThiccTable, NotConnectedToWeb3},
  data() {
    return {
      web3Plug: new Web3Plug() , 
      activePanelId: null,
      selectedTab:"bids",
      shareRowsArray:[],
      donationRowsArray:[],

      guildBalances: {},
       
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
        this.currentBlockNumber = await this.web3Plug.getBlockNumber()

         
         
      }.bind(this));
   this.web3Plug.getPlugEventEmitter().on('error', function(errormessage) {
        console.error('error',errormessage);
         
        this.web3error = errormessage
       
      }.bind(this));

      this.web3Plug.reconnectWeb()
    
 

  },
  mounted: async function () {
    
      await this.fetchGuildBalances()
      await this.fetchReserveBalances()
      await this.fetchDonations()
  }, 
  methods: {
          async fetchGuildBalances(){

            let apiURI = 'https://api.starflask.com/api/v1/testapikey'
            let inputData = {requestType: 'ERC20_balance_by_owner', input: { account:'0x167152a46e8616d4a6892a6afd8e52f060151c70' } } 
            let results = await StarflaskAPIHelper.resolveStarflaskQuery(apiURI ,  inputData   )
            console.log('results',results)
            
            let balances = results.output 


            this.guildBalances = {}

            for(let balance of balances){
               if(balance.amount > 0){
                 this.guildBalances[balance.contractAddress] = balance.amount
                
               }
              }

          //this.guildBalances.sort( (a, b) => b.amount - a.amount )
                console.log('guild balances', this.guildBalances)
              this.$forceUpdate()
          },
           async fetchReserveBalances(){

            let apiURI = 'https://api.starflask.com/api/v1/testapikey'
            let inputData = {requestType: 'ERC20_balance_by_token', input: { token:'0x657223e3fdf539d92c40664db340097d5d6bd9f5' } } 
            let results = await StarflaskAPIHelper.resolveStarflaskQuery(apiURI ,  inputData   )
            console.log('results',results)
            
            let balances = results.output 

            let totalShares = 0

            for(let balance of balances){
              if(balance.amount > 0){

              totalShares += balance.amount 
              }
            }
          

            console.log('totalShares',totalShares)

            let primaryTokenAddress = web3utils.toChecksumAddress('0xb6ed7644c69416d67b522e20bc294a9a9b405b31')

            let primaryGuildBalance = this.guildBalances[primaryTokenAddress]

            this.shareRowsArray = [] 

            for(let balance of balances){
               if(balance.amount > 0){

                 let balanceFromShare = parseFloat( primaryGuildBalance * (balance.amount/totalShares)) 

                this.shareRowsArray.push(
                  {accountAddress: balance.accountAddress,  
                  amount: MathHelper.rawAmountToFormatted(balance.amount,8) ,
                  balanceFromShare: MathHelper.rawAmountToFormatted(balanceFromShare,8)
                    })
           
               }
              }

          this.shareRowsArray.sort( (a, b) => b.amount - a.amount )

          },
            async fetchDonations(){

            let apiURI = 'https://api.starflask.com/api/v1/testapikey'
            let inputData = {requestType: 'ERC20_transferred_to', input: { to:'0x167152a46e8616d4a6892a6afd8e52f060151c70' } } 
            let results = await StarflaskAPIHelper.resolveStarflaskQuery(apiURI ,  inputData   )
            console.log('results',results)
            

            let _0xBTCAddress = '0xb6ed7644c69416d67b522e20bc294a9a9b405b31'.toLowerCase()


            let donations = results.output 

            let addressesWithReserve = this.shareRowsArray.map(r => r.accountAddress )

            this.donationRowsArray = [] 

            for(let donation of donations){
               if(donation.amount > 0 && donation.contractAddress.toLowerCase() == _0xBTCAddress && !addressesWithReserve.includes(donation.from)){
                this.donationRowsArray.push({from: donation.from, amount: MathHelper.rawAmountToFormatted(donation.amount,8), token: donation.contractAddress   })
           
               }
              }

          this.donationRowsArray.sort( (a, b) => b.amount - a.amount )
          console.log('this.donationRowsArray',this.donationRowsArray)
          },

          onClickedRow(row){
              
             let accountAddress  = row.accountAddress
             if(!accountAddress) accountAddress= row.from 

            window.location.href = this.web3Plug.getExplorerLinkForAddress(accountAddress)
          },

          getFormattedGuildBalance(){
            let primaryTokenAddress = web3utils.toChecksumAddress('0xb6ed7644c69416d67b522e20bc294a9a9b405b31')

            let primaryBalance = this.guildBalances[primaryTokenAddress]
            return parseFloat( MathHelper.rawAmountToFormatted(primaryBalance,8) )
          }
  }
}
</script>
