<template>
  <v-flex>
    <v-data-table
      :headers="headers"
      :items="desserts"
      :search="search"
      sort-by="calories"
      class="elevation-1"
    >
    <template v-slot:[`item.opciones`]="{ item }">
        <v-icon
        small
        class="mr-2"
        @click="editItem(item)"
        >
            edit
        </v-icon>
        <v-icon v-if="item.estado"
        small
        @click="activarDesactivarMostrar(2,item)"
        >
            block
        </v-icon>
        <v-icon v-else
        small
        @click="activarDesactivarMostrar(1,item)"
        >
            check
</v-icon>
<v-dialog v-model="adModal" max-width="290">
 <v-card>
 <v-card-title class="headline" v-if="adAccion==1">
 Activar Item
 </v-card-title>
 <v-card-title class="headline" v-if="adAccion==2">
 Desactivar Item
 </v-card-title>
 <v-card-text>
 Estás a punto de <span v-if="adAccion==1">activar </span>
 <span v-if="adAccion==2">desactivar </span> el item {{adNombre}}
 </v-card-text>
 <v-card-actions>
 <v-spacer></v-spacer>
 <v-btn @click="activarDesactivarCerrar()" color="green darken-1" flat="flat">
 Cancelar
 </v-btn>
 <v-btn v-if="adAccion==1" @click="activar()" color="orange darken-4"
flat="flat">
 Activar
 </v-btn>
 <v-btn v-if="adAccion==2" @click="desactivar()" color="orange darken-4"
flat="flat">
 Desactivar
 </v-btn>
 </v-card-actions>
 </v-card>
 </v-dialog>

</template>

      <template v-slot:top>
        <v-toolbar flat>
          <v-toolbar-title>Categorías</v-toolbar-title>
          <v-divider class="mx-4" inset vertical></v-divider>
          <v-spacer>
            <v-text-field
              class="text-xs-center"
              v-model="search"
              append-icon="search"
              label="Búsqueda"
              single-line
              hide-details
            ></v-text-field>
            <v-spacer></v-spacer>
          </v-spacer>
          <v-dialog v-model="dialog" max-width="500px">
            <template v-slot:activator="{ on }">
              <v-btn color="primary" dark class="mb-2" v-on="on"> Nuevo </v-btn>
            </template>
            <v-card>
              <v-card-title>
                <span class="headline">{{ formTitle }}</span>
              </v-card-title>

              <v-card-text>
                <v-container grid-list-md>
                  <v-layout wrap>
                    <v-flex xs12 sm12 md12>
                      <v-text-field
                        v-model="nombre"
                        label="Nombre"
                      ></v-text-field>
                    </v-flex>
                    <v-flex xs12 sm12 md12>
                      <v-text-field
                        v-model="descripcion"
                        label="Descripción"
                      ></v-text-field>
                    </v-flex>
                    <v-flex xs12 sm12 md12 v-show="valida">
                      <div
                        class="red--text"
                        v-for="v in validaMensaje"
                        :key="v"
                        v-text="v"
                      ></div>
                    </v-flex>
                  </v-layout>
                </v-container>
              </v-card-text>

              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="blue darken-1" text @click="close">
                  Cancelar
                </v-btn>
                <v-btn color="blue darken-1" text @click="guardar">
                  Guardar
                </v-btn>
              </v-card-actions>
            </v-card>
          </v-dialog>
          <v-dialog v-model="dialogDelete" max-width="500px">
            <v-card>
              <v-card-title class="headline"
                >Are you sure you want to delete this item?</v-card-title
              >
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="blue darken-1" text @click="closeDelete"
                  >Cancel</v-btn
                >
                <v-btn color="blue darken-1" text @click="deleteItemConfirm"
                  >OK</v-btn
                >
                <v-spacer></v-spacer>
              </v-card-actions>
            </v-card>
          </v-dialog>
        </v-toolbar>
      </template>
           
      <template v-slot:no-data>
        <v-btn color="primary" @click="initialize"> Reset </v-btn>
      </template>
    </v-data-table>
  </v-flex>
