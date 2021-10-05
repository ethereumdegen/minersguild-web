<template> 
<div class="   pt-8   pb-2 mb-12 text-white" style="min-height:300px">
            
            <div class="text-lg"> Guild Sponsors </div>    
 


            <div class="  flex flex-col" >
            <div class="my-4 inline-block" >
                
                <a class="border-2 border-gray-500 p-2 hover:bg-gray-600" href="http://tokenminingpool.com" target="_blank" > 
                    <img src="@/assets/images/tokenminingpool.png" class="inline-block" width="24px" />
                    <div class="inline-block ml-2 text-white ">Token Mining Pool</div>
                    
                    
              </a> 
               </div> 
               <div class="my-4 inline-block" >

                  <a class="border-2 border-gray-500 p-2 hover:bg-gray-600" href="https://treasures.dmaddestudios.com/" target="_blank" > 
                    <img src="@/assets/images/treasure.png" class="inline-block" width="24px" />
                    <div class="inline-block ml-2 text-white ">0xBitcoin Treasures</div>
                    
                    
              </a> 
              </div> 

                  <div class="my-4 inline-block" >

                  <a class="border-2 border-gray-500 p-2 hover:bg-gray-600" href="https://0xstats.0xbtc.io" target="_blank" > 
                    <img src="@/assets/images/0xstats.png" class="inline-block" width="24px" />
                    <div class="inline-block ml-2 text-white ">0xStats NFT</div>
                    
                    
              </a> 
              </div> 

 
                
           </div> 


 </div>
 </template>



<script>
    export default {
    name: 'FrontPageMedia',
    props: [ ],
    data() {
        return {
            startTime: 0
        }
    },
    created(){
        this.startTime= Date.now()

        setInterval(this.update,500)
    },
    methods: {
         update(){
             this.$forceUpdate()
         },
         getTimeElapsed(){
             return Date.now() - this.startTime
         }
    }
    }
</script>
