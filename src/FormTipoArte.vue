<template class="row">
  <div id="form" class="col-sm-6">
    <form v-if="seen" class="form-horizontal" onsubmit="return false" >

     <div class="col-sm-7" style="display: inline-block">
      <h1>Formulario</h1>
    </div>
    <div class="col-sm-4" style="display: inline-block">
      <button class="form-control btn-success btn-info" v-on:click="close">Cerrar</button>
    </div>
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
        <input v-if="tipoArte.FechaIni" type="date" id="fechaIni" name="fechaIni" class="form-control" v-bind:value="new Date(tipoArte.FechaIni).toISOString().split('T')[0]" v-on:change="cambiarFecha"/>
        <input v-else type="date" id="fechaIni" name="fechaIni" class="form-control" v-bind:value="tipoArte.FechaIni" v-on:change="cambiarFecha"/>
      </div>

      <div class="form-group">
        <label class="control-label">Composicion</label>
        <textarea id="composicion" name="composicion" class="form-control" rows="4" v-model:value="tipoArte.Composicion" />
      </div>

      <div>
        <input type="checkbox" id="colectivo" name="colectivo" v-model:value="tipoArte.Colectivo"/> 
        <label for="colectivo">¿Es un arte que se crea de manera colectiva?</label>
      </div>

      <div class="form-group">
        <button id="submit" value="Enviar" class="form-control btn-success btn-block" v-on:click="enviar">Enviar</button>
      </div>

    </form>
    <div v-else>
      <h3>Selecciona un elemento de la lista para continuar</h3>
    </div>
  </div>
</template>

<script>
  import axios from 'axios';
  import {EventBus} from './EventBus.js';
  import Vue from 'vue'
  import VeeValidate from 'vee-validate';
  Vue.use(VeeValidate);

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
      let FechaIni = new Date(this.tipoArte.FechaIni);
      FechaIni.setHours(12);
      let data = {
        Nombre: this.tipoArte.Nombre,
        Tangible: this.tipoArte.Tangible,
        FechaIni: FechaIni.toISOString(),
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
          if(this.tipoArteBackUp.Nombre !== this.tipoArte.Nombre || this.tipoArteBackUp.Tangible !== this.tipoArte.Tangible  || this.tipoArteBackUp.FechaIni !== this.tipoArte.FechaIni || this.tipoArteBackUp.Composicion !== this.tipoArte.Composicion || this.tipoArteBackUp.Colectivo !== this.tipoArte.Colectivo || this.tipoArteBackUp.FechaIni !== this.tipoArte.FechaIni){
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
    cambiarFecha: function(e){
      this.tipoArte.FechaIni = e.target.value;
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
      console.log(this.tipoArte.FechaIni);
    },
  }
</script>

<style>
</style>
