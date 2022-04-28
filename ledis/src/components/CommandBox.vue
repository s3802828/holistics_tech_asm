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
            expiration: {}
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
                this.set(input_split[1], input_split[2])
                req_res["res"] = "OK"
            }
            if (input_split[0].toUpperCase() === "GET") {
                req_res["res"] = this.get(input_split[1])
            }
            if (input_split[0].toUpperCase() === "SADD") {
                for (let index = 2; index < input_split.length; index++) {
                    req_res["res"] = "Current length of this set: " + this.sadd(input_split[1], input_split[index])
                }
            }
            if (input_split[0].toUpperCase() === "SREM") {
                for (let index = 2; index < input_split.length; index++) {
                    req_res["res"] = "Current length of this set: " + this.srem(input_split[1], input_split[index])
                }
            }
            if (input_split[0].toUpperCase() === "SMEMBERS") {
                const result = this.smembers(input_split[1])
                let res = ''
                for (let index = 0; index < result.length; index++) {
                    res += result[index] + ' '
                }
                req_res["res"] = res
            }

            if (input_split[0].toUpperCase() === "SINTER") {
                req_res["res"] = this.sinter(input_split.splice(1))
            }

            if (input_split[0].toUpperCase() === "KEYS") {
                req_res["res"] = this.keys()
            }

            if (input_split[0].toUpperCase() === "DEL") {
                this.del(input_split[1])
                req_res["res"] = "OK"
            }
            if (input_split[0].toUpperCase() === "EXPIRE") {
                this.expire(input_split[1], input_split[2])
                req_res["res"] = "OK"
            }
            if (input_split[0].toUpperCase() === "TTL") {
                req_res["res"] = this.ttl(input_split[1])
            }
            this.conversation.push(req_res)
            this.input = ''
        },

        beforWindowClose() {
            const string_result = JSON.stringify(JSON.parse(JSON.stringify(this.stored_data)))
            console.log(string_result)
            localStorage.setItem("stored_data", string_result)
        },

        set(key, value) {
            this.stored_data[key] = value
        },

        get(key) {
            return this.stored_data[key]
        },

        sadd(key, value) {
            if (!this.stored_data[key]) {
                this.stored_data[key] = []
            }
            this.stored_data[key].push(value)
            return this.stored_data[key].length
        },

        srem(key, value) {
            this.stored_data[key] = this.stored_data[key].filter((element) => element !== value)
            return this.stored_data[key].length
        },

        smembers(key) {
            return this.stored_data[key]
        },
        sinter(key) {
            let result = []
            const element = this.stored_data[key[0]]
            element.forEach(ele => {
                let count = 0
                for (let index = 1; index < key.length; index++) {
                    const key_value = this.stored_data[key[index]];
                    if (key_value.includes(ele)) {
                        count += 1
                    }
                }
                if (count == key.length - 1) {
                    result.push(ele)
                }
            });
            return result
        },

        keys() {
            return Object.keys(this.stored_data)
        },

        del(key) {
            delete this.stored_data[key]
        },
        expire: function(key, seconds) {
            var currentTime = Date.now()

            let interval = setInterval(() => {
                console.log((seconds - getTimeLeft()) + "s")
                this.expiration[key] = (seconds - getTimeLeft()) + "s"
                if (getTimeLeft() >= seconds){
                    this.del(key)
                    delete this.expiration[key]
                    clearInterval(interval)
                }
            }, 1000);

            function getTimeLeft() {
                return Math.ceil((Date.now() - currentTime) / 1000);
            }
        },
        ttl(key){
            if(!this.expiration[key]){
                return "null"
            }
            return this.expiration[key]
        }
    },
}
</script>