<template>
  <div id="maestrodetalle">
    <div id="lista">
      <h1>Arte</h1>
      <table v-if="arte && arte.length" class="table table-bordered table-hover">
        <thead class="thead-inverse">
          <tr>
            <th>Nombre</th>
            <th>Autor</th>
            <th>Pais</th>
            <th>Precio</th>
            <th>Eliminar</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="arte of arte" v-on:click="detail" v-bind:id="arte.Id" v-bind:class="{ 'table-active': arte.Id == idSeleccionado}">
            <td> {{arte.Nombre}} </td>
            <td> {{arte.Autor}}</td>
            <td> {{arte.Pais}} </td>
            <td> {{arte.Precio}} </td>
            <td><button v-bind:id="arte.Id" value="Eliminar" class="form-control btn-success btn-danger" v-on:click="eliminar">Eliminar</button> </td>
          </tr>
        </tbody>
      </table>

      <button id="submit" class="btn btn-success btn-block" v-on:click="nuevo">Nuevo</button>
    </div>
    <div id="form" ></div>
  </div>

</template>

<script>
  import axios from 'axios';
  import {EventBus} from './EventBus.js';
  import Form from './FormArte.vue'
  import Vue from 'vue'

  export default {
    name: 'app',
    data () {
      return {
        arte: undefined,
        idSeleccionado: undefined
      }
    },
    methods: {
      detail: function (e) {
        let id = e.target.id;
        if(id == ""){
          id = e.target.parentNode.id;
        }
        this.idSeleccionado = id;

        this.arte.forEach((p, index) => {
          if(p.Id == id){
            let arte = {
              Nombre: p.Nombre,
              Autor: p.Autor,
              Pais: p.Pais,
              Precio: p.Precio,
              Id: p.Id
            }
            this.openDetail(arte);
          }
        });
      },
      nuevo: function (e) {
        this.idSeleccionado = undefined;
        let arte =  {
          Nombre: undefined,
          Autor: undefined,
          Pais: undefined,
          Precio:undefined,
          Id:-1
        }
        this.openDetail(arte);
      },
      eliminar: function(e){
        let id = e.target.id;
        swal({
          title: '¿Quieres eliminar la obra arte?',
          type: 'warning',
          showCancelButton: true,
          confirmButtonColor: '#3085d6',
          cancelButtonColor: '#d33',
          confirmButtonText: 'Si, eliminar!',
          cancelButtonText: 'Cancelar'
        }).then( () => {
          this.seen = false;
          let url = config.address + 'Arte/';
          axios.delete(url + id)
          .then(response => {
            this.init();
            swal(
              '',
              'La obra de arte ha sido borrada.',
              'success'
              )
          })
          .catch(response => {
            swal(
              '',
              'Ha ocurrido un error',
              'error'
              )
          })
        })


      },
      openDetail:function(arte){
        new Vue({
          el: '#form',
          render: h => h(Form),
          data: {
            arte:arte,
            idSeleccionado: this.idSeleccionado
          },
        });
        EventBus.$on("seleccionarId",(id)=>{this.idSeleccionado = id});
      },
      init: function(){
        let url = config.address + 'Arte/';
        axios.get(url)
        .then(response => {
          this.arte = response.data;
        })
        .catch(response => {
          swal(
            '',
            'Ha ocurrido un error',
            'error'
            )
        })
      }

    },
    created() {
      this.init();
      EventBus.$on('updateArte', (() => {
        this.init();
      }));
    }
  }
</script>

<style>
</style>
