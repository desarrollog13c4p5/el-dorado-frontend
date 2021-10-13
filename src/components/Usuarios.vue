<template>
  <div>
    <!-- Crear Usuarios -->
    <div class="row">
      <form @submit.prevent="handleSubmitForm">
        <h5>Crear Usuario</h5>
        <div class="row pb-1">
          <div class="col-8">
            <input
              type="text"
              id="inNombre"
              placeholder="Nombres del Usuario"
              v-model="nuevoUsuario.nombre"
              class="form-control"
              required
            />
          </div>
        </div>
        <div class="row">
          <div class="col-8">
            <input
              type="text"
              id="inApellido"
              placeholder="Apellidos del Usuario"
              v-model="nuevoUsuario.apellido"
              class="form-control"
              required
            />
          </div>
          <div class="col">
            <button class="btn btn-primary" :disabled="bloquear">Crear</button>
          </div>
        </div>
        <hr />
      </form>
    </div>

    <!-- Alertas -->
    <div class="alert alert-danger" role="alert" v-if="this.mensajeError != ''">
      {{mensajeError}}
    </div>

    <!-- Lista de Usuarios -->
    <div class="row">
      <h5>Lista de Usuarios</h5>
      <div class="row pb-2 d-none">
        <div class="col-6">
          <input
            type="text"
            id="inFiltro"
            placeholder="Todos"
            v-model="filtro"
            class="form-control"
          />
        </div>
        <div class="col-auto px-1">
          <button class="btn btn-primary" :disabled="bloquear">Aplicar</button>
        </div>
        <div class="col-auto px-1">
          <button class="btn btn-secondary" :disabled="bloquear">Limpiar</button>
        </div>
      </div>
      <div class="row ps-4 pe-1">
        <table class="table table-sm table-bordered">
          <thead>
            <tr id="cent">
              <th scope="col">Id</th>
              <th scope="col">Nombres</th>
              <th scope="col">Apelidos</th>
              <th scope="col">Acciones</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="item in Usuarios" :key="item._id">
              <td>{{ item._id.substring(20) }}</td>
              <td>{{ item.nombre }}</td>
              <td>{{ item.apellido }}</td>
              <td id="cent">
                <!-- <button
                  @click.prevent="editarUsuario(item._id, item.nombre)"
                  class="btn btn-outline-success btn-sm p-1"
                >
                  Editar
                </button> -->
                <button
                  @click.prevent="borrarUsuario(item._id, item.nombre)"
                  class="btn btn-outline-danger btn-sm p-1"
                >
                  Borrar
                </button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>

  </div>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      nuevoUsuario: {
        nombre: "",
        apellido: "",
      },
      filtro: "",
      mensajeError: "",
      bloquear: true,
      Usuarios: [],
      usuarioBorrar: {
        _id: "",
        nombre: ""
      },
    };
  },

  created() {
    console.log('Listando Usuarios');
    this.listarUsuarios();
  },

  methods: {
    handleSubmitForm() {  // Crear Usuario
      this.bloquear = true;

      let apiURL =
        "http://localhost:3000/api/nuevo-usuario"; //Enrutamiento con el back
      console.log(apiURL);

      axios.post(apiURL, {
          nombre: this.nuevoUsuario.nombre,
          apellido: this.nuevoUsuario.apellido
        })
        .then((res) => {
          console.log(res.data);
          this.listarUsuarios();
        })
        .catch((error) => {
          console.log(error);
          this.mensajeError =
            "Error creando nuevo Usuario, verifique el Nombre y Apellido e intentelo numevamente.";
          this.bloquear = false;
        });
    },

    listarUsuarios() {
      axios
        .get('http://localhost:3000/api/usuario/')
        .then((res) => {
          this.Usuarios = res.data.reverse();
          // console.log(JSON.stringify(this.Usuarios));
          this.mensajeError = ""
          this.bloquear = false;
        })
        .catch((error) => {
          console.log(error);
          this.mensajeError = "No se puede traer la lista de Usuarios."
        });
      
      this.nuevoUsuario.nombre = "";
      this.nuevoUsuario.apellido = "";
      this.filtro = "";
    },

    borrarUsuario(id, nombre) {
      console.log("Borrando " + nombre);
      axios
        .delete("http://localhost:3000/api/usuario/" + id)
        .then((res) => {
          console.log(res.data);
          this.listarUsuarios();
        })
        .catch((error) => {
          console.log(error);
          this.mensajeError = "No se borro Usuario " + nombre
        });
    },

    editarUsuario(id, nombres) {
      alert("No Implementado! (" + id + " - " + nombres + ")");
    },
  },
};
</script>

<style scoped>
a {
  text-decoration: none;
}

#cent {
  text-align: center;
}
</style>