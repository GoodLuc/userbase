<template>
  <q-page>
    <h1 class="text-center">Userbase</h1>

    <!-- Lista de usuarios, si existe -->
    <div v-if="!users.length" class="text-center">
      <p>No hay usuarios cargados.</p>
      <q-btn color="primary" @click="showUserForm" icon="person_add" label="Cargar usuario" />
    </div>
    <div v-else>
      <table class="userTable">
        <thead>
          <tr>
            <td>ID</td><td>Nombre</td><td>Edad</td><td>Sexo</td><td>Email</td><td></td>
          </tr>
        </thead>
        <tbody>
          <tr v-for="user in userPage" :key="user.id">
            <td v-if="user.id">{{ user.id }}</td>
            <td v-if="user.nombre">{{ user.nombre }}</td>
            <td v-if="user.nacimiento">{{ user.nacimiento }}</td>
            <td v-if="user.sexo"><span class="sx">{{ user.sexo }}</span></td>
            <td v-if="user.email">{{ user.email }}</td>
            <td align="right">
              <div class="q-gutter-xs">
                <q-btn flat round color="primary" icon="edit" @click="editUser(user.id)" />
                <q-btn flat round color="red" icon="delete" @click="deleteDialogPrompt(user.id)" />
              </div>
            </td>
          </tr>
        </tbody>
      </table>

      <!-- Paginación -->
      <div v-if="users.length > 5" align="center" class="q-py-md">
        <q-btn v-for="n in pages" flat :label="n" :key="n" @click="setPage(n)" :class="{ currentPage: (page == n)}" />
      </div>

      <div align="right" class="q-py-lg">
        <q-btn color="primary" @click="showUserForm" icon="person_add" label="Cargar usuario" />
      </div>
    </div>

    <!-- Formulario de carga y edición de usuarios -->
    <q-dialog v-model="userFormDialog" ref="userFormDialog">
      <q-card class="userFormDialog">
        <q-card-section>
          <h2><q-icon :name="userFormIcon" /> {{ userFormTitle }}</h2>
          <form>
            <div class="q-mb-sm">
              <q-input outlined v-model="userInput['nombre']"
               label="Nombre" :rules="[ val => val && val.length > 0 || 'Por favor complete este campo']" />
            </div>
              
            <div class="q-mb-sm">
              <q-input outlined v-model="userInput['nacimiento']"
               :rules="[ val => val && val.length > 0 || 'Por favor complete este campo']" 
               label="Fecha de nacimiento" @focus="$refs.qDateProxy.show()">
                <template v-slot:append>
                  <q-icon name="event" class="cursor-pointer">
                    <q-popup-proxy ref="qDateProxy" transition-show="scale" transition-hide="scale">
                      <q-date mask="DD-MM-YYYY" v-model="userInput['nacimiento']" 
                      :default-year-month="defaultCalendarYearAndMonth" @input="() => $refs.qDateProxy.hide()" />
                    </q-popup-proxy>
                  </q-icon>
                </template>
              </q-input>
            </div>

            <div class="q-mb-lg">
              <div class="q-gutter-sm flex">
                <label class="self-center"><strong>Sexo</strong></label>
                <q-radio v-model="userInput['sexo']" val="Femenino" label="Femenino" color="pink" />
                <q-radio v-model="userInput['sexo']" val="Masculino" label="Masculino" color="blue" />
              </div>
            </div>

            <div class="q-mb-sm">
              <q-input outlined v-model="userInput['email']" type="email"
               label="Email" :rules="[ val => val && val.length > 0 || 'Por favor complete este campo', isValidEmail]" />
            </div>

          </form>
        </q-card-section>

        <q-card-actions align="right" class="q-mb-lg q-pa-md">
          <q-btn label="Cancelar" v-close-popup />
          <q-btn label="Guardar" color="primary" icon="save" @click="saveUser" />
        </q-card-actions>
      </q-card>
    </q-dialog>

    <!-- Mensaje de alerta o éxito -->
    <q-dialog v-model="message.on" persistent transition-show="scale" transition-hide="scale">
      <q-card class="bg-primary text-white" style="width: 300px">
        <q-card-section>
          <div class="text-h6">{{ message.title }}</div>
        </q-card-section>

        <q-card-section class="q-pt-none">
          {{ message.text }}
        </q-card-section>

        <q-card-actions align="right" class="bg-white text-teal">
          <q-btn flat label="OK" v-close-popup />
        </q-card-actions>
      </q-card>
    </q-dialog>

    <!-- Diálogo de confirmación para eliminar -->
    <q-dialog v-model="deleteDialog" persistent>
      <q-card>
        <q-card-section class="row items-center">
          <q-avatar icon="delete" color="red" text-color="white" />
          <span class="q-ml-sm">¿Seguro que desea eliminar el usuario <strong>{{ usuarioAEliminar.nombre }}</strong>?</span>
        </q-card-section>

        <q-card-actions align="right">
          <q-btn label="Cancelar" v-close-popup />
          <q-btn label="Eliminar" icon="delete" color="red" v-close-popup @click="deleteConfirm(usuarioAEliminar.id)" />
        </q-card-actions>
      </q-card>
    </q-dialog>
  </q-page>
