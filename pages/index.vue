<template>
  <div>
    <v-card width="400" class="mx-auto mt-12">
      <v-card-title class="justify-center">
        <h2>INICIO DE SECCION</h2>
      </v-card-title>
      <v-card-text>
        <v-form v-model="valid">
          <v-text-field
            v-model="login.username"
            label="Usuario"
            :counter="11"
            type="text"
            :rules="UserRules"
            prepend-icon="mdi-account"
            onkeyup="javascript:this.value=this.value.toUpperCase();"
          />
          <v-text-field
            v-model="login.password"
            :type="showPassword ? 'text' : 'password'"
            label="Contraseña"
            :rules="PasswordRules"
            prepend-icon="mdi-lock"
            :append-icon="showPassword ? 'mdi-eye' : 'mdi-eye-off'"
            @click:append="showPassword = !showPassword"
          />
        </v-form>
      </v-card-text>
      <v-divider></v-divider>
      <v-card-actions>
        <v-col class="text-center" cols="6" sm="12">
          <v-btn color="info" large :disabled="!valid" @click="userLogin">Ingresar</v-btn>
          <!--<v-btn color="info">Login</v-btn>-->
        </v-col>
      </v-card-actions>
    </v-card>
  </div>
</template>

<script>
export default {
  layout: "login",
  data() {
    return {
      obras_asignadas: [],
      dialog: false,
      showPassword: false,
      login: {
        username: "00000000000",
        password: "Gianpier1",
      },
      valid: true,
      UserRules: [
        (v) => !!v || "DNI requerido",
        (v) => (v && v.length == 11) || "Ingrese 11 caracteres",
      ],
      PasswordRules: [(v) => !!v || "Contraseña obliglatoria"],
    };
  },
  methods: {
    async userLogin() {
      this.$router.push("/users");
    },
  },
};
</script>
