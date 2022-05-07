<template>

<PrimaryLayout>

  
  

   <div class="section  border-b-2 border-black" style="background:#1d1d1d;">
     <div class=" ">
       <div class=" ">

       </div>
       <div class="  flex lg:flex-row flex-col  ">
 
         <div class="  md:w-1/2  w-full mt-8 py-8  px-1  text-center">

            <div class="hidden text-white text-xl my-4">  Neutral grass-roots organization.  </div> 


        <div>
            <router-link to="/members"  class="select-none no-underline bg-teal-700 mb-4 p-2 inline-block rounded hover:bg-blue-900 border-gray-800 border-2 cursor-pointer text-blue-100" style=" text-shadow: 1px 1px #222;">View Members List  </router-link> 
        </div>

        <div>
            <router-link to="/stake"  class="select-none no-underline bg-teal-700 mb-4 p-2 inline-block rounded hover:bg-blue-900 border-gray-800 border-2 cursor-pointer text-blue-100" style=" text-shadow: 1px 1px #222;">Stake 0xBTC to Earn  </router-link> 
        </div>

            <div>
            <a href="https://snapshot.org/#/0xbtcvault.eth" target="_blank"  class="select-none no-underline bg-gray-700 mb-16 p-2 inline-block rounded hover:bg-blue-900 border-gray-800 border-2 cursor-pointer text-blue-100" style=" text-shadow: 1px 1px #222;">Vote on Snapshot.org </a> 
        </div>

 
      
            <img src="@/assets/images/hero-dark2.png" class="pl-4" style="margin:0 auto;" />

         </div>
         <div class="   md:w-1/2  w-full  text-center ">
           
            
           <FrontPageMedia />


         </div>
       </div>
     </div>
   </div>


    <div class="section  text-white  border-b-2 border-black bg-gray-800 " >
     <div class="w-container  ">

         

          <div class=" w-2/3  mt-8 py-8" style="margin: 0 auto;">
           
                <div class="text-2xl text-center"> How It Works </div>

                <ul class=" "> 
                    <li class="my-4"> 1. Stake 0xBTC in the Guild Contract Pool </li>
                    <li class="my-4" > 2. Neutral Dapps/Contracts donate their fees [in execution flow] to the Pool to earn Community Goodwill </li>
                    <li class="my-4" > 3. Unstake at any time to withdraw <span class="text-purple-300"> your original deposit + accrued donations </span>  </li>
               </ul>

                <div class="text-lg text-center my-16 hidden "> TLDR: This is a funnel for donations from community-driven dapps. </div>


                <p class="text-gray-500 bg-neutral-800 p-4 mt-8"> <img src="@/assets/images/information.png" width="20" class="inline"/> Smart contracts that pay fees into the Guild contract [during execution flow] are more likely to be promoted, supported, and used by the community. </p>

         </div>

        
         


     </div>
   </div>


 

</PrimaryLayout>
</template>


<script>


 
import PrimaryLayout from './PrimaryLayout.vue'
 
import FrontPageMedia from './components/FrontPageMedia.vue';
  
import TabsBar from './components/TabsBar.vue';
  
import StarflaskAPIHelper from '../js/starflask-api-helper.js';
import FrontendHelper from '../js/frontend-helper.js';


export default {
  name: 'Home',
  props: [],
  components: {  TabsBar, FrontPageMedia, PrimaryLayout },
  data() {
    return {
     
      activePanelId: null, 
       

      
    }
  },

  created(){

  
   
       

  },
  mounted: function () {
         
    
  }, 
  methods: {
          setActivePanel(panelId){
              if(panelId == this.activePanelId){
                this.activePanelId = null;
                return 
              }
               this.activePanelId = panelId ;
          },
          onTabSelect(tabname){
            console.log(tabname)

              this.selectedTab = tabname.toLowerCase()


          },

          getRouteTo(dest){
            return FrontendHelper.getRouteTo(dest)
          }

       
 

  }
}
</script>
