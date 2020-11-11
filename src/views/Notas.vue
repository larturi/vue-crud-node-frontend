<template>
    <div class="container">
        <h1 class="mt-4">Notas</h1>

        <b-alert
            :show="dismissCountDown"
            dismissible
            :variant="mensaje.color"
            @dismissed="dismissCountDown=0"
            @dismiss-count-down="countDownChanged"
            >
            {{mensaje.texto}}
        </b-alert>

        <form @submit.prevent="editarNota(notaEditar )" v-if="editar">
            <h3>Editar nota</h3>
            <input type="text" class="form-control my-2" placeholder="Nombre" v-model="notaEditar.nombre">
            <input type="text" class="form-control my-2" placeholder="Descripcion" v-model="notaEditar.descripcion">

            <div class="row">
                <div class="col">
                    <b-button class="btn btn-warning btn-block mt-3 mb-5" type="submit">Guardar</b-button>
                </div>
                <div class="col">
                    <b-button class="btn btn-secundary btn-block mt-3 mb-5" @click="editar=false">Cancelar</b-button>
                </div>
            </div>
        </form>

        <form @submit.prevent="agregarNota()" v-if="!editar">
            <h3>Agregar nueva nota</h3>
            <input type="text" class="form-control my-2" placeholder="Nombre" v-model="nota.nombre">
            <input type="text" class="form-control my-2" placeholder="Descripcion" v-model="nota.descripcion">
            <b-button class="btn btn-success btn-block mt-3 mb-5" type="submit">Agregar</b-button>
        </form>

        <table class="table">
            <thead>
                <tr>
                <th scope="col">#</th>
                <th scope="col">Nombre</th>
                <th scope="col">Descripcion</th>
                <th scope="col">Acciones</th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="(item, index) in notas" :key="index">
                    <th scope="row">{{ index + 1 }}</th>
                    <td>{{ item._id }}</td>
                    <td>{{ item.nombre }}</td>
                    <td>{{ item.descripcion }}</td>
                    <td>
                        <b-button class="btn btn-danger btn-sm mx-2" @click="eliminarNota(item._id)">Eliminar</b-button>
                        <b-button class="btn btn-alert btn-sm" @click="activarEdicion(item._id)">Editar</b-button>
                    </td>
                </tr>
            </tbody>
        </table>

        <nav aria-label="Page navigation example">
            <ul class="pagination">
                <li class="page-item" :class="{'disabled': paginaActual === 1}">
                <router-link class="page-link" :to="{ query: { pagina: paginaActual - 1 }}">Anterior</router-link>
                </li>
                <li class="page-item" :class="{'active':paginaActual === index + 1}"
                v-for="(item, index) in cantidadPaginas" :key="index">
                <router-link :to="{ query: { pagina: index + 1 }}" class="page-link">{{index + 1}}</router-link>
                </li>
                <li class="page-item" :class="{'disabled': paginaActual === cantidadPaginas}">
                <router-link class="page-link" :to="{ query: { pagina: paginaActual + 1 }}">Siguiente</router-link>
                </li>
            </ul>
        </nav>

<p>Total notas: {{totalNotas}} - Cantidad de páginas: {{cantidadPaginas}}</p>

    </div>
</template>

<script>
import {mapState} from 'vuex';

export default {
    data() {
        return {
            totalNotas: 0,
            limite: 5,
            paginaActual: 1,
            notas: [],
            dismissSecs: 5,
            dismissCountDown: 0,
            mensaje: {color: '', texto: ''},
            nota: {
                nombre: '',
                descripcion: ''
            },
            editar: false,
            notaEditar: {}
        }
    },
    watch: {
        "$route.query.pagina": {
            immediate: true,
            handler(pagina) {
                pagina = parseInt(pagina) || 1;
                this.paginacion(pagina);
                this.paginaActual = pagina;
            }
        }
    },
    computed: {
        ...mapState(['token']),
        cantidadPaginas() {
            return Math.ceil(this.totalNotas / this.limite);
        }
    },
    // created() {
    //     this.listarNotas()
    // },
    methods: {
        paginacion(pagina) {
            let config = {
                headers: {
                token: this.token
                }
            };
            let skip = (pagina - 1) * this.limite;
            this.axios
                .get(`/nota?skip=${skip}&limit=${this.limite}`, config)
                .then(res => {
                    console.log(res.data.notaDB);
                    this.notas = res.data.notaDB;
                    this.totalNotas = res.data.cantidad;
                })
                .catch(e => {
                  console.log(e.response);
                });
        },
        alerta() {
            this.mensaje.color = 'bg-success';
            this.mensaje.texto = 'Probando alerta';
            this.showAlert();
        },
        // listarNotas(){
        //     let config = {
        //         headers: {
        //            token: this.token
        //         }
        //     }

        //     this.axios.get('/nota', config)
        //         .then(res => {
        //             this.notas = res.data.notaDB;
        //         })
        //         .catch(e => {
        //             console.log(e.response);
        //         })
        // },
        agregarNota(){
            let config = {
                headers: {
                token: this.token
                }
            }
            // console.log(this.nota);
            this.axios.post('/nueva-nota', this.nota, config)
              .then(res => {
                this.notas.push(res.data)
                this.nota.nombre = '';
                this.nota.descripcion = '';
                this.mensaje.color = 'success';
                this.mensaje.texto = 'Nota Agregada!';
                this.showAlert()
              })
              .catch(e => {
                console.log(e.response);
                if(e.response.data.error.errors.nombre.message){
                    this.mensaje.texto = e.response.data.error.errors.nombre.message
                }else{
                    this.mensaje.texto = 'Error de sistema';
                }
                this.mensaje.color = 'danger';
                this.showAlert()
              })
        },
        eliminarNota(id) {
            this.axios.delete(`/nota/${id}`)
                .then(res => {
                    const index = this.notas.findIndex(item => item._id === res.data._id);
                    this.notas.splice(index, 1);
                    this.mensaje.color = 'success';
                    this.mensaje.texto = 'Nota eliminada';
                    this.showAlert();
                })
                .catch(error => {
                    console.error(error);
                });
        },
        countDownChanged(dismissCountDown) {
            this.dismissCountDown = dismissCountDown
        },
        showAlert() {
            this.dismissCountDown = this.dismissSecs
        },
        activarEdicion(id) {
            this.editar = true;
            this.axios.get(`/nota/${id}`)
               .then(res => {
                   this.notaEditar = res.data;
               })
               .catch(error => {
                   console.error(error);
               });
        },
        editarNota(notaEditar) {
            this.axios.put(`/nota/${notaEditar._id}`, notaEditar)
                .then(res => {
                    const index = this.notas.findIndex(n => n._id === res.data._id);
                    this.notas[index].nombre = res.data.nombre;
                    this.notas[index].descripcion  = res.data.descripcion;
                    this.mensaje.color = 'success';
                    this.mensaje.texto = 'Nota editada';
                    this.showAlert();
                    this.editar = false;
                })
                .catch(error => {
                    console.error(error);
                });
        },
        
    },
    
}
</script>