<template class="container-fluid">
  <div id="maestrodetalle" class="row">
    <div id="lista" class="col-sm-6">
      <h1>Lista</h1>
      <table v-if="tipos && tipos.length" class="table table-bordered">
        <thead class="thead-inverse">
          <tr>
            <th>Nombre</th>
            <th>Tangible</th>
            <th>Colectivo</th>
            <th>Eliminar</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(tipo, index) of tipos" v-on:click="detail" v-bind:id="index" v-bind:class="{ 'table-active': tipo.Id == idSeleccionado}">
            <td> {{tipo.Nombre}}</td>
            <td v-if= "tipo.Tangible"> Si </td>
            <td v-else> No </td>
            <td v-if="tipo.Colectivo"> Si </td>
            <td v-else> No </td>
            <td><button v-bind:id="tipo.Id" value="Eliminar" class="form-control btn-success btn-danger" v-on:click="eliminar">Eliminar</button> </td>
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
  import Form from './FormTipoArte.vue'
  import {EventBus} from './EventBus.js';
  import Vue from 'vue'

  export default {
    name: 'app',
    data () {
      return {
        tipos: undefined,
        idSeleccionado: undefined
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
        let selected = this.tipos[id];

        let tipoArte = {
          Nombre: selected.Nombre,
          Tangible: selected.Tangible,
          FechaIni: selected.FechaIni,
          Composicion: selected.Composicion,
          Colectivo: selected.Colectivo,
          Id: selected.Id
        }
        this.openDetail(tipoArte);
      },
      nuevo: function (e) {
        let tipoArte = {
          Nombre: undefined,
          Tangible: true,
          FechaIni: undefined,
          Composicion: undefined,
          Colectivo: true,
          Id: -1
        }
        this.openDetail(tipoArte);
      },
      openDetail: function(tipoArte){
        new Vue({
          el: '#form',
          render: h => h(Form),
          data: {
            tipoArte:tipoArte,
            idSeleccionado: this.idSeleccionado
          },
        });
        EventBus.$on("seleccionarId",(id)=>{this.idSeleccionado = id});
      },
      eliminar: function(e){
        let id = e.target.id;
        let url = config.address + 'TipoArte/';
        swal({
          title: '¿Quieres eliminar el tipo de arte?',
          type: 'warning',
          showCancelButton: true,
          confirmButtonColor: '#3085d6',
          cancelButtonColor: '#d33',
          confirmButtonText: 'Si, eliminar!',
          cancelButtonText: 'Cancelar'
        }).then( () => {
          this.seen = false;
          axios.delete(url + id)
          .then(response => {
            this.init();
            swal(
              '',
              'El tipo de arte ha sido borrado.',
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
      init: function(){
        let url = config.address + 'TipoArte/';
        axios.get(url)
        .then(response => {
          this.tipos = response.data;
        })
      }

    },
    created() {
      this.init();
      EventBus.$on('updateTipoArte', () => {
        this.init();
      });

    }

  }
</script>

<style>
</style>
