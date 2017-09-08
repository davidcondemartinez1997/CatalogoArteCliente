<template>
  <div id="form">
    <form v-if="seen" class="form-horizontal" onsubmit="return false">
      <h1 class="col-sm-12">Formulario Arte <a class="close" v-on:click="close">&times;</a></h1>
      <div class="form-group" v-if="tipos && tipos.length">
        <label  class="control-label">Tipo de arte:</label>
        <select id="tipo" name="tipoArte" class="form-control" v-on:change="seleccionarTipo" v-model="arte.TipoArte">
        <option v-for="tipoArte in tipos" v-bind:id="tipoArte.Nombre" v-bind:value="tipoArte.Nombre">{{tipoArte.Nombre}}</option>
        </select>
      </div>
      <div v-else>
        No existen tipos de arte creados, por favor crea uno
      </div>
      <div class="form-group">
        <label  class="control-label col-sm-2">Destinatario:</label>
        <input type="text" id="destinatario" name="destinatario" class="form-control" v-model:value="arte.Destinatario"/>   
      </div>
      <div class="form-group">
        <label class="control-label col-sm-2">Asunto:</label>
        <input type="text" id="asunto" name="asunto" class="form-control" v-model:value="arte.Asunto"/>   
      </div>
      <div class="form-group">
        <label class="control-label col-sm-2">Contenido:</label>
        <input type="text" id="contenido" name="contenido" class="form-control" v-model:value="arte.Contenido"/>   
      </div>
      <div class="form-group" v-if="permiteFicheros == true">
        <label class="control-label col-sm-2">Archivo:</label>

        <input type="file" id="archivo" name="archivo" class="form-control" v-on:change="cambioArchivo"/>   
      </div>
      <div  v-if="permiteDestacados == true" class="form-group">
        <input type="checkbox" id="dest" value="Activado" v-model:value="arte.Destacado"/>
        <label for="dest">Permitir destacar arte</label>
      </div>
      <div class="form-group">
        <button id="submit" value="Enviar" class="form-control btn-success btn-block" v-on:click="enviar">Enviar</button>
        <button id="remove" value="Eliminar" class="form-control btn-success btn-danger" v-on:click="eliminar" v-if="arte.Id !== -1">Eliminar</button>
      </div>
    </form>
  </div>
</template>

<script>
  import axios from 'axios';
  import {EventBus} from './EventBus.js';
  let url = config.address + 'Arte/';
  export default {
    name: 'app',
    data () {
      return {
        arte: {},
        seen: true,
        idSeleccionado: undefined,
        tipos: undefined,
        permiteFicheros: true,
        permiteDestacados: true,
      }
    },
    methods: {
      enviar: function(){
        let fecha = new Date();
        let data = {
          Destinatario: this.arte.Destinatario,
          Asunto: this.arte.Asunto,
          Contenido: this.arte.Contenido,
          Archivo: this.arte.Archivo,
          Fecha: fecha.toISOString(),
          Destacado: this.arte.Destacado,
          TipoArte: this.arte.TipoArte
        }

        let arteValidacion = isFormularioValido(data);

        if(arteValidacion ==''){
          if(this.arte.Id  == -1){
            axios.post(url ,data)
            .then(response => {
              this.arte.Id = response.data.Id;
              this.arte.Destinatario = response.data.Destinatario;
              this.arte.Asunto = response.data.Asunto;
              this.arte.Contenido = response.data.Contenido;
              this.arte.Archivo = response.data.Archivo;
              this.arte.Destacado = response.data.Destacado;
              this.arteBackUp.Destinatario = response.data.Destinatario;
              this.arteBackUp.Asunto = response.data.Asunto;
              this.arteBackUp.Contenido = response.data.Contenido;
              this.arteBackUp.Archivo = response.data.Archivo;
              this.arteBackUp.Destacado = response.data.Destacado;
              this.fireEvent();
            })
            .catch(response => {
              swal(
                '',
                'Ha ocurrido un error al enviar',
                'error'
                )
            })
          }
          else{
            if(this.arte.Destinatario !== this.arteBackUp.Destinatario || this.arte.Asunto !== this.arteBackUp.Asunto || this.arte.Contenido !== this.arteBackUp.Contenido || this.arte.Archivo !== this.arteBackUp.Archivo || this.arte.Destacado !== this.arteBackUp.Destacado || this.arte.TipoArte !== this.arteBackUp.TipoArte){
              data.Id = this.arte.Id;
              axios.put(url + data.Id, data)
              .then(response => {
                this.fireEvent();
              })
              .catch(response => {
                swal(
                  '',
                  'Ha ocurrido un error al actualizar',
                  'error'
                  )
              })
            }
            else{
              swal(
                '',
                'Los datos no han cambiado',
                'info'
                )
            }     
          }
        }
        else{
          swal(
            '',
            arteValidacion,
            'error'
            )
        }
      },
      close: function(){
        this.seen = false;
        EventBus.$emit("seleccionarId", undefined);
      },
      seleccionarTipo: function(){
        var selectorTipo = document.getElementById("tipo");
        var nombre = selectorTipo.options[selectorTipo.selectedIndex].value;
        this.tipos.forEach((tipo, index)=>{
          if(tipo.Nombre == nombre){
            this.arte.TipoArte = this.tipos[index].Nombre;
            this.permiteDestacados = this.tipos[index].Destacado;
            this.permiteFicheros = this.tipos[index].Fichero;
          }
        });
      },
      fireEvent: function(){
        swal(
          '',
          'Operacion completada con exito!',
          'success'
          ).then( () =>  {
            EventBus.$emit("updateArte", this.tipoArte);
          }


          );
        },
        cambioArchivo: function(e){
          this.arte.Archivo = e.target.value;
        },
        eliminar: function(){
          if(this.arte.Id != -1){
            swal({
              title: '¿Quieres eliminar el arte?',
              type: 'warning',
              showCancelButton: true,
              confirmButtonColor: '#3085d6',
              cancelButtonColor: '#d33',
              confirmButtonText: 'Si, eliminar!',
              cancelButtonText: 'Cancelar'
            }).then( () => {
              this.seen = false;
              axios.delete(url + this.arte.Id)
              .then(response => {
                this.fireEvent();
                swal(
                  '',
                  'El arte ha sido borrado.',
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


          }
        }
      },

      created() {
        this.seen = true;
        this.arte = this.$parent.arte;
        if(!this.arte.TipoArte){
          this.arte.TipoArte = {};
        }
        this.idSeleccionado = this.$parent.idSeleccionado;
        this.arteBackUp = {
          Destinatario: this.arte.fDestinatario,
          Asunto:  this.arte.Asunto,
          Contenido: this.arte.Contenido,
          Archivo:  this.arte.Archivo,
          Destacado: this.arte.Destacado
        };
        let url = config.address + 'TipoArte/';
        axios.get(url)
        .then(response => {
          this.tipos = response.data;
          this.arte.TipoArte = response.data[0].Nombre;
        })
      }
    }

    function isFormularioValido(data){
      let arteVal='';
      /*if(data.TipoArte || data.TipoArte == ""){
        return 'Debes seleccionar un tipo de arte. Si no existe ninguno crea uno en la pestaña superior Tipo Arte'
      }*/

      if(!data.Destinatario || data.Destinatario.trim() == ''){
        return 'El destinatario debe estar relleno.'
      }
      if(!data.Asunto || data.Asunto.trim() == ''){
        return 'El asunto debe estar relleno'
      }

      if(!data.Contenido || data.Contenido.trim() == ''){
        return 'El contenido debe estar relleno'
      }

      return '';
    } 
  </script>

  <style>
  </style>
