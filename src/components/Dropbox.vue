<script setup lang="ts">
    import { ref } from 'vue';
    import moment from 'moment';

    const dropboxFiles = ref([])

    const tableSettings = ref({
        size: 1.44,
        format: "MB",
        column: "name",
        ascending: true,
        timeFormat: "",
    })

    function toMB(bytes: number) {
        return (bytes / (1024*1024)).toFixed(1)
    }

    function toKB(bytes: number) {
        return (bytes / 1024).toFixed(1)
    }

    function outOfSpace(maxAllowedSizeMB: number) {
        const maxSizeBytes = maxAllowedSizeMB * 1048576; // Convert MB to bytes
        const totalSizeBytes = dropboxFiles.value.reduce((sum, file) => sum + file.size, 0);
        return totalSizeBytes >= maxSizeBytes;
    }

    function formatTime(time: string, stringFormat="DD/MM/YYYY, HH:mm"){
        return moment(time).format(tableSettings.value.timeFormat || stringFormat)
    }

    function addFile(e: any){
        const files = (e.target as HTMLInputElement).files

        for (let f = 0; f < files.length; f++ ) {
            const file = files[f]
            const newFile = {
                name: file.name,
                type: file.type,
                size: file.size,
                unixtime: Date.now(),
            }

            dropboxFiles.value.push(newFile)
        }
        sortFiles(tableSettings.value.column);
    }
    
    function removeFile(file: any){
        if(typeof file === "string" && file === "all") {
            dropboxFiles.value = []
            return
        }
        dropboxFiles.value = dropboxFiles.value.filter(i => i.name !== file.name)
    }

    function sortFiles(column: string, reoder=false){
        // reodering 
        if(tableSettings.value.column === column && reoder)
            tableSettings.value.ascending = !tableSettings.value.ascending;
        else
            tableSettings.value.ascending = true

        tableSettings.value.column = column;

        // sorting
        if(tableSettings.value.column === "name")
            dropboxFiles.value.sort((a, b) => {
                if (a.name.toUpperCase() < b.name.toUpperCase())
                    return tableSettings.value.ascending ? -1 : 1;
                else if (a.name.toUpperCase() > b.name.toUpperCase())
                    return tableSettings.value.ascending ? 1 : -1;
                else
                    return 0;
            })

        if(tableSettings.value.column === "size")
            dropboxFiles.value.sort((a, b) => {
                if (a.size < b.size)
                    return tableSettings.value.ascending ? -1 : 1;
                else if (a.size > b.size)
                    return tableSettings.value.ascending ? 1 : -1;
                else
                    return 0;
            })

        if(tableSettings.value.column === "type")
            dropboxFiles.value.sort((a, b) => {
                if (a.type.toUpperCase() < b.type.toUpperCase())
                    return tableSettings.value.ascending ? -1 : 1;
                else if (a.type.toUpperCase() > b.type.toUpperCase())
                    return tableSettings.value.ascending ? 1 : -1;
                else
                    return 0;
            })
        if(tableSettings.value.column === "date")
            dropboxFiles.value.sort((a, b) => {
                if (a.unixtime < b.unixtime)
                    return tableSettings.value.ascending ? -1 : 1;
                else if (a.unixtime > b.unixtime)
                    return tableSettings.value.ascending ? 1 : -1;
                else
                    return 0;
            })
    }
</script>

<template>
    <div>
        <div class="floppy-stats">
            <!-- top -->
            <div class="floppy-stats-row">
                <div class="floppy-size-type">
                    <span>floppy size: </span>
                    <select v-model="tableSettings.size">
                        <option :value="0.11">8-inch (110 KB)</option>
                        <option :value="0.36">5.25-inch (360 KB)</option>
                        <option :value="1.44">3.5-inch (1.44 MB)</option>
                    </select>
                </div>

                <div class="floppy-size-format">
                    <form>
                        <span>shown in:</span> <br>
                        <input type="radio" id="MB" value="MB" v-model="tableSettings.format"> <label for="MB">MB</label>
                        <input type="radio" id="KB" value="KB" v-model="tableSettings.format"> <label for="KB">KB</label>
                    </form>
                </div>
            </div>

            <!-- middle -->
            <div class="time-format">
                <span>time format: </span>
                <input type="text" v-model="tableSettings.timeFormat" placeholder="DD/MM/YYYY, HH:mm">
                <br>
                <small>(time format documentation can be found <a target="_blank" href="https://momentjs.com/docs/#/displaying/format/">here</a>)</small>
            </div>

            <!-- bottom -->
            <div class="floppy-stats-row">
                <div class="floppy-size-total">
                    <p :class="{ 'error' : outOfSpace(tableSettings.size) }" 
                     v-if="tableSettings.format === 'MB'">{{ toMB(dropboxFiles.reduce((sum, file) => sum + file.size, 0)) }} MB / {{ tableSettings.size }} MB</p>
                    <p :class="{ 'error' : outOfSpace(tableSettings.size) }"
                     v-else>{{ toKB(dropboxFiles.reduce((sum, file) => sum + file.size, 0)) }} KB / {{ Math.floor(tableSettings.size * 1000)}} KB</p>
                </div>

                <input type="file" @change="addFile" class="file-input" multiple>
            </div>
        </div>
        <div class="dropbox">
            <div v-if="dropboxFiles.length < 1">
                <p>drop shyt here</p>
            </div> 
            <table class="dropbox-table" v-else>
                <tr>
                    <th @click="removeFile('all')"><button>🗑️</button></th>
                    <th><a @click="sortFiles('name', true)">name</a></th>
                    <th><a @click="sortFiles('size', true)">size</a></th>
                    <th><a @click="sortFiles('type', true)">file type</a></th>
                    <th><a @click="sortFiles('date', true)">date added</a></th>
                </tr>
                <tr v-for="file in dropboxFiles" :class="{ 'error': file.size >= tableSettings.size * 1048576 }">
                    <td><button href="#" @click="removeFile(file)">🗑️</button></td>
                    <td>{{ file.name }}</td>
                    <td>
                        <span v-if="tableSettings.format === 'MB'">{{ toMB(file.size) }} MB</span>
                        <span v-else>{{ toKB(file.size) }} KB</span>
                    </td>
                    <td>{{ file.type }}</td>
                    <td>{{ formatTime(file.unixtime) }}</td>
                </tr>
            </table>
        </div>
    </div>
</template>

<style scoped>
.dropbox {
    min-width: 800px;
    min-height: 400px;
    background-color: rgba(0, 0, 0, 0.2);

    p {
        padding: 25%;
        vertical-align: middle;
    }
}

.dropbox-table {
    width: 100%;
}

.error {
    color: red;
}

.file-input {
    width: 100px;
    overflow: hidden;
    color: transparent;
    margin: auto 0;
}

.floppy-stats {
    display: flex;
    flex-direction: column;
    align-items: center;
}

.floppy-stats-row {
    display: flex;
    justify-content: center;
}

.floppy-size-type {
    display: flex;
    align-items: center;
    justify-content: center;
}

p {
    padding: 0 0.5rem;
}

span {
    padding: 0 0.5rem;
}

a {
    cursor: pointer;
}

tr {
    text-align: left;
    padding: 0 0.5rem;
}

small {
    opacity: 50%;
}
</style>
