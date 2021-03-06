<template>
  <v-layout align-center justify-center>
    <v-progress-circular v-if="loading" :size="70" :width="5" indeterminate></v-progress-circular>

    <v-flex xs10 sm8 md6 ref="flex" v-else>
      <v-container>
        <v-row>
          <v-col cols="12">
            <div class="logo-container">
              <img src="~/assets/purple-logo.svg" alt="castor" />
            </div>
          </v-col>
        </v-row>

        <v-row>
          <v-col cols="12">
            <h1 class="page-title">Cadastro</h1>
          </v-col>
        </v-row>
        <v-row>
          <v-col cols="12">
            <v-form ref="form" v-model="status" lazy-validation>
              <v-text-field color="#60c" v-model="form.name" label="Nome *" name="name" required></v-text-field>
              <v-text-field
                color="#60c"
                v-model="form.email"
                :rules="emailRules"
                label="Email *"
                name="email"
                required
              ></v-text-field>
              <v-text-field
                color="#60c"
                v-model="form.password"
                label="Senha *"
                name="password"
                :rules="passwordRules"
                :type="showPass ? 'password' : 'text'"
                :append-icon="showPass ? 'mdi-eye-off' : 'mdi-eye'"
                @click:append="() => (showPass = !showPass)"
                required
              ></v-text-field>
              <v-text-field
                color="#60c"
                v-model="form.confirmPassword"
                label="Confirmar senha *"
                :rules="confirmPasswordRules"
                :type="showConfirmPass ? 'password' : 'text'"
                :append-icon="showConfirmPass ? 'mdi-eye-off' : 'mdi-eye'"
                @click:append="() => (showConfirmPass = !showConfirmPass)"
                required
              ></v-text-field>
              <v-text-field
                color="#60c"
                v-model="form.urlFacebook"
                label="URL do Facebook"
                name="urlFacebook"
                required
              ></v-text-field>
              <v-text-field
                color="#60c"
                type="text"
                v-model="form.urlInstagram"
                label="URL do Instagram"
                name="urlInstagram"
                required
              ></v-text-field>
              <v-btn color="#60c" dark block depressed large @click="submit">Cadastrar</v-btn>
            </v-form>
          </v-col>
          <v-col cols="12" class="text-center">
            <a class="login-link" @click="gotoLogin">Ops, já tenho conta</a>
          </v-col>
          <v-snackbar
            v-model="snackbar"
            :color="snackbarStatus"
            :timeout="5000"
            :top="true"
            :right="true"
          >
            {{ snackbarText }}
            <v-btn color="#FFF" text @click="snackbar = false">Fechar</v-btn>
          </v-snackbar>
        </v-row>
      </v-container>
    </v-flex>
  </v-layout>
</template>

<router>
{
  path : '/cadastro'
}

</router>

<script scoped>
import auth from '../../services/http/auth'
import utils from '~/utils/index'

export default {
  data() {
    return {
      title: 'Cadastro',
      status: true,
      loading: false,
      showPass: String,
      showConfirmPass: String,
      snackbar: false,
      snackbarText: '',
      snackbarStatus: '',
      form: {
        name: "",
        email: "",
        password: "",
        confirmPassword: "",
        urlFacebook: "",
        urlInstagram: "",
        role: "STUDENT"
      },

      nameRules: [v => !!v || 'Digite seu nome'],
      passwordRules: [
        v => !!v || 'Digite a senha',
        v => (v && v.length >= 6) || 'A senha deve ter no mínimo 6 caractéres',
      ],
      emailRules: [
        v => !!v || 'Digite o e-mail',
        v => /.+@.+\..+/.test(v) || 'E-mail inválido',
      ],
    }
  },

  head() {
    return {
      title: this.title,
      meta: [
        {
          hid: 'description',
          name: 'description',
          content:
            'Cadastra-se no aplicativo da New School - Levamos educação de qualidade na linguagem da quebrada para as periferias do Brasil, através da tecnologia e da curadoria de conteúdos baseados nas habilidades do futuro.',
        },
      ],
    }
  },

  methods: {
    submit() {
      if (this.$refs.form.validate()) {
        const postObject = Object.assign({}, this.form)
        delete postObject.confirmPassword
        this.animateForm(true)
        this.loadClientCredentials()
          .then(res => {
            const token = res.data.accessToken
            auth
              .signUp(postObject, token)
              .then(res => {
                this.loading = false
                this.confirmSnackbar('Cadastro efetuado! ;)', 'success')
                setTimeout(() => {
                  this.gotoLogin()
                }, 2500)
              })
              .catch(err => {
                setTimeout(() => {
                  this.loading = false
                }, 500)
                console.error(err)
              })
          })
          .catch(() => {
            $nuxt._router.push('/login')
          })
      } else {
        this.animateForm(false)
      }
    },

    animateForm(status) {
      if (status) {
        this.$refs.flex.classList.add('hide-form')
        document.querySelector('html').style.overflow = 'hidden'
        setTimeout(() => {
          this.loading = true
        }, 300)
      } else {
        this.$refs.flex.classList.add('error-form')
        setTimeout(() => {
          this.$refs.flex.classList.remove('error-form')
        }, 500)
      }
      document.querySelector('html').style.overflow = 'scroll'
    },

    showPassword() {
      this.eyeIcon === 'mdi-eye'
        ? (this.eyeIcon = 'mdi-eye-off')
        : (this.eyeIcon = 'mdi-eye')
    },

    showConfirmPassword() {
      this.eyeIcon2 === 'mdi-eye'
        ? (this.eyeIcon2 = 'mdi-eye-off')
        : (this.eyeIcon2 = 'mdi-eye')
    },

    gotoLogin() {
      $nuxt._router.push('/login')
    },

    confirmSnackbar(text, status) {
      this.snackbarText = text
      this.snackbarStatus = status
      this.snackbar = true
    },
    loadClientCredentials() {
      return utils.getExternalCredentials()
    },
  },

  computed: {
    confirmPasswordRules() {
      return [
        v => !!v || 'Confirme a senha',
        () =>
          this.form.password === this.form.confirmPassword ||
          'As senhas devem ser idênticas.',
      ]
    },
  },
}
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css?family=Montserrat:400,500,900&display=swap');

