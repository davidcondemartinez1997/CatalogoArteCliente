<template>
  <div id="home" class="container-fluid">
    <header class="jumbotron text-center">
      <h1>Catalogo de arte</h1>
      <button type="button" id="arte" class="btn btn-md btn-outline-primary" v-on:click="openComponent">Arte</button>
      <button type="button" id="tipoArte" class="btn btn-md btn-outline-primary" v-on:click="openComponent">Tipo Arte</button>
    </header>
    
    <div id="maestrodetalle"></div>
    <div id="messages"></div>
  </div>
</template>

<script>
  import axios from "axios"
  import Vue from 'vue'
  import MaestroTipoArte from "./AppTipoArte.vue"
  import MaestroArte from "./AppArte.vue"

  import {EventBus} from './EventBus.js';
  export default {
    name: 'app',
    data () {
      return {

      }
    },
    methods: {
      openComponent: function(e){
        if($('#' + e.target.id).hasClass("active")){
          document.getElementById("maestrodetalle").innerHTML = "";
          $('button').removeClass("active");
        }else{
          $('button').removeClass("active");
          switch(e.target.id){
            case "arte": 
            $('#arte').addClass('active');
            new Vue({
              el: '#maestrodetalle',
              render: h => h(MaestroArte)
            })

            break;
            case "tipoArte": 
            $('#tipoArte').addClass('active');
            new Vue({
              el: '#maestrodetalle',
              render: h => h(MaestroTipoArte)
            })

            break;
          }
        }

        
      }

    },

  }
</script>

<style>
</style>