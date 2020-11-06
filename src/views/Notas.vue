<template>
    <div class="container">
        <h1>Notas</h1>

        <b-alert
            :show="dismissCountDown"
            dismissible
            :variant="mensaje.color"
            @dismissed="dismissCountDown=0"
            @dismiss-count-down="countDownChanged"
            >
            {{mensaje.texto}}
        </b-alert>

        <form @submit.prevent="agregarNota()">
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
                        <b-button @click="eliminarNota(item._id)">Eliminar</b-button>
                    </td>
                </tr>
            </tbody>
        </table>

    </div>
</template>

<script>
export default {
    data() {
        return {
            notas: [],
            dismissSecs: 5,
            dismissCountDown: 0,
            mensaje: {color: '', texto: ''},
            nota: {
                nombre: '',
                descripcion: ''
            }
        }
    },
    created() {
        this.listarNotas()
    },
    methods: {
        alerta() {
            this.mensaje.color = 'bg-success';
            this.mensaje.texto = 'Probando alerta';
            this.showAlert();
        },
        listarNotas() {
            this.axios.get('/nota')
                .then(res => {
                    this.notas = res.data;
                })
                .catch(error => {
                    console.error(error);
                })
        },
        agregarNota() {
            this.axios.post('/nueva-nota', this.nota)
                .then(res => {
                    this.notas.push(res.data);
                    this.nota.nombre = '';
                    this.nota.descripcion = '';
                    this.mensaje.color = 'success';
                    this.mensaje.texto = 'Nota agregada';
                    this.showAlert();
                })
                .catch(error => {
                    console.error(error);

                    if(error.response.data.error.errors.nombre.message){
                        this.mensaje.texto = error.response.data.error.errors.nombre.message
                    }else{
                        this.mensaje.texto = 'Error de sistema';
                    }

                    this.mensaje.color = 'danger';
                    this.showAlert();
                });
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
        
    },
    
}
</script>