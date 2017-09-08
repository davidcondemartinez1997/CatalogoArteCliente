<template>
  <div id="form">
    <form v-if="seen" class="form-horizontal" onsubmit="return false" >

      <h1>Formulario Tipo de Arte <a class="close" v-on:click="close">&times;</a></h1>
      <div class="form-group">
        <label class="control-label">Nombre:</label>
        <input type="text" id="nombre" name="nombre" class="form-control" v-model:value="tipoArte.Nombre"/>   
      </div>
      <div class="form-group">
        <input type="checkbox" id="tangible" name="tangible" v-model:value="tipoArte.Tangible"/> 
        <label class="">¿Las obras son tangibles?</label>
      </div>
      <div class="form-group">
        <label class="control-label" for="fechaIni">Fecha de inicio:</label>
        <input type="date" id="fechaIni" name="fechaIni" class="form-control" v-model="tipoArte.FechaIni"/>
      </div>

      <div class="form-group">
        <label class="control-label">Composicion</label>
        <textarea id="composicion" name="composicion" class="form-control" v-model:value="tipoArte.Composicion" />
      </div>

      <div class="form-group">
      <input type="checkbox" id="colectivo" name="colectivo" class="form-control" v-model:value="tipoArte.Colectivo"/> 
        <label class="control-label" for="colectivo">¿Es un arte que se crea de manera colectiva?</label>
      </div>

      <div class="form-group">
        <button id="submit" value="Enviar" class="form-control btn-success btn-block" v-on:click="enviar">Enviar</button>
      </div>

    </form>
  </div>
</template>

<script>
  import axios from 'axios';
  import {EventBus} from './EventBus.js';
  import Vue from 'vue'

  let url = config.address + 'TipoArte/';
  export default {
    name: 'app',
    data () {
      return {
       tipoArte: {},
       seen: true,
       idSeleccionado: undefined
     }
   },
   methods: {
    enviar: function(){
      let preventUpdate = false;

      let data = {
        Nombre: this.tipoArte.Nombre,
        Tangible: this.tipoArte.Tangible,
        FechaIni: this.tipoArte.FechaIni,
        Composicion: this.tipoArte.Composicion,
        Colectivo: this.tipoArte.Colectivo,
      }

      let arteValidacion = this.isFormularioValido(data);
      if(arteValidacion == ''){
        if(this.tipoArte.Id == -1){

          axios.post(url, data)
          .then(response => {
            this.tipoArte.Id = response.data.Id;
            this.tipoArte.Nombre = response.data.Nombre;
            this.tipoArte.Tangible = response.data.Tangible;
            this.tipoArte.FechaIni = response.data.FechaIni;
            this.tipoArte.Composicion = response.data.Composicion;
            this.tipoArte.Colectivo = response.data.Colectivo;
            this.fireEvent();
          })
          .catch(response => {
            swal(
              '',
              'Ha ocurrido un error',
              'error'
              )
          })
        }else{
          if(this.tipoArteBackUp.Nombre !== this.tipoArte.Nombre || this.tipoArteBackUp.Tangible !== this.tipoArte.Tangible  || this.tipoArteBackUp.FechaIni !== this.tipoArte.FechaIni || this.tipoArteBackUp.Composicion !== this.tipoArte.Composicion || this.tipoArteBackUp.Colectivo !== this.tipoArte.Colectivo){
            data.Id = this.tipoArte.Id;
            axios.put(url + data.Id, data)
            .then(response => {
              this.fireEvent();
              EventBus.$emit("seleccionarId", response.data.Id);
            })
            .catch(response => {
              swal(
                '',
                'Ha ocurrido un error',
                'error'
                )
            })
          }else{
            swal(
              '',
              'Los datos no han cambiado',
              'info'
              )
          }

        }
      }else{
        swal(
          '',
          arteValidacion,
          'error'
          )
      }
    },
    isFormularioValido: function(data){
      if(!data.Nombre || data.Nombre.trim() == ''){
        return 'El nombre deberia estar relleno'
      }
      if(!data.FechaIni){
        return 'Selecciona una fecha de inicio'
      }

      if(!data.Composicion || data.Composicion.trim() == ''){
        return 'La composicion deberia estar rellena'
      }

      return '';
    },

    fireEvent: function(){
      swal(
        '',
        'Operacion completada con exito!',
        'success'
        ).then( () =>  {
          EventBus.$emit("updateTipoArte", this.tipoArte);
        }


        );

      },

      close: function(){
        this.seen = false;
        EventBus.$emit("seleccionarId", undefined);
      }
    },
    created() {
      this.seen = true;
      this.tipoArte = this.$parent.tipoArte;
      this.tipoArteBackUp = {
        Nombre: this.tipoArte.Nombre,
        Tangible: this.tipoArte.Tangible,
        FechaIni: this.tipoArte.FechaIni,
        Composicion: this.tipoArte.Composicion,
        Colectivo: this.tipoArte.Colectivo,
      };
      this.idSeleccionado = this.$parent.idSeleccionado;
    },
  }
</script>

<style>
</style>
