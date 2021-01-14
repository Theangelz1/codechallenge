<template>
    <div>
        <div v-if="cargando_datos" class="text-center"> 
            <h3>Cargando Datos...</h3>
            <pulse-loader ></pulse-loader>
        </div>
        <v-data-table dark
            v-if="!cargando_datos"
            :headers="headers"
            :items="usuarios"
            sort-by="id"
            class="elevation-2"
            item-key="id"
        >
            <template v-slot:top>
            <v-toolbar flat>
                <div>
                <v-icon large left> mdi-format-list-bulleted </v-icon>
                USUARIOS
                <v-divider class="mx-3" inset vertical></v-divider>
                </div>
                <v-spacer></v-spacer>
                <v-dialog v-model="dialog" max-width="500px">
                <template v-slot:activator="{ on }">
                    <v-btn color="info" class="mb-2" v-on="on">AGREGAR NUEVO USUARIO</v-btn>
                </template>
                <v-card>
                    <v-card-title>
                    <span class="headline">{{ formTitle }}</span>
                    </v-card-title>

                    <v-card-text>
                    <v-container>
                        <v-form ref="form" v-model="valid" lazy-validation>
                        <v-text-field
                            prepend-icon="mdi-account"
                            v-model="editedItem.name"
                            type="text"
                            label="Nombre"
                            :rules="nombreRules"
                        ></v-text-field>

                        <v-text-field
                            prepend-icon="mdi-email"
                            v-model="editedItem.email"
                            type="text"
                            label="Email"
                            :rules="emailRules"
                        ></v-text-field>

                        <v-btn
                            :disabled="!valid"
                            color="success"
                            class="mr-4"
                            @click="validate"
                        >
                            GUARDAR
                        </v-btn>
                        </v-form>
                    </v-container>
                    </v-card-text> </v-card
                >|
                </v-dialog>
            </v-toolbar>
            </template>
            <template v-slot:[`item.actions`]="{ item }">
            <v-icon large color="blue" class="mr-2" @click="editItem(item)">
                mdi-pencil-circle
            </v-icon>
            <v-icon large color="red" @click="deleteItem(item)"> mdi-delete-circle </v-icon>
            </template>
        </v-data-table>
    </div>
</template>
<script>
import PulseLoader from 'vue-spinner/src/PulseLoader.vue'
export default {
  components: {
      PulseLoader
  },
  data: () => ({
    cargando_datos:false,
    dialog: false,
    usuarios: [],
    headers: [
      { text: "ID", align: "start", sortable: false, value: "id", align: " d-none" },
      { text: "NOMBRE", value: "name" },
      { text: "EMAIL", value: "email" },
      { text: "ACCION", value: "actions", sortable: false },
    ],
    editedIndex: -1,
    editedItem: {
      name: "",
      email: "",
    },
    defaultItem: {
      name: "",
      email: "",
    },
    valid: true,
    nombreRules: [(v) => !!v || "Ingrese un Nombre"],
    emailRules: [
      (v) => !!v || "E-mail obligatorio",
      (v) => /^\w+([.-]?\w+)*@\w+([.-]?\w+)*(\.\w{2,3})+$/.test(v) || "E-mail no es valido",
    ],
  }),
  methods: {
    async validate() {
      if (this.$refs.form.validate()) {
        if (this.editedIndex > -1) {
          try {
            const resp = await this.$axios.put(`/users/${this.editedItem.id}/`, {
              name: this.name,
              email: this.email,
            });
            this.$swal({
              icon: "success",
              title: "Editado Correctamente",
            });
            this.initialize();
            this.dialog = false;
            this.reset();
          } catch (error) {
            this.$swal({
              icon: "error",
              title: "No se pudo Editar",
            });
            this.dialog = false;
            this.reset();
          }
        } else {
          try {
            const resp = await this.$axios.$post(`users/`, {
              name: this.name,
              email: this.email,
            });
            this.$swal({
              icon: "success",
              title: "Guardado Correctamente",
            });
            this.initialize();
            this.dialog = false;
            this.reset();
          } catch (error) {
            this.$swal({
              icon: "error",
              title: "No se pudo Guardar",
            });
            this.dialog = false;
            this.reset();
          }
        }
      }
    },
    async initialize() {
        this.cargando_datos = true
        const resp = await this.$axios.$get("/users/")
        this.usuarios = resp
        this.cargando_datos = false
    },

    editItem(item) {
    //orden de item desde 0
    this.editedIndex = this.usuarios.indexOf(item);
    this.editedItem = Object.assign({}, item);
    this.dialog = true;
    },

    deleteItem(item) {
      this.editedIndex = this.usuarios.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.$swal({
        icon: "error",
        title: "Esta seguro que desea eliminar?",
        text: "Se eliminara todos los registros de esta empresa!",
        type: "warning",
        showCancelButton: true,
        confirmButtonColor: "#DD6B55",
        confirmButtonText: "Si estoy Seguro",
        cancelButtonText: "No, Cancelar esto",
        closeOnConfirm: false,
        closeOnCancel: false,
      }).then((result) => {
        if (result.value) {
          this.$axios
            .$delete("/api/users/" + this.editedItem.id + "/")
            .then((resp) => {
              this.$swal({
                icon: "success",
                title: "Eliminado Correctamente",
              }).then((result) => {
                if (result.value) {
                  this.initialize();
                  this.dialog = false;
                  this.editedItem = Object.assign({}, this.defaultItem);
                  this.editedIndex = -1;
                }
              });
              this.reset();
            })
            .catch((error) => {
              this.$swal({
                icon: "error",
                title: "No se pudo Eliminar",
              });
              this.reset();
            });
        } else {
          this.$swal({
            icon: "success",
            title: "Cancelado",
          });
          this.reset();
        }
      });
    },
    close() {
      this.dialog = false;
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem);
        this.editedIndex = -1;
      });
      this.reset();
    },
    reset() {
      this.$refs.form.reset();
    },
  },
  computed: {
    formTitle() {
      return this.editedIndex === -1 ? "Agregar Usuario" : "Editar Usuario";
    },
  },
  watch: {
    dialog(val) {
      val || this.close();
    },
  },
  created() {},
  beforeCreate() {},
  beforeMount() {
    this.initialize();
  },
  mounted() {},
  beforeUpdate() {
    console.log("5 - beforeUpdate");
  },
  updated() {
    console.log("6 - updated");
  },
  beforeDestroy() {
    console.log("7 - beforeDestroy");
  },
  destroyed() {
    console.log("8 - destroyed");
  },
};
</script>