</template>

<script>
export default {
  name: 'PageIndex',
  data() {
    return {
      users: [],
      page: 1,
      userFormDialog: false,
      userFormIcon: 'person_add',
      userFormTitle: 'Agregar usuario',
      userInput: { nombre: '', nacimiento: '', sexo: '', email: ''},
      defaultCalendarYearAndMonth: ( new Date().getFullYear() - 21 ).toString() + '/01',
      message: { on: false, title: '', text: ''},
      deleteDialog: false,
      usuarioAEliminar: ''
    }
  },
  methods: {
    // Mostrar formulario
    showUserForm() {
      this.userFormTitle = 'Agregar usuario'
      this.userFormIcon = 'person_add'
      this.userFormDialog = true
      this.userInput = { nombre: '', nacimiento: '', sexo: '', email: ''}
    },
    // Guardar usuario
    saveUser() {
      // Chequear campos requeridos
      var req = true
      for(var campo in this.userInput) {
        if (this.userInput[campo] == '') {
          req = false
        }
      }
      if (req) {
        // Si no tiene campo ID, agregar nuevo usuario
        if (!this.userInput.hasOwnProperty('id')) {
          this.users.push({...this.userInput, id: this.users.length+1}) 
        } else {
          // Si tiene campo ID, actualizar usuario existente
          this.users[this.users.findIndex((user) => user.id == this.userInput.id)] = this.userInput
        }
        window.localStorage.setItem('usuarios', JSON.stringify(this.users))
        this.users = JSON.parse(window.localStorage.getItem('usuarios'))
        // Diálogo de éxito
        this.message = { 
          on: true, 
          title: 'Usuario guardado', 
          text: 'Se ha guardado exitosamente el usuario '+this.userInput.nombre
        }
        this.$refs.userFormDialog.hide()
        this.userInput = { nombre: '', nacimiento: '', sexo: '', email: ''}
      } else {
        // Diálogo de error
        this.message = { 
          on: true, 
          title: 'Faltan completar campos', 
          text: 'Por favor complete todos los campos para poder guardar el usuario.'
        }
      }
    },
    // Editar usuario por campo ID
    editUser(id) {
      this.userFormTitle = 'Editar usuario'
      this.userFormIcon = 'edit'
      Object.assign(this.userInput, this.users.find(user => user.id == id))
      this.userFormDialog = true
    },
    // Eliminar usuario
    deleteDialogPrompt(id) {
      this.usuarioAEliminar = this.users.find(user => user.id == id)
      this.deleteDialog = true
    },
    deleteConfirm(id) {
      this.users.splice(this.users.findIndex((user) => user.id == id), 1)
      this.deleteDialog = false
      this.message = { 
        on: true, 
        title: 'Usuario eliminado', 
        text: 'Se ha eliminado el usuario '+this.usuarioAEliminar.nombre
      }
      window.localStorage.setItem('usuarios', JSON.stringify(this.users))
    },
    // Paginación
    setPage(n) {
      this.page = n
    },
    // Chequear que el email sea válido
    isValidEmail (val) {
      const emailPattern = /^(?=[a-zA-Z0-9@._%+-]{6,254}$)[a-zA-Z0-9._%+-]{1,64}@(?:[a-zA-Z0-9-]{1,63}\.){1,8}[a-zA-Z]{2,63}$/;
      return emailPattern.test(val) || 'Esto no parece un email';
    }
  },
  computed: {
    // Paginación
    userPage: function() {
      if (this.page == 1) { var marker = 0
      } else { var marker = (this.page * 5) - 5 }
      return this.users.slice(marker, marker+5)
    },
    pages: function() {
      return Math.ceil(this.users.length / 5)
    }
  },
  mounted() {
    // Al montar la aplicación, cargar usuarios desde LocalStorage
    if (!window.localStorage.getItem('usuarios')) {
      console.log('No user base detected. Creating...')
      window.localStorage.setItem('usuarios', JSON.stringify([]))
      this.users = JSON.parse(window.localStorage.getItem('usuarios'))
    } else {
      this.users = JSON.parse(window.localStorage.getItem('usuarios'))
    }
  }
}
</script>

<style lang="stylus">
  .userFormDialog { width: calc(100% - 20px); max-width: 800px !important }
  .userFormDialog h2 { font-size: 28px; margin: 10px 0 }
  .userTable { width: 100% }
  .userTable thead td { font-weight: bold; border-bottom: 2px solid #c4c4c4 }
  .userTable td { padding: 10px }
  .userTable tbody tr:nth-child(odd) { background: #E0F2F1 }
  .currentPage { background: #407BE3; color: white }

  @media (max-width: 600px) {
    .userTable td { padding: 5px }
    .userTable .sx {
      width: 1.7em;
      white-space: nowrap;
      overflow: hidden;
      text-overflow: ellipsis;
      display: block;
    }
  }
</style>