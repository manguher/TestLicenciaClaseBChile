<script setup>
import Header from './layouts/Header.vue';
import Main from './layouts/Main.vue';
import Footer from './layouts/Footer.vue';
import TestTeorico from './components/TestTeorico.vue';
</script>

<template>
    <!-- <Header/> -->
    
    <!-- Puedes elegir dónde mostrar el test: dentro de Main o independiente -->
    <!-- Opción 1: Mostrar el test como contenido principal -->
    <div class="container mx-auto px-4 py-8">
        <TestTeorico />
    </div>
    
    <!-- Opción 2: Mantener el contenido original -->
    <!-- <Main/> -->

    <!-- <Footer /> -->
</template>

<style scoped></style>
