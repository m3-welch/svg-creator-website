<template>
    <div class="sign-up-form">
        <div class="form-container">
            <div class="logout-confirm">
                <div class="welcome-message">
                    Please enter a name to save your project as:
                </div>
                                
                <div class="item text">
                    <input type="text" id="saveName" placeholder="Project_1" v-model="saveName" required>
                </div>

                <div class="item button">
                    <input type="button" id="save" value="Save" @click="save">
                </div>

                <div class="item skip" @click="closeSaveForm">
                    <span class="skip">Cancel Save</span>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import axios from 'axios'

export default {
    name: 'SaveForm',
    props: {
        saveContent: String,
    },
    data() {
        return {
            saveName: '',
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
    methods: {
        closeSaveForm() {
            this.$emit('close-save-form')
        },
        async save() {
            if (localStorage.getItem(btoa('user')) == null) {
                alert('You must be signed in to save a project')
                return
            }
            if (this.saveName === '') {
                alert('You must give this project a name.')
                return
            }

            await this.getResponse()

            // Logic for response
            if (this.status == 200) {
                this.messageContent = 'Saved layout as ' + this.saveName
                this.messageType = 'success'
                this.username = ''
                this.email = ''
                this.password = ''
                this.$emit('close-save-form')
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
        saveToCloud() {
            // Set Content-Type header
            axios.defaults.headers.post['Content-Type'] = 'application/x-www-form-urlencoded;';

            var config = {
                headers: {
                    Authorization: 'Bearer ' + atob(localStorage.getItem(btoa('token'))) 
                }
            }

            var bodyParameters = {
                params: {
                    save_name: encodeURI(this.saveName),
                    saved_content: encodeURI(this.saveContent)
                }
            }

            // Call the API
            return axios.post(process.env.VUE_APP_SVG_API_URL + '/cloud/save', bodyParameters, config)
        },
        async getResponse() {
            await this.saveToCloud()
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

.sign-up-form {
    position: absolute;
    height: 100%;
    width: 100%;
    background-color: #00000055;
    z-index: 200;
}

.logout-confirm {
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
