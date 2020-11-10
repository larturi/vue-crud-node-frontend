<template>
    <div class="container">
        <h1>Login</h1>

        <form @submit.prevent="login">
            <input type="email" placeholder="Email" class="form-control my-2" v-model="usuario.email">
            <input type="password" placeholder="Password" class="form-control my-2" v-model="usuario.password">

            <b-button type="submit" class="btb btn-primary btn-block">Acceder</b-button>
        </form>

        <div v-if="mensaje!==''" class="my-4">
            <p>{{ mensaje }}</p>
        </div>
    </div>
</template>

<script>
import { mapActions } from 'vuex';

export default {

    data() {
        return {
            usuario: {email: '', password: ''},
            mensaje: ''
        }
    },

    methods: {
        ...mapActions(['guardarUsuario']),
        login() {
            this.axios.post('/login', this.usuario)
                .then(res => {
                    console.log(res);
                    const token = res.data.token;
                    this.guardarUsuario(token);
                })
                .catch(error => {
                    this.mensaje = error.response.data.mensaje;
                    console.log(error.response.data.mensaje);
                });
        }
    },
    
}
</script>