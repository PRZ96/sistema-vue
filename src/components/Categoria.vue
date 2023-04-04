<template>
  <v-layout align-start class="mt-6">
    <v-flex>

      <v-data-table :headers="headers" :items="categorias" :search="search" class="elevation-1">
        <template v-slot:top>
          <v-toolbar flat color="white">
            <v-toolbar-title>Categorías</v-toolbar-title>
            <v-divider class="mx-4" inset vertical></v-divider>
            <v-spacer></v-spacer>
            <v-text-field class="text-xs-center" v-model="search" append-icon="search" label="Búsqueda" single-line
              hide-details></v-text-field>
            <v-spacer></v-spacer>
            <v-divider class="mx-4" inset vertical></v-divider>
            <v-dialog v-model="dialog" max-width="500px">
              <template v-slot:activator="{ on, attrs }">
                <v-btn color="primary" dark class="mb-2" v-bind="attrs" v-on="on">Nuevo</v-btn>
              </template>
              <v-card>
                <v-card-title>
                  <span class="headline">{{ formTitle }}</span>
                </v-card-title>
                <v-card-text>
                  <v-container>
                    <v-row>
                      <v-col cols="12" sm="12" md="12">
                        <v-text-field v-model="editedItem.nombre" label="Nombre"></v-text-field>
                      </v-col>
                      <v-col cols="12" sm="12" md="12">
                        <v-text-field v-model="editedItem.descripcion" label="Descripción"></v-text-field>
                      </v-col>
                      <v-col cols="12" sm="12" md="12" v-show="valida"> <!-- Muestra este mensaje solo cuando valida es true -->
                        <div class="red--text" v-for="v in validaMensaje" :key="v" v-text="v"></div>
                      </v-col>
                    </v-row>
                  </v-container>
                </v-card-text>
                <v-card-actions>
                  <v-spacer></v-spacer>
                  <v-btn color="blue darken-1" text @click="close">Cancelar</v-btn>
                  <v-btn color="blue darken-1" text @click="guardar">Guardar</v-btn>
                </v-card-actions>
              </v-card>
            </v-dialog>

          </v-toolbar>
        </template>
        <template v-slot:item.opciones="{ item }">
          <v-icon small class="mr-2" @click="editItem(item)">
            mdi-pencil
          </v-icon>
          <v-icon small @click="deleteItem(item)">
            mdi-delete
          </v-icon>
        </template>
        <template v-slot:item.estado="{ item }">
          <div v-if="item.estado">
            <span class="blue--text">Activo</span>
          </div>
          <div v-else="item.estado">
            <span class="red--text">Inactivo</span>
          </div>
          <!--         {{ item.estado }} -->
        </template>
        <template v-slot:no-data>
          <v-btn color="primary" @click="listar()">Resetear</v-btn>
        </template>
      </v-data-table>
    </v-flex>
  </v-layout>
</template>

<script>
import axios from "axios";
export default {
  data() {
    return {
      dialog: false,
      search: '',
      categorias: [],
      headers: [
        { text: 'Opciones', value: 'opciones', sortable: false },
        { text: 'Nombre', value: 'nombre', sortable: true },
        { text: 'Descripción', value: 'descripcion', sortable: false },
        { text: 'Estado', value: 'estado', sortable: false },
      ],
      editedIndex: -1,
      _id: '',
      nombre: '',
      descripcion: '',
     editedItem: {
      nombre: '',
      descripcion: '',
     },
     defaultItem: {
      nombre: '',
      descripcion: '',
     },
     valida: 0,
     validaMensaje: [],
    };
  },
  computed: {
    formTitle() {
      return this.editedIndex === -1 ? 'Nuevo registro' : 'Editar registro'
    },
  },

  watch: {
    dialog(val) {
      val || this.close()
    },
  },

  created() {
    this.listar()
  },

  methods: {
    listar() {
      let me = this;
      axios.get('categoria/list').then(function (response) {
        me.categorias = response.data;
      }).catch(function (error) {
        console.log(error);
      })
    },

    editItem(item) {
      /* this.editedIndex = this.categorias.indexOf(item) */
      this.editedItem = Object.assign({}, item)
      this.dialog = true;
      this.editedIndex = 1; //Indicar que lo que quiero es editar
    },

    deleteItem(item) {
      const index = this.categorias.indexOf(item)
      confirm('Estás seguro de borrar este registro?') && this.categorias.splice(index, 1)
    },

    close() {
      this.dialog = false;

      /* this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem)
        this.editedIndex = -1
      }) */

      this.editedItem = Object.assign({}, this.defaultItem);
      this.editedIndex = -1;
      this.valida = 0;
      this.validaMensaje = [];
    },

    save() {
      if (this.editedIndex > -1) {
        Object.assign(this.categorias[this.editedIndex], this.editedItem)
      } else {
        this.categorias.push(this.editedItem)
      }
      this.close()
    },
    limpiar() {
      this.editedItem._id = '';
    },
    guardar() {
      let me = this;
      if (this.validar()) {
        return;
      }
      if(this.editedIndex > -1) {
        //Editar los datos del registro
        axios.put('categoria/update', { '_id': this.editedItem._id, 'nombre': this.editedItem.nombre, 'descripcion': this.editedItem.descripcion })
        .then(function(response) {
          me.limpiar();
          me.close();
          me.listar();
        }) 
        .catch(function(error){
          console.log(error);
        });
      } else {
        //Guardar un nuevo registro
        axios.post('categoria/add', { 'nombre': this.editedItem.nombre, 'descripcion': this.editedItem.descripcion })
        .then(function(response) {
          me.limpiar();
          me.close();
          me.listar();
        }) 
        .catch(function(error){
          console.log(error);
        });
      }
      this.close();
    },
    validar() {
      // Valida si el nombre o la descripcion de la categoria ingresada son dentro de los limites correctos
      this.valida = 0;
      this.validaMensaje = [];
      if(this.editedItem.nombre.length < 1 || this.editedItem.nombre.length > 50 ) {
        this.validaMensaje.push('El nombre de la categoria debe tener entre 1 y 50 caracteres');
      }
      if(this.editedItem.descripcion.length > 255 ) {
        this.validaMensaje.push('La descripción de la categoria no debe tener más de 255 caracteres');
      }
      if(this.validaMensaje.length) {
        this.valida = 1;
      }
      return this.valida;
    },
  },
};
</script>
