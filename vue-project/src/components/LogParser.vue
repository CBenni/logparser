<template>
    <div class="container">
        <textarea v-model="log" placeholder="Paste log here" @change="parseLog()"></textarea>
        <div class="log">
            <div v-for="log in parsedLog" :key="log.originalText">
                {{ log.preamble }}
                <JsonViewer v-if="log.parsed" :value="log.parsed" copyable boxed sort theme="jv-light" />
            </div>
        </div>
    </div>
</template>

<script>
import JsonViewer from "vue-json-viewer";

function unString(str) {
    try {
        return JSON.parse(str);
    } catch (e) {
        return str;
    }
}
function recursivelyUnString(obj) {
    if (typeof obj === 'object') {
        for (let key in obj) {
            if (typeof obj[key] === 'string') {
                obj[key] = unString(obj[key]);
            } else {
                obj[key] = recursivelyUnString(obj[key]);
            }
        }
    }
    return obj;
}

export default {
    data() {
        return {
            log: '',
            parsedLog: '',
            test: {"hello": "world", "nested": {"key": "value"}}
        };
    },
    methods: {
        parseLog() {
            this.parsedLog = this.log.split("\n").map(line => {
                // find JSON objects in the log and parse them
                const indexStart = line.indexOf('{');
                if (indexStart === -1) {
                    return {
                        originalText: line,
                        parsed: null,
                        preamble: line
                    };
                }
                let log = null;
                try {
                    log = recursivelyUnString(JSON.parse(line.slice(indexStart)));
                } catch(e) {
                    console.log("Failed decode on line", line, e)
                    return {
                        originalText: line,
                        parsed: null,
                        preamble: line
                    };
                }
                return {
                    originalText: line,
                    parsed: log,
                    preamble: line.slice(0, indexStart)
                };
            })
            console.log(this.parsedLog);
        }
    },
    components: {
        JsonViewer
    }
};
</script>

<style scoped>
.container {
    width: 100%;
    height: 90vh;
    display: flex;
    flex-direction: row;
    flex-grow: 1;
}

textarea {
    width: 500px;
    min-width: 500px;
    flex-grow: 0;
    flex-shrink: 0;
    height: 100%;
    padding: 10px;
    border: 1px solid #ccc;
    border-radius: 5px;
    margin-right: 20px;

}

.log {
    margin-top: 20px;
    padding: 10px;
    border: 1px solid #ccc;
    border-radius: 5px;
    background-color: #f9f9f9;
    white-space: pre-wrap;
    flex-grow: 1;
    font-size: small;
}
</style>