<template>
    <div class="profile-container">
        <NavBar :usertype="user_type" :username="username"></NavBar>
    </div>  
</template>

<script>
    import NavBar from '../misc/NavBar.vue';
    import axios from 'axios';

    export default {

        // Component name
        name: "driver-settings",

        // Component specific variables and data
        data() {
            return {
                user_type: "driver",
                username: null,
                localhost_path: "http://localhost:5000",
                production_path: "https://www.spacebarcowboys.com",
                path: null
            };
        },

        // Mounted function is used for doing operations right after the component
        // Is mounted and right before the component is shown to the user
        mounted() {
            this.username = this.$route.params.username;
            this.path = this.localhost_path;

            axios.get(this.path + '/userinfo', {params: {username: this.username}})
                .then((res) => {
                    if (res.data.status !== 'success') {
                        window.alert('Could not find this user, logging out now');
                        this.$router.push({name: 'login'});
                    }
                })
                .catch((err) => {
                    console.log(err);
                });
        },

        // Components used from external files
        components: { NavBar }
    }
</script>

<style scoped>
    .profile-container {
        display: flex;
        flex-direction: column;
        width: 98.5vw;
        height: 97.5vh;
        border-style: solid;
        border-color: black;
        gap: 1rem;
        background-color: #0096c7;
    }

    .password, .username, .email {
        width: 100%;
        display: flex;
        flex-direction: row;
        align-items: center;
    }

    .row {
        width: 100%;
        height: 45px;
        display: flex;
        flex-direction: row;
        justify-content: center;
        gap: 1rem;
    }

    .user-id-container, .username-container, .password-container, .email-container, .user-type-container {
        display: flex;
        width: 49%;
        border-style: solid;
        border-color: black;
        justify-content: center;
        align-items: center;
    }

    button {
        margin-right: 5px;
        margin-left: auto;
    }

    .password-container .password .show-password {
        margin-left: 0;
    }

    p {
        margin-left: 5px;
    }

    .points-container {
        margin-left: 0;
    }

    p {
        margin-left: 5px;
    }
</style>