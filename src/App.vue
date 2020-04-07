<template>
  <div id="app">
    <button @click="getOptions">저장</button>
    <!-- <defaultGrid /> -->
    <ul id="manu-tab" class="nav nav-tabs nav-justified" role="tablist" style="width:600px;">
      <li class="nav-item" v-for="tab in tabs" :key="tab[0]" @click="tabChange(tab[1])">
        <a class="nav-link" :class="{ active: currentTab.id === tab[0]}"
            data-toggle="pill" href="#manu-tabContent1"
            role="tab" aria-controls="manu-tabContent" aria-selected="true">
            {{ tab[1].name }}
        </a>

      </li>
    </ul>

    <keep-alive>
      <component v-bind:is="currentTab.tabName" :tabId="currentTab.id" :key="currentTab.id" :id="currentTab.id">
      </component>
    </keep-alive>
  </div>
</template>

<script>
import defaultGrid from './components/defaultGrid.vue'
import { EventBus } from './components/event-bus.js'

export default {
  name: 'app',
  components: {
    defaultGrid
  },
  data () {
    return {
      tabs: new Map(),
      /* tabs의 형태는 아래와 같다.
      ["000", {id: "000", name: '그리드1', tabName: 'defaultGrid'}]
      ["001", {id: "001", name: '그리드2',  tabName: 'defaultGrid'}]
      */
      currentTab: null
    }
  },

  created () {
    this.tabs.set("000", {id: "000", name: '그리드1', tabName: 'defaultGrid'})
    this.tabs.set("001", {id: "001", name: '그리드2', tabName: 'defaultGrid'})
    this.tabs.set("002", {id: "002", name: '그리드3', tabName: 'defaultGrid'})
    this.tabs.set("003", {id: "003", name: '그리드4', tabName: 'defaultGrid'})
    
    this.currentTab = this.tabs.get("000")

    EventBus.$on("returnOptions", payload => {
      console.log("received option from ", payload.tabId, payload)
    })
  },
  methods: {
    tabChange(tab){
      this.tabInfoPush()

      this.currentTab = tab;
    },

    tabInfoPush(){
      var i = 0;
      var xChecked = []
      var yChecked = []

      while(document.getElementById("X" + i)){
        if(document.getElementById("X" + i).checked){xChecked.push(1)}
        else{xChecked.push(0)}

        if(document.getElementById("Y" + i).checked){yChecked.push(1)}
        else{yChecked.push(0)}
        /* xChecked.push(document.getElementById("X" + i).checked)
        yChecked.push(document.getElementById("Y" + i).checked) */
        i++;
      }
      
      window.localStorage.setItem("X" + this.currentTab.id,xChecked)
      window.localStorage.setItem("Y" + this.currentTab.id,yChecked)
    },

    getOptions () {
      this.tabInfoPush()
      EventBus.$emit("getOptions")
    }
  }
}
</script>

<style lang="scss">
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
  @import "../node_modules/ag-grid-community/dist/styles/ag-grid.css";
  @import "../node_modules/ag-grid-community/dist/styles/ag-theme-balham.css";
}

.ag-header-cell-label{justify-content: center;}
</style>