/* Global */
* {
  font-family: 'Montserrat', Helvetica, Arial, sans-serif !important;
}

.flex {
  animation: intro 300ms backwards;
  animation-delay: 350ms;
}

.layout {
  background: #fff !important;
}

/* Page */
.page-title {
  font-size: 20px;
  font-weight: 900;
  line-height: 24px;
  text-transform: uppercase;
  color: #6600cc;
  width: 90%;
  margin-left: 50%;
  transform: translateX(-50%);
}

/* Logo */
.logo-container {
  display: flex;
  -webkit-box-pack: start;
  justify-content: flex-start;
  -webkit-box-align: start;
  align-items: flex-start;
}

.logo-container img {
  width: 48px;
}

/* Form */
.v-form {
  width: 100%;
/* inputs */
}
::v-deep .theme--light.v-text-field {
  margin-top: 0;
}

::v-deep .v-input {
  width: 90%;
  margin-left: 50%;
  transform: translateX(-50%);
}

::v-deep .theme--light.v-input:not(.v-input--is-disabled) input {
  font-size: 12px;
  color: #60c;
}

::v-deep
  .theme--light.v-text-field:not(.v-input--has-state)
  > .v-input__control
  > .v-input__slot:hover:before {
  border-color: #60c;
}

.theme--light.v-text-field:not(.v-input--has-state)
  > .v-input__control
  > .v-input__slot:hover:before {
  border-color: #6600cc !important;
  }
::v-deep .theme--light.v-label,
::v-deep .theme--light.v-icon {
  font-size: 12px;
  font-weight: 600;
  line-height: 15px;
  color: #aa56ff;
}

.theme--light.v-input:not(.v-input--is-disabled) input {
  color: #6600cc !important;
}
::v-deep .theme--light.v-icon {
  font-size: 20px;
}

::v-deep .v-btn {
  margin-top: 15px;
}

::v-deep .v-btn__content {
  color: #fff;
  font-size: 12px;
  font-weight: 900;
  line-height: 15px;
}
.v-text-field {
  padding-top: 0 !important;
  margin-top: 0 !important;
  color: #6600cc;
}

::v-deep
  .theme--light.v-text-field
  > .v-input__control
  > .v-input__slot::before {
  border-color: #aa56ff;}
.theme--dark.v-input:not(.v-input--is-disabled) input {
  color: #6600cc;
}

::v-deep
  .v-text-field.v-input--has-state
  > .v-input__control
  > .v-input__slot:before {
  border-color: #ff5252; /* cor da borda quando der estado de erro */
}

::v-deep .v-messages__message {
  color: #ff5252;
  font-size: 12px;
  margin-left: 5px;
}

.login-link {
  font-size: 12px;
  color: #6600cc;
}

.hide-form {
  animation: down 300ms forwards;
}

.error-form {
  animation: nono 300ms, intro paused;
}
</style>

/* Error messages */
.v-messages__message {
  color: #ff5252 !important;
  font-size: 12px !important;
  margin-left: 5px;
}

/* Snackbar */
.v-snack__content {
  border-radius: 5px;
}
</style>
