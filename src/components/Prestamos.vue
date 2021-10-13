<template>
  <div>
    <!-- Nuevo Prestamo -->
    <div class="row">
      <form @submit.prevent="handleSubmitForm">
        <h5>Nuevo Préstamo</h5>
        <div class="row pb-2">
          <div class="col-9">
            <input
              type="text"
              id="inNombre"
              v-model="nuevo_prestamo.libro"
              placeholder="Nombre del Libro"
              class="form-control"
              value=""
              readonly
              required
            />
          </div>
        </div>

        <div class="row">
          <div class="col-5">
            <select
              v-model="nuevo_prestamo.idUsuario"
              name=""
              id="selUsuario"
              class="form-select pe-1"
              required
            >
              <option value="" selected disabled>
                Seleccione un usuario...
              </option>
              <option
                v-for="usuario in Usuarios"
                :value="usuario._id"
                :key="usuario._id"
              >
                {{ usuario.nombre + " " + usuario.apellido }}
              </option>
            </select>
          </div>
          <div class="col-4">
            <input
              type="date"
              id="inFecha"
              class="form-control ps-1"
              v-model="nuevo_prestamo.fecha"
              data-bs-toggle="tooltip"
              title="Fecha de entrega"
              required
            />
          </div>
          <div class="col">
            <div class="col"></div>
            <button
              class="col btn btn-primary"
              :disabled="nuevo_prestamo.libro == '' || bloquear"
            >
              Prestar
            </button>
          </div>
        </div>
        <div class="col">
          <div class="row"></div>
        </div>
        <hr />
      </form>
    </div>

    <!-- Alertas -->
    <div class="alert alert-danger" role="alert" v-if="this.mensajeError != ''">
      {{ mensajeError }}
    </div>

    <!-- Lista de Libros -->
    <h5>Lista de Préstamos</h5>
    <div class="row pb-3 d-none">
      <div class="col-5 pe-1">
        <div class="input-group">
          <input
            type="text"
            id="inFitroTexto"
            v-model="filtro"
            placeholder="Todos"
            class="form-control"
          />
        </div>
      </div>
      <div class="col-auto">
        <button class="btn btn-primary" :disabled="bloquear">Buscar</button>
      </div>
    </div>
    <div class="row ps-3 pe-1">
      <table class="table table-sm table-bordered">
        <thead>
          <tr id="cent">
            <th scope="col">Libro</th>
            <th scope="col">Préstamo</th>
            <th scope="col">Usuario</th>
            <th scope="col">Acción</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="item in Prestamos" :key="item._id">
            <td>{{ item.libro }}</td>
            <td id="cent">{{ item.prestamo }}</td>
            <td>{{ item.usuario }}</td>
            <td id="cent">
              <button
                v-if="item.prestamo == 'Ninguno'"
                @click.prevent="seleccionarLibro(item._id, item.libro)"
                class="btn btn-outline-primary btn-sm p-1"
              >
                Seleccionar
              </button>
              <button
                v-else-if="item.prestamo == 'Vencido'"
                @click.prevent="entregarLibro(item._id)"
                class="btn btn-outline-danger btn-sm p-1"
              >
                Entregar
              </button>
              <button
                v-else
                @click.prevent="entregarLibro(item._id)"
                class="btn btn-outline-success btn-sm p-1"
                :disabled="item.prestamo == 'Error'"
              >
                Entregar
              </button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      nuevo_prestamo: {
        libro: "",
        idLibro: "",
        idUsuario: "",
        fecha: "",
      },
      filtro: "",
      mensajeError: "",
      bloquear: true,
      Prestamos: [],
      Usuarios: [],
    };
  },

  created() {
    console.log("Listando Prestamos");
    this.listarPrestamos();
  },

  mounted() {
    let hoy = new Date();
    document.getElementById("inFecha").min =
      hoy.getFullYear() +
      "-" +
      (hoy.getMonth() + 1) +
      "-" +
      (hoy.getDate() + 1);
    console.log(document.getElementById("inFecha").min);
  },

  methods: {
    handleSubmitForm() {
      // Prestar Libro
      this.bloquear = true;

      let apiURL = "http://localhost:3000/api/nuevo-prestamo";
      console.log(apiURL);

      axios
        .post(apiURL, {
          fecha_entrega: this.nuevo_prestamo.fecha,
          id_usuario: this.nuevo_prestamo.idUsuario,
          id_libro: this.nuevo_prestamo.idLibro,
        })
        .then((res) => {
          console.log(res.data);
          this.listarPrestamos();
        })
        .catch((error) => {
          console.log(error);
          this.mensajeError = "Error prestando Libro.";
          this.bloquear = false;
        });
    },

    listarPrestamos() {
      axios
        .get("http://localhost:3000/api/prestamo")
        .then((res) => {
          this.Prestamos = res.data;
          console.log(JSON.stringify(this.Prestamos));
          this.listarUsuarios();
          this.bloquear = false;
        })
        .catch((error) => {
          console.log(error);
          this.mensajeError = "No se puede traer la lista de Prestamos.";
        });

      this.nuevo_prestamo.libro = "";
      this.nuevo_prestamo.idUsuario = "";
      this.nuevo_prestamo.fecha = "";
      this.filtro = "";
    },

    listarUsuarios() {
      axios
        .get("http://localhost:3000/api/usuario/")
        .then((res) => {
          this.Usuarios = res.data;
          // console.log(JSON.stringify(this.Prestamos));
          this.mensajeError = "";
        })
        .catch((error) => {
          console.log(error);
          this.mensajeError = "No se puede traer la lista de Usuarios.";
        });
    },

    seleccionarLibro(id, nombre) {
      this.nuevo_prestamo.libro = nombre;
      this.nuevo_prestamo.idLibro = id;
    },

    entregarLibro(id) {
      this.bloquear = true;

      let apiURL = "http://localhost:3000/api/prestamo/" + id;
      console.log(apiURL);

      axios
        .delete(apiURL)
        .then((res) => {
          console.log(res.data);
          this.listarPrestamos();
        })
        .catch((error) => {
          console.log(error);
          this.mensajeError = "Error entregando Libro.";
          this.bloquear = false;
        });
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