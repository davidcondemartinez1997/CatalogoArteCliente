<template>
  <div id="form">
    <form v-if="seen" class="form-horizontal" onsubmit="return false">
      <div>
        <h1 class="col-sm-9">Formulario Arte</h1>
      </div>
      <div>
        <button class="form-control btn-success btn-danger col-sm-2" v-on:click="close">Cerrar</button>
      </div>
      <div class="form-group" v-if="tipos && tipos.length">
        <label  class="control-label">Tipo de arte:</label>
        <select id="tipo" name="tipoArte" class="form-control" v-model="arte.TipoArte">
          <option v-for="tipoArte in tipos" v-bind:id="tipoArte.Nombre" v-bind:value="tipoArte.Nombre">{{tipoArte.Nombre}}</option>
        </select>
      </div>
      <div v-else>
        No existen tipos de arte creados, por favor crea uno
      </div>
      <div class="form-group">
        <label  class="control-label col-sm-2">Nombre:</label>
        <input type="text" id="nombre" name="nombre" class="form-control" v-model:value="arte.Nombre"/>   
      </div>
      <div class="form-group">
        <label class="control-label col-sm-2">Autor:</label>
        <input type="text" id="autor" name="autor" class="form-control" v-model:value="arte.Autor"/>   
      </div>
      <div class="form-group">
        <label class="control-label col-sm-2">Pais:</label>
        <input type="text" id="pais" name="pais" class="form-control" v-model:value="arte.Pais"/>   
      </div>
      <div class="form-group">
        <label class="control-label col-sm-2">Precio:</label>
        <input type="number" id="archivo" name="precio" class="form-control" v-model:value="arte.Precio"/>   
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
        permitePrecios: true,
      }
    },
    methods: {
      enviar: function(){
        let fecha = new Date();
        let data = {
          Nombre: this.arte.Nombre,
          Autor: this.arte.Autor,
          Pais: this.arte.Pais,
          Precio: this.arte.Pais,
          Fecha: fecha.toISOString(),
          Precio: this.arte.Precio,
          TipoArte: this.arte.TipoArte
        }

        let arteValidacion = isFormularioValido(data);

        if(arteValidacion ==''){
          if(this.arte.Id  == -1){
            axios.post(url ,data)
            .then(response => {
              this.arte.Id = response.data.Id;
              this.arte.TipoArte = response.data.TipoArte;
              this.arte.Nombre = response.data.Nombre;
              this.arte.Autor = response.data.Autor;
              this.arte.Pais = response.data.Pais;
              this.arte.Precio = response.data.Precio;
              this.arteBackUp.Nombre = response.data.Nombre;
              this.arteBackUp.Autor = response.data.Autor;
              this.arteBackUp.Pais = response.data.Pais;
              this.arteBackUp.Precio = response.data.Precio;
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
            if(this.arte.Nombre !== this.arteBackUp.Nombre || this.arte.Autor !== this.arteBackUp.Autor || this.arte.Pais !== this.arteBackUp.Pais || this.arte.Pais !== this.arteBackUp.Pais || this.arte.Precio !== this.arteBackUp.Precio || this.arte.TipoArte !== this.arteBackUp.TipoArte){
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
        cambioPais: function(e){
          this.arte.Pais = e.target.value;
        },
      },

      created() {
        this.seen = true;
        this.arte = this.$parent.arte;
        this.idSeleccionado = this.$parent.idSeleccionado;
        this.arteBackUp = {
          Nombre: this.arte.Nombre,
          Autor:  this.arte.Autor,
          TipoArte: this.arte.TipoArte,
          Pais: this.arte.Pais,
          Pais:  this.arte.Pais,
          Precio: this.arte.Precio
        };
        let url = config.address + 'TipoArte/';
        axios.get(url)
        .then(response => {
          this.tipos = response.data;
          if(!this.arte.TipoArte){
            this.arte.TipoArte = response.data[0].Nombre;  
          }

        })
      }
    }

    function isFormularioValido(data){
      let arteVal='';
      if(!data.TipoArte || data.TipoArte == ""){
        return 'Debes seleccionar un tipo de arte. Si no existe ninguno crea uno en la pestaña superior Tipo Arte'
      }

      if(!data.Nombre || data.Nombre.trim() == ''){
        return 'El nombre debe estar relleno.'
      }
      if(!data.Autor || data.Autor.trim() == ''){
        return 'El autor debe estar relleno'
      }

      if(!data.Pais || data.Pais.trim() == ''){
        return 'El pais debe estar seleccionado'
      }

      return '';
    } 
  </script>

  <style>
  </style>
