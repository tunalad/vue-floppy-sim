<script setup>
import Navbar from "./components/Navbar.vue";
import Dropbox from "./components/Dropbox.vue";
</script>

<template>
    <Navbar />
    <div class="container">
        <Dropbox />
    </div>
</template>

<style scoped>
.container {
    display: flex;
    justify-content: center;
    align-items: center;

    padding: 1.5rem 0;
}
</style>
