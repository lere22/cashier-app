<template>
  <div>
    <v-row>
      <v-col cols="6" offset="3">
        <v-card class="mb-2">
          <v-toolbar color="primary" class="white--text">
            Create User
          </v-toolbar>
          <v-card-text>
            <v-breadcrumbs class="pa-0" :items="breadcrumbs" />
            <v-form ref="form">
              <v-text-field name="fullname" label="Full Name" type="text" v-model="form.fullname"
                :rules="rules.fullname"></v-text-field>
              <v-text-field name="email" label="Email" type="email" v-model="form.email" :rules="rules.email"
                @keydown="checkEmailExist"></v-text-field>
              <v-text-field name="password" label="Password" type="password" v-model="form.password"
                :rules="rules.password"></v-text-field>
              <v-text-field name="retype_password" label="Retype Password" type="password"
                v-model="form.retype_password" :rules="rules.retype_password"></v-text-field>
              <v-select :items="roles" v-model="form.role" label="Role"></v-select>
            </v-form>
          </v-card-text>
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn color="primary" @click="onSubmit" :loading="isDisable">
              Save
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-col>
    </v-row>
  </div>
</template>

<script>
export default ({
  middleware: ['authenticated'],
  head: {
    title: 'Create User'
  },
  data() {
    return {
      emailExist: false,
      isDisable: false,
      roles: ['admin', 'cashier', 'employee'],
      breadcrumbs: [
        {
          text: 'Users',
          disabled: false,
          to: '/users',
          exact: true
        },
        {
          text: 'Create',
          disabled: true,
        }
      ],
      form: {
        fullname: '',
        email: '',
        password: '',
        retype_password: '',
        role: '',
      },
      rules: {
        fullname: [
          v => !!v || this.$t('FIELD_REQUIRED', { field: 'Nama Lengkap' }),
        ],
        email: [
          v => !!v || this.$t('FIELD_REQUIRED', { field: 'Email' }),
          v => /.+@.+/.test(v) || this.$t('EMAIL_INVALID'),
          v => !this.emailExist || this.$t('EMAIL_EXIST'),
        ],
        password: [
          v => !!v || this.$t('FIELD_REQUIRED', { field: 'Kata Sandi' }),
          v => v.length >= 6 || this.$t('FIELD_MIN_PASS', { field: 'Kata Sandi', min: 6 }),
        ],
        retype_password: [
          v => !!v || this.$t('FIELD_REQUIRED', { field: 'Ulangi Kata Sandi' }),
          v => v === this.form.password || this.$t('FIELD_CONFIRM_PASS', { field: 'Kata Sandi', confirm: 'Ulangi Kata Sandi' }),
        ],
        role: [
          v => !!v || this.$t('FIELD_REQUIRED', { field: 'Role' }),
        ],
      }
    }
  },
  methods: {
    checkEmailExist() {
      this.emailExist = false;
    },
    onSubmit() {
      if (this.$refs.form.validate()) {
        this.isDisable = true;
        console.log(this.form);
        this.$axios.$post('/users', this.form)
          .then(response => {
            this.isDisable = false;
            // jika berhasil redirect ke halaman users
            this.$router.push({
              name: 'users___' + this.$i18n.locale,
              params: { message: 'CREATE_SUCCESS', fullname: this.form.fullname }
            })
          }).catch(error => {
            // email exist
            if (error.response.data.message == "EMAIL_EXIST") {
              this.emailExist = true
              this.$refs.form.validate()
            }
            this.isDisable = false;
          });
      }
    }
  }
})
</script>
