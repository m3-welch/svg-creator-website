<template>
    <!-- Top Menu Bar (Title, Export/Device/Save, Signin/Logout) -->
    <TopMenuBar
    ref="TopMenuBar"
    @toggle-sign-in-form="showSignInForm = !showSignInForm"
    @change-device="changeDevice"
    @show-export-menu="showExportMenu = !showExportMenu"
    :loggedIn="loggedIn()"/>

    <!-- Side bar panel to add controls to the canvas -->
    <LeftControlPicker 
    @add-control="addControl"/>

    <!-- Sign in form (shown when login-button is pressed) -->
    <transition
    name="show-hide-sign-in-form"
    enter-active-class="animate__animated animate__fadeInDown animate__faster"
    leave-active-class="animate__animated animate__fadeOutUp animate__faster"
    >
    <SignInForm 
    v-if="showSignInForm"
    @display-message="displayMessage"
    @close-sign-in-form="showSignInForm = !showSignInForm"
    @show-sign-up-form="showSignUpForm = !showSignUpForm; showSignInForm = !showSignInForm;"
    @successful-sign-in="successfulSignIn"
    @show-logout-confirm="this.showLogoutConfirm = true"/>
    </transition>

    <!-- Sign up form (shown when signup-button is pressed or if no account detected from cookies) -->
    <transition
    name="show-hide-sign-up-form"
    enter-active-class="animate__animated animate__fadeInDown animate__faster"
    leave-active-class="animate__animated animate__fadeOutUp animate__faster">
        <SignUpForm
        @display-message="displayMessage"
        v-if="showSignUpForm"
        @close-signup-form="this.showSignUpForm = false"/>
    </transition>

    <!-- Sign up form (shown when signup-button is pressed or if no account detected from cookies) -->
    <transition
    name="show-hide-export-menu"
    enter-active-class="animate__animated animate__fadeInDown animate__faster"
    leave-active-class="animate__animated animate__fadeOutUp animate__faster">
        <ExportMenu
        @display-message="displayMessage"
        v-if="showExportMenu"
        @close-export-menu="this.showExportMenu = false"/>
    </transition>

    <!-- Logout Confirm (shown when user tries to logout) -->
    <transition
    name="show-hide-logout-confirm"
    enter-active-class="animate__animated animate__fadeInDown animate__faster"
    leave-active-class="animate__animated animate__fadeOutUp animate__faster">
        <LogoutConfirm
        @display-message="displayMessage"
        v-if="showLogoutConfirm"
        @logout="logout"
        @close-logout-confirm="this.showLogoutConfirm = false; this.showSignInForm = false"/>
    </transition>
    
    <!-- Custom Device Size Form (shown when selecting 'Custom' in Top Menu Bar dropdown) -->
    <transition
    name="show-hide-custom-size-form"
    enter-active-class="animate__animated animate__fadeInDown animate__faster"
    leave-active-class="animate__animated animate__fadeOutUp animate__faster">
        <CustomSizeForm
        @display-message="displayMessage"
        v-if="showCustomSizeForm"
        @set-custom-size-device="setCustomSizeDevice"
        @close-custom-size-form="this.showCustomSizeForm = false"/>
    </transition>

    <!-- Message dialog to inform user of system events -->
    <transition
    name="show-hide-message"
    enter-active-class="animate__animated animate__fadeInDown animate__faster"
    leave-active-class="animate__animated animate__fadeOutUp animate__faster"
    >
    <Message
    v-if="showMessage"
    :messageType="messageType"
    :messageContent="messageContent"
    @close-message="showMessage = false"/>
    </transition>

    <!-- Canvas (the main body of the document) -->
    <Canvas 
    ref="canvas"
    @show-custom-size-form="showCustomSizeForm = !showCustomSizeForm"/>
</template>

<script>
import TopMenuBar from "./components/TopMenuBar.vue"
import LeftControlPicker from "./components/LeftControlPicker.vue"
import SignInForm from "./components/SignInForm.vue"
import Message from "./components/Message.vue"
import SignUpForm from "./components/SignUpForm.vue"
import Canvas from "./components/Canvas.vue"
import CustomSizeForm from "./components/CustomSizeForm.vue"
import LogoutConfirm from "./components/LogoutConfirm.vue"
import ExportMenu from "./components/ExportMenu.vue"

export default {
    name: "App",
    components: {
        TopMenuBar,
        LeftControlPicker,
        SignInForm,
        Message,
        SignUpForm,
        Canvas,
        CustomSizeForm,
        LogoutConfirm,
        ExportMenu,
    },
    data() {
        return {
            showSignInForm: false,
            showMessage: false,
            messageType: '',
            messageContent: '',
            showSignUpForm: false,
            showCustomSizeForm: false,
            custom_size: null,
            showLogoutConfirm: false,
            showExportMenu: false,
        }
    },
    methods: {
        changeDevice(device) {
            this.$refs.canvas.changeDevice(device)
        },
        displayMessage(messageType, messageContent) {
            this.messageType = messageType
            this.messageContent = messageContent
            this.showMessage = true
            setTimeout(() => this.showMessage = false, 4000)
        },
        loggedIn() {
            return (localStorage.getItem(btoa('user')) !== null)
        },
        displaySignUpForm() {
            this.showSignUpForm = true;
        },
        addControl(color, size, type, shape, annotations) {
            this.$refs.canvas.addControl(color, size, type, shape, annotations)
        },
        successfulSignIn() {
            this.$refs.TopMenuBar.$refs.SignIn.signIn()
        },
        logout() {
            localStorage.clear()
            this.showSignInForm = false
            this.$refs.TopMenuBar.$refs.SignIn.signIn()
        }
    },
    async mounted() {
        if (await this.loggedIn() == false) {
            setTimeout(() => {
                this.displaySignUpForm()
            }, 500);
        }
    }
};
</script>

<style>
    body {
        margin: 0;
        font-family: 'Roboto', sans-serif;
        height: 100%;
        overflow: hidden;
    }

    html {
        background-color: #F4F4F9;
        height: 100%;
    }

    #app {
        height: 100%;
    }

    .top-menu-bar {
        z-index: 100;
    }

    .sign-in-form {
        z-index: 50;
    }

    .left-control-picker {
        float: left;
    }
</style>
