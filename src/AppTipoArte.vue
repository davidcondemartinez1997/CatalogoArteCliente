<template>
  <div id="maestrodetalle">
    <div id="lista">
      <h1>Tipos de arte</h1>
      <table v-if="tipos && tipos.length" class="table table-bordered">
        <thead class="thead-inverse">
          <tr>
            <th>Nombre</th>
            <th>Tangible</th>
            <th>Fecha de inicio</th>
            <th>Colectivo</th>
            <th>Eliminar</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="tipo of tipos" v-on:click="detail" v-bind:id="tipo.Id" v-bind:class="{ 'table-active': tipo.Id == idSeleccionado}">
            <td> {{tipo.Nombre}}</td>
            <td v-if= "tipo.Tangible"> Si </td>
            <td v-else> No </td>
            <td> {{new Date(tipo.FechaIni).toLocaleDateString()}}</td>
            <td v-if="tipo.Colectivo"> Si </td>
            <td v-else> No </td>
            <td><button v-bind:id="tipo.Id" value="Eliminar" class="form-control btn-success btn-danger" v-on:click="eliminar">Eliminar</button> </td>
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
        this.idSeleccionado = id;
        this.tipos.forEach((p, index) => {
          if(p.Id == id){
            let tipoArte = {
              Nombre: p.Nombre,
              Tangible: p.Tangible,
              FechaIni: p.FechaIni,
              Composicion: p.Composicion,
              Colectivo: p.Colectivo,
              Id: p.Id
            }
            this.openDetail(tipoArte);
          }
        });
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
        let id = e.target.Id;
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
            this.fireEvent();
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
