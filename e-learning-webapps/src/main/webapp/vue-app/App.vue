<template>
    <div id="app">
        <app-nav-main></app-nav-main>
        <router-view />
    </div>
</template>

<script>

    import AppNavMain from './components/commun/AppNavMain.vue'

    export default {
        components:{AppNavMain},
    }
</script>


