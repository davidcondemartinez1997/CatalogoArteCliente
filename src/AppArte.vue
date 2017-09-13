z<template class="container-fluid">
  <div id="maestrodetalle" class="row">
    <div id="lista" class="col-sm-6">
      <h1>Lista</h1>
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
          <tr v-for="(arte, index) of arte" v-on:click="detail" v-bind:id="index" v-bind:class="{ 'table-active': arte.Id == idSeleccionado}">
            <td> {{arte.Nombre}} </td>
            <td> {{arte.Autor}}</td>
            <td> {{arte.Pais}} </td>
            <td v-if="arte.Precio != 0"> {{arte.Precio}} </td>
            <td v-else>Sin precio</td>
            <td><button v-bind:id="arte.Id" value="Eliminar" class="form-control btn-success btn-danger" v-on:click="eliminar">Eliminar</button> </td>
          </tr>
        </tbody>
      </table>

      <button id="submit" class="btn btn-success btn-block" v-on:click="nuevo">Nuevo</button>
    </div>
    <div id="form" class="col-sm-6">
      <h3>Selecciona un elemento de la lista para continuar</h3>
    </div>
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
        idSeleccionado: undefined,
        tiposArte: undefined
      }
    },
    methods: {
      detail: function (e) {
        let id = e.target.id;
        if(id == ""){
          id = e.target.parentNode.id;
        }
        
        //Le doy a los id de los tr su posicion en el array en vez de el id del arte.
        //Al hacer esto me ahorro recorrer todo el array en busca de ese id
        let selected = this.arte[id];

        let arte = {
          Nombre: selected.Nombre,
          Autor: selected.Autor,
          Pais: selected.Pais,
          Precio: selected.Precio,
          TipoArte: selected.TipoArte,
          Id: selected.Id
        }
        this.openDetail(arte);
      },
      nuevo: function (e) {
        this.idSeleccionado = undefined;
        let arte =  {
          Nombre: undefined,
          Autor: undefined,
          Pais: undefined,
          Precio:undefined,
          TipoArte: this.tiposArte[0].Id,
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
            idSeleccionado: this.idSeleccionado,
            tiposArte: this.tiposArte
          },
        });
        EventBus.$on("seleccionarId",(id)=>{this.idSeleccionado = id});
      },
      init: function(){
        let url = config.address + 'Arte/';
        axios.get(url)
        .then(response => {
          this.arte = response.data;
          setTimeout("", 1000);
        })
        .catch(response => {
          swal(
            '',
            'Ha ocurrido un error',
            'error'
            )
        })
        url = config.address + 'TipoArte/';
        axios.get(url)
        .then(response => {
          this.tiposArte = response.data;
          
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
