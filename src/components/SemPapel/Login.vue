<template>
  <v-container fill-height style="height: 90vh">
    <v-row class="text-center" align="center" justify="center">
      <v-col cols="12" sm="6">
        <Logo :letras="true" :quadrados="true" :animado="true" />
        <br />
        <br />
        <v-form ref="loginForm">
          <v-text-field
            v-model="username"
            :rules="[rules.required, rules.characters]"
            label="Usuário"
            placeholder="Digite seu CPF ou matrícula"
            filled
            autofocus
            prepend-inner-icon="fa-user"
            color="primary"
            v-upper-case
            @input="username = username.replace(/[^\w]/, '')"
            @keyup.enter="entrar"
          ></v-text-field>

          <v-text-field
            v-model="password"
            :append-icon="show ? 'fa-eye' : 'fa-eye-slash'"
            :rules="[rules.required]"
            :type="show ? 'text' : 'password'"
            label="Senha"
            placeholder="Digite sua senha"
            filled
            prepend-inner-icon="fa-lock"
            color="primary"
            @click:append="show = !show"
            @keyup.enter="entrar"
          ></v-text-field>
        </v-form>

        <v-btn large color="accent" dark @click="entrar">
          <v-icon left>fa-sign-in-alt</v-icon>
          Entrar</v-btn
        >
      </v-col>
    </v-row>
  </v-container>
</template>

<script lang="ts">
import Vue from "vue";
import Logo from "./Logo.vue";

import { ipcRenderer } from "electron";
declare var api: {
  entrar: (creds: { username: string; password: string }) => void;
  semPapelEntrarSucesso: (
    callback: (creds: { username: string; password: string }) => void
  ) => void;
};

export default Vue.extend({
  name: "LoginSemPapel",

  components: {
    Logo,
  },

  data() {
    return {
      password: "",
      username: "",
      show: false,
      rules: {
        required: (value: string) => !!value || "Obrigatório.",
        characters: (value: string) =>
          !/[^\w]/.test(value) || "Somente letras e números.",
      },
    };
  },

  methods: {
    entrar(): void {
      if (
        !(
          this.$refs as unknown as { loginForm: { validate: () => boolean } }
        ).loginForm.validate()
      )
        return;

      api.entrar({ username: this.username, password: this.password });
    },

    entrarSucesso(creds: { username: string; password: string }): void {
      this.$root.$data.credentials.sempapel = creds;
      this.$router.push("/sempapel/inserviveis");
    },
  },

  mounted(): void {
    api.semPapelEntrarSucesso((creds): void => {
      this.entrarSucesso(creds);
    });
  },
});
</script>