</template>
<script>
import axios from "axios";
export default {
  data: () => ({
    dialog: false,
    search: "",
    dialog: false,
    search: "",
    categorias: [],
    headers: [
      { text: "Opciones", value: "opciones", sortable: false },
      { text: "Nombre", value: "nombre", sortable: true },
      { text: "Descripción", value: "descripcion", sortable: false },
      { text: "Estado", value: "estado", sortable: false },
    ],
    editedIndex: -1,
    _id: "",
    nombre: "",
    descripcion: "",
    valida: 0,
    validaMensaje: [],
    adModal:0,
    adAccion:0,
    adNombre:'',
    adId:'',
  }),

  computed: {
    formTitle() {
      return this.editedIndex === -1 ? "New Item" : "Edit Item";
    },
  },

  watch: {
    dialog(val) {
      val || this.close();
    },
    dialogDelete(val) {
      val || this.closeDelete();
    },
  },

  created() {
    this.initialize();
  },

  methods: {
    listar() {
      let me = this;
      axios
        .get("categoria/list", configuracion)
        .then(function (response) {
          me.categorias = response.data;
        })
        .catch(function (error) {
          console.log(error);
        });
    },

    initialize() {
      this.desserts = [
        {
          nombre: "Frozen Yogurt",
          descripcion: 159,
          estado: 6.0,
        },
      ];
    },

    editItem(item) {
        this._id=item.id;
        this.nombre=item.nombre;
        this.descripcion=item.descripcion;
        this.dialog = true;
        this.editedIndex=1;

    },

    

    deleteItemConfirm() {
      this.desserts.splice(this.editedIndex, 1);
      this.closeDelete();
    },

    close() {
      this.dialog = false;
    },

    

    guardar() {
      let me = this;
      let header = { Token: this.$store.state.token };
      let configuracion = { headers: header };
      if (this.validar()) {
        return;
      }
      if (this.editedIndex > -1) {
        //Código para editar
        axios
          .put(
            "categoria/update",
            {
              _id: this._id,
              nombre: this.nombre,
              descripcion: this.descripcion,
            },
            configuracion
          )
          .then(function (response) {
            me.limpiar();
            me.close();
            me.listar();
          })
          .catch(function (error) {
            console.log(error);
          });
      } else {
        //Código para guardar
        axios
          .post(
            "categoria/add",
            { nombre: this.nombre, descripcion: this.descripcion },
            configuracion
          )
          .then(function (response) {
            me.limpiar();
            me.close();
            me.listar();
          })
          .catch(function (error) {
            console.log(error);
          });
      }
    },

    validar() {
      this.valida = 0;
      this.validaMensaje = [];
      if (this.nombre.length < 1 || this.nombre.length > 50) {
        this.validaMensaje.push(
          "El nombre de la categoría debe tener entre 1-50 caracteres."
        );
      }
      if (this.descripcion.length > 255) {
        this.validaMensaje.push(
          "La descripción de la categoría no debe tener más de 255 caracteres."
        );
      }
      if (this.validaMensaje.length) {
        this.valida = 1;
      }
      return this.valida;
    },

    limpiar() {
      this._id = "";
      this.nombre = "";
      this.descripcion = "";
      this.valida = 0;
      this.validaMensaje = [];
      this.editedIndex = -1;
    },

    activarDesactivarCerrar(){
 this.adModal=0;
 },
 activar(){
 let me=this;
 let header={"Token" : this.$store.state.token};
 let configuracion= {headers : header};
 axios.put('categoria/activate',{'_id':this.adId},configuracion)
 .then(function(response){
 me.adModal=0;
 me.adAccion=0;
 me.adNombre='';
 me.adId='';
 me.listar();
 })
 .catch(function(error){
 console.log(error);
 });
 },
 desactivar(){
 let me=this;
 let header={"Token" : this.$store.state.token};
 let configuracion= {headers : header};
 axios.put('categoria/deactivate',{'_id':this.adId},configuracion)
 .then(function(response){
 me.adModal=0;
 me.adAccion=0;
 me.adNombre='';
 me.adId='';
 me.listar();
 })
 .catch(function(error){
 console.log(error);
 });
 },
activarDesactivarCerrar(){
 this.adModal=0;
 },


  },
};
</script>