<template>
    <div class="load-form">
        <div class="form-container">
            <div class="load-form-container">
                <div class="welcome-message">
                    Select a file to load:
                </div>
                <div class="load-form-conditional" v-if="loaded">
                    <table class="file-list">
                        <tr>
                            <th>File Name</th>
                            <th>Created At</th>
                            <th>Updated At</th>
                        </tr>
                        <tr
                        v-for="file in files"
                        :key="file.id"
                        class="item"
                        @click="load(file.id)">
                            <td> {{ file.save_name }} </td>
                            <td> {{ file.created_at }} </td>
                            <td> {{ file.updated_at }} </td>
                        </tr>
                    </table>
                </div>
                <div class="load-form-conditional" v-else>
                    <div class="lds-ellipsis"><div></div><div></div><div></div><div></div></div>
                </div>

                <div class="item skip" @click="closeLoadForm">
                    <span class="skip">Cancel Load</span>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import axios from 'axios'

export default {
    name: 'LoadForm',
    data() {
        return {
            loaded: false,
            response: '',
            files: null
        }
    },
    computed: {
        status() {
            if (this.error.length !== 0) {
                return this.error.response.status
            } else if (this.response.length !== 0) {
                return this.response.status
            } else {
                return 500;
            }
        }
    },
    async mounted() {
        if (localStorage.getItem(btoa('user')) == null) {
            alert('You must be signed in to load a file')
            this.closeLoadForm
        }

        await this.getFileList()

        // Logic for response
        if (this.status == 200) {
            this.files = JSON.parse(this.response.file_list)
            this.messageContent = 'Successfully retrieved file list'
            this.messageType = 'success'
        } else if (this.status == 400) {
            this.messageContent = this.response.data.message.toString()
            this.messageType = 'warning'
            this.password = ''
        } else if (this.error == 500) {
            console.log(this.error)
            this.messageContent = 'Something went wrong. Please try again'
            this.messageType = 'error'
            this.password = ''
        }

        this.$emit('display-message', this.messageType, this.messageContent)
        this.loaded = true
    },
    methods: {
        closeLoadForm() {
            this.$emit('close-load-form')
        },
        async load(id) {
            if (id === '') {
                alert('You must select a file to load.')
                return
            }

            await this.getResponse()

            // Logic for response
            if (this.status == 200) {
                this.messageContent = 'Loaded ' + this.saveName
                this.messageType = 'success'
                this.username = ''
                this.email = ''
                this.password = ''
                this.$emit('close-load-form')
            } else if (this.status == 400) {
                this.messageContent = this.response.data.message.toString()
                this.messageType = 'warning'
                this.password = ''
            } else if (this.error == 500) {
                console.log(this.error)
                this.messageContent = 'Something went wrong. Please try again'
                this.messageType = 'error'
                this.password = ''
            }

            this.$emit('display-message', this.messageType, this.messageContent)
        },
        getFileListFromCloud() {
            // Set Content-Type header
            axios.defaults.headers.post['Content-Type'] = 'application/x-www-form-urlencoded;';

            var config = {
                headers: {
                    Authorization: 'Bearer ' + atob(localStorage.getItem(btoa('token'))) 
                }
            }

            // Call the API
            return axios.post(process.env.VUE_APP_SVG_API_URL + '/cloud/list', {params: {'data': 'data'}}, config)
        },
        async getResponse() {
            await this.loadFromCloud()
            .then((response) => {
                this.response = response.data
                this.error = ''
            })
            .catch((error) => {
                if (error.response) {
                    this.response = error.response 
                }
                this.error = error
            })
        },
        async getFileList() {
            await this.getFileListFromCloud()
            .then((response) => {
                this.response = response.data
                this.error = ''
            })
            .catch((error) => {
                if (error.response) {
                    this.response = error.response 
                }
                this.error = error
            })
        }
    }
}
</script>

<style scoped>

.file-list {
    width: 100%;
    border: none;
}

th, td {
    border-bottom: 1px solid #B8DBD9;
}

td {
    text-align: center;
    padding: 10px;
}

tr:hover {
    cursor: pointer;
}

.lds-ellipsis {
  display: inline-block;
  position: relative;
  width: 80px;
  height: 80px;
}
.lds-ellipsis div {
  position: absolute;
  top: 33px;
  margin-left: 295px;
  width: 13px;
  height: 13px;
  border-radius: 50%;
  background: #586F7C;
  animation-timing-function: cubic-bezier(0, 1, 1, 0);
}
.lds-ellipsis div:nth-child(1) {
  left: 8px;
  animation: lds-ellipsis1 0.6s infinite;
}
.lds-ellipsis div:nth-child(2) {
  left: 8px;
  animation: lds-ellipsis2 0.6s infinite;
}
.lds-ellipsis div:nth-child(3) {
  left: 32px;
  animation: lds-ellipsis2 0.6s infinite;
}
.lds-ellipsis div:nth-child(4) {
  left: 56px;
  animation: lds-ellipsis3 0.6s infinite;
}
@keyframes lds-ellipsis1 {
  0% {
    transform: scale(0);
  }
  100% {
    transform: scale(1);
  }
}
@keyframes lds-ellipsis3 {
  0% {
    transform: scale(1);
  }
  100% {
    transform: scale(0);
  }
}
@keyframes lds-ellipsis2 {
  0% {
    transform: translate(0, 0);
  }
  100% {
    transform: translate(24px, 0);
  }
}

div .skip {
    text-align: center;
    color: #586F7C;
    margin-top: 10px;
    cursor: pointer;
}

div .skip :hover {
    color: #B8DBD9;
}

span .skip {
    width: 100%;
    box-sizing: border-box;
    height: 35px;
    outline: none;
    border: none;
    font-size: 15px;
    display: inline-block;
}

.text {
    border-bottom: 2px black solid;
}

.form-container {
    width: 100%;
    top: 20%;
    position: absolute;
    margin-left: auto;
    margin-right: auto;
}

.load-form {
    position: absolute;
    height: 100%;
    width: 100%;
    background-color: #00000055;
    z-index: 200;
}

.load-form-container {
    max-width: 35%;
    background-color: #F4F4F9;
    border: solid #B8DBD9 3px;
    border-radius: 2px;
    z-index: 200;
    margin-left: auto;
    margin-right: auto
}

.item {
    margin-left: auto;
    margin-right: auto;
    margin-bottom: 10px;
    width: 65%;
}

input {
    width: 100%;
    box-sizing: border-box;
    padding: 5px;
    height: 35px;
    outline: none;
    background-color: #F4F4F9;
    border: none;
    font-size: 20px;
    margin-left: auto;
    margin-right: auto;
}

input[type='button']:hover {
    border-bottom: 2px #586F7C solid;
    cursor: pointer;
}

.button {
    text-align: center; 
    border: 2px black solid;
    width: 49%;
    margin: auto;
    margin-bottom: 5px;
    border-bottom: 4px black solid;
}

.welcome-message {
    margin-left: auto;
    margin-right: auto;
    line-height: 50px;
    position: relative;
    left: 10px;
    font-weight: bold;
    font-size: 25px;
    color: #586F7C;
    text-align: center;
}

.title {
    color: #2F4550;
}


</style>
