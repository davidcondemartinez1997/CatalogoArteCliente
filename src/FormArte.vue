<template class="row">
  <div id="form" class="col-sm-6">
    <form v-if="seen" class="form-horizontal" onsubmit="return false">
      <div class="col-sm-7" style="display: inline-block">
        <h1>Formulario</h1>
      </div>
      <div class="col-sm-4" style="display: inline-block">
        <button class="form-control btn-success btn-info" v-on:click="close">Cerrar</button>
      </div>
      <div class="form-group" v-if="tipos && tipos.length">
        <label  class="control-label">Tipo de arte:</label>
        <select id="tipo" name="tipoArte" class="form-control" v-model="arte.TipoArte">
          <option v-for="tipoArte in tipos" v-bind:value="tipoArte.Id" >{{tipoArte.Nombre}}</option>
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
        <select id="pais" class="form-control" v-model="arte.Pais">
          <option v-for="country of countries" v-bind:value="country.code">{{country.name}}</option>
        </select>
      </div>
      <div class="form-group">
        <label class="control-label col-sm-2">Precio:</label>
        <div class="input-group">
          <input type="number" id="archivo" min="0" step="0.50" data-number-to-fixed="2" data-number-stepfactor="100" name="precio" class="form-control currency" v-model:value="arte.Precio"/>     
          <span class="input-group-addon">€</span>
        </div>
        
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
  import {countries} from './countries.js'
  
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
        countries: countries,
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
            if(this.arte.Nombre !== this.arteBackUp.Nombre || this.arte.Autor !== this.arteBackUp.Autor || this.arte.Pais !== this.arteBackUp.Pais || this.arte.Precio !== this.arteBackUp.Precio || this.arte.TipoArte !== this.arteBackUp.TipoArte){
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
        this.tipos = this.$parent.tiposArte;
        this.arteBackUp = {
          Nombre: this.arte.Nombre,
          Autor:  this.arte.Autor,
          TipoArte: this.arte.TipoArte,
          Pais: this.arte.Pais,
          Pais:  this.arte.Pais,
          Precio: this.arte.Precio
        };

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
