<template>
    <div class="card" style="height: 100vh">
        <div class="card-header text-center">WELCOME TO LEDIS</div>
        <div class="card-body">
            <div :key="element.id" v-for="element in conversation">
                <div>> {{ element.req }}</div>
                <div>{{ element.res }}</div>
            </div>
        </div>
        <div class="card-footer text-muted containter-fluid" style="background-color: black">
            <div class="row">
                <form @submit="onSubmit">
                    <div class="input-group">
                        <div class="input-group-prepend">
                            <div class="input-group-text" style="color: white; background-color: black; border: 0">
                                >Ledis:</div>
                        </div>
                        <input type="text" class="form-control" id="inlineFormInputGroupUsername2"
                            style="color: white; background-color: black; border: 0" name="input" v-model="input" />
                    </div>
                </form>

            </div>
        </div>
    </div>
</template>
<script>
import { onMounted } from '@vue/runtime-core'
export default {
    name: 'CommandBox',
    data() {
        return {
            input: '',
            conversation: [],
            stored_data: {},
        }
    },
    mounted() {
        var get_data = localStorage.getItem("stored_data")
        console.log(get_data)
        if (get_data) {
            this.stored_data = JSON.parse(get_data)
            localStorage.clear()
        }
        window.addEventListener('beforeunload', this.beforWindowClose)
    },
    methods: {
        onSubmit(e) {
            e.preventDefault()
            if (!this.input) {
                alert('Please input command')
                return
            }
            let input_split = this.input.split(" ")
            var req_res = { id: this.conversation.length + 1, req: this.input }
            if (input_split[0].toUpperCase() === "SET") {
                this.setCommand(input_split[1], input_split[2])
                req_res["res"] = "OK"
            }
            if (input_split[0].toUpperCase() === "GET") {
                req_res["res"] = this.getCommand(input_split[1])
            }
            if (input_split[0].toUpperCase() === "SADD") {
                for (let index = 2; index < input_split.length; index++) {
                    req_res["res"] = "Current length of this set: " + this.saddCommand(input_split[1], input_split[index])
                }
            }
            if (input_split[0].toUpperCase() === "SREM") {
                for (let index = 2; index < input_split.length; index++) {
                    req_res["res"] = "Current length of this set: " + this.sremCommand(input_split[1], input_split[index])
                }
            }
            this.conversation.push(req_res)
            this.input = ''
        },

        beforWindowClose() {
            const string_result = JSON.stringify(JSON.parse(JSON.stringify(this.stored_data)))
            console.log(string_result)
            localStorage.setItem("stored_data", string_result)
        },

        setCommand(key, value) {
            this.stored_data[key] = value
        },

        getCommand(key) {
            return this.stored_data[key]
        },

        saddCommand(key, value) {
            if (!this.stored_data[key]) {
                this.stored_data[key] = []
            }
            this.stored_data[key].push(value)
            return this.stored_data[key].length
        },

        sremCommand(key, value) {
            this.stored_data[key] = this.stored_data[key].filter((element) => element !== value)
            return this.stored_data[key].length
        }
    },
}
</script>