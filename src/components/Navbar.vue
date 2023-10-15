<template>
    <nav>
        <ul>
            <li><a href="#">floppy</a></li>
            <li><a href="#">about</a></li>
            <li><a href="#">settings</a></li>
        </ul>
    </nav>
</template>

<style scoped>
nav {
    background-color: purple;
    display: flex;
    justify-content: center;
    ul {
        list-style-type: none;
        padding: 0;
        margin: 0;
        overflow: hidden;

        li {
            display: inline;
            background-color: orange;
            padding: 0.2rem;
            margin: 0.2rem;
        }
    }
}

</style>
