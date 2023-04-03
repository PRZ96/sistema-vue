<template>
  <v-layout align-start class="mt-6">
    <v-flex>
      <v-toolbar flat color="white">
        <v-toolbar-title>Categorías</v-toolbar-title>
        <v-divider class="mx-2" inset vertical></v-divider>

        <v-spacer></v-spacer>

        <v-text-field class="text-xs-center" v-model="search" append-icon="search" label="Búsqueda" single-line
          hide-details></v-text-field>

        <v-spacer></v-spacer>

        <v-dialog v-model="dialog" max-width="500px">
          <template v-slot:activator="{ on }">
            <v-btn color="primary" dark class="mb-2" v-on="on">Nuevo</v-btn>
          </template>

          <v-card>
            <v-card-title>
              <span class="headline">{{ formTitle }}</span>
            </v-card-title>
            <v-card-text>
              <v-container grid-list-md>
                <v-layout wrap>
                  <v-flex xs12 sm6 md4>
                    <v-text-field v-model="editedItem.name" label="Dessert name"></v-text-field>
                  </v-flex>
                  <v-flex xs12 sm6 md4>
                    <v-text-field v-model="editedItem.calories" label="Calories"></v-text-field>
                  </v-flex>
                  <v-flex xs12 sm6 md4>
                    <v-text-field v-model="editedItem.fat" label="Fat (g)"></v-text-field>
                  </v-flex>
                  <v-flex xs12 sm6 md4>
                    <v-text-field v-model="editedItem.carbs" label="Carbs (g)"></v-text-field>
                  </v-flex>
                  <v-flex xs12 sm6 md4>
                    <v-text-field v-model="editedItem.protein" label="Protein (g)"></v-text-field>
                  </v-flex>
                </v-layout>
              </v-container>
            </v-card-text>
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="blue darken-1" class="white--text" flat @click="close">Cancelar</v-btn>
              <v-btn color="blue darken-1" class="white--text" flat @click="save">Guardar</v-btn>
            </v-card-actions>
          </v-card>

        </v-dialog>
      </v-toolbar>

      <v-data-table :headers="headers" :items="categorias" :search="search" class="elevation-1">
        <template v-slot:items="props">
          <td>{{ props.item.nombre }}</td>
          <td>{{ props.item.descripcion }}</td>
          <td>
            <div v-if="prop.item.estado">
              <span class="blue--text">Activo</span>
            </div>
            <div v-else="prop.item.estado">
              <span class="red--text">Inactivo</span>
            </div>
          </td>
        </template>
        <template v-slot:item.opciones="{ item }"> <!-- usa los iconos en el header opciones -->
          <td>
            <v-icon small class="mr-2" @click="editItem(item)"> mdi-pencil </v-icon>
            <v-icon small @click="deleteItem(item)"> mdi-delete </v-icon>
          </td>
        </template>
        <template v-slot:no-data>
          <v-btn color="primary" @click="listar()">Resetear</v-btn>
          <!-- Si no hay valores se muestra un boton de -->
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
      search: "",
      categorias: [],
      headers: [
        { text: 'Opciones', value: 'opciones', sortable: false },
        { text: "Nombre", value: "nombre", sortable: true },
        { text: "Descripción", value: "descripcion", sortable: false },
        { text: "Estado", value: "estado", sortable: false },
      ],
      editedIndex: -1,
      editedItem: {
        name: "",
        calories: 0,
        fat: 0,
        carbs: 0,
        protein: 0,
      },
      defaultItem: {
        name: "",
        calories: 0,
        fat: 0,
        carbs: 0,
        protein: 0,
      },
    };
  },
  computed: {
    formTitle() {
      return this.editedIndex === -1 ? "New Item" : "Edit Item";
    },
  },
  watch: {
    dialog(val) {
      val || this.close();
    },
  },
  created() {
    this.listar();
  },
  methods: {
    listar() {
      let me = this;
      axios
        .get("categoria/list")
        .then(function (response) {
          //console.log(response);
          me.categorias = response.data; //Se asigna las categorias que se obtienen del response en data en el array categorias
        })
        .catch(function (error) {
          console.log(error);
        });
    },

    editItem(item) {
      this.editedIndex = this.desserts.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialog = true;
    },

    deleteItem(item) {
      const index = this.desserts.indexOf(item);
      confirm("Are you sure you want to delete this item?") &&
        this.desserts.splice(index, 1);
    },

    close() {
      this.dialog = false;
      setTimeout(() => {
        this.editedItem = Object.assign({}, this.defaultItem);
        this.editedIndex = -1;
      }, 300);
    },

    save() {
      if (this.editedIndex > -1) {
        Object.assign(this.desserts[this.editedIndex], this.editedItem);
      } else {
        this.desserts.push(this.editedItem);
      }
      this.close();
    },
  },
};
</script>
