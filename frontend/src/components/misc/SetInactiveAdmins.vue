<template>
    <div class="profile-container">
        <form style="max-width:800px;margin:auto">
            <h1>Set Users Inactive</h1>
            <div>Select User:</div>

            <!-- Dropdown menu for user selection -->
            <select name = "selected" @change="onChange($event)" required>
                <option disabled value="">Please select one user to make inactive:</option>
                <option value="None">None</option>
                <option v-for="user in users" :key="user">{{ user }}</option>
            </select>

            <br><br>

            <button type="submit" class="btn" @click="submit_inactivation" >Submit</button>
        </form>
        <div class="admin-dashboard-button">
            <button @click="go_to_admin_dashboard">Return to Dashboard</button>
        </div>
    </div>
</template>

<script>
    import axios from 'axios';

    export default {

        // Component name
        name: "set-inactive-admins",

        // Component specific variables and data
        data() {
            return {
                username: null,
                user_id: null,
                users: [],
                production_path: "https://www.spacebarcowboys.com",
                localhost_path: "http://localhost:5000",
                path: null,
                user_selected:''
            };
        },

        // Component specific methods
        methods: {

            // Method to select a user based on dropdown menu selection
            onChange(e)
            {
                this.user_selected=e.target.value;
            },

            // Method to route user back to dashboard
            go_to_admin_dashboard() {
                this.$router.push({
                    name: 'admin-dashboard',
                    params: { username: this.username }
                });
            },

            // Method to get all active users
            fetchUsers() {

                // Axios API call to python backend to get all active users from database
                axios.get(this.path + '/get-users')
                    .then((res) => {
                        if (res.data.status === 'success') {
                            for (const name of res.data.results) {
                                this.users.push(name[0]);
                            }
                        }
                    })
                    .catch((error) => {
                        console.log(error);
                    })
            },

            // Method to change selected user to inactive in database
            submit_inactivation() { 
                console.log(this.user_selected);
                
                if (this.user_selected.trim() === '' || this.user_selected === null) {
                    window.alert("Please select a user to set inactive");
                    return;
                }

                // Axios API call to python backend to set selected user to inactive in database
                axios.post(this.path + '/deactivate-user-admin', null, {params: {username: this.user_selected}}) 
                    .then((res) => {
                        if (res.data.status === "success") {
                            console.log("success");
                        }
                        else {
                            window.alert("Cannot set user inactive.");
                        }
                    })
                    .catch((error) => {
                        // esling-disable-next-line
                        console.log(error);
                    })
            },
        },

        // Mounted function is used for doing operations right after the component
        // Is mounted and right before the component is shown to the user
        mounted() {

            // Preventing users from accessing the application without logging in
            if (sessionStorage.getItem('loggedIn') !== 'true') this.$router.push({name: 'login'});

            // Getting username from route URL and setting Axios API path to either
            // localhost or production
            this.path = this.production_path;
            this.username = this.$route.params.username;

            // Axios API call to python backend to get current user information
            axios.get(this.path + '/userinfo', {params: {username: this.username}})
                .then((res) => {
                    if (res.data.status === 'success') {
                        this.user_id = res.data.results[0][0];

                        if (sessionStorage.getItem('userID') !== this.user_id.toString()) this.$router.push({name: 'login'});

                        this.fetchUsers();
                    }
                    else {
                        window.alert('Could not find this user, logging out now');
                        this.$router.push({name: 'login'});
                    }
                })
                .catch((error) => {
                    console.log(error);
                });
        },
    }
</script>

<style scoped>
    * {
        box-sizing: border-box;
    }

    .profile-container {
        display: flex;
        flex-direction: column;
        width: 98.5vw;
        height: 97.5vh;
        border-style: solid;
        border-color: black;
        gap: 1rem;
        background-color: #ff90b3;
        overflow-y: auto;
    }

    .btn {
        background-color: #8c72e0;
        color: white;
        padding: 15px 20px;
        border: none;
        cursor: pointer;
        width: 100%;
        opacity: 0.9;
    }

    .btn:hover {
        opacity: 1;
    }
</style>