<template>
    <div class="profile-container">
        <NavBar :usertype="user_type" :username="username"></NavBar>

        <!-- Row for holding sponsor User ID, sponsor Username, and update Username button -->
        <div class="row">

            <!-- Container for displaying sponsor User ID -->
            <div class="user-id-container">
                <p><strong>UserID: </strong>{{ user_id }}</p>
            </div>

            <!-- Container for displaying sponsor Username -->
            <div class="username-container">
                <div class="username">
                    <p><strong>Username: </strong>{{ username }}</p>
                </div>
            </div>
        </div>

        <!-- Row for holding sponsor Password, display drivers associated with the sponsor button, and update/show password buttons -->
        <div class="row">

            <!-- Container for displaying sponsor password -->
            <div class="password-container">
                <div class="password">
                    <p><strong>Password: </strong>{{ password_text }}</p>
                    <button @click="edit_password"><span>Edit Password</span></button>
                    <button class="show-password" @click="show_password"><span>{{ button_text }}</span></button>
                </div>
            </div>

            <!-- Container for button to display drivers associated with the sponsor -->
            <div class="drivers-container">
                <div class="drivers">
                    <p><strong>Drivers Associated with You: </strong></p>
                    <p>{{ display_drivers() }} </p>
                </div>
            </div>
        </div>

        <!-- Row for holding sponsor Email, sponsor Usertype (Sponsor), and update email button -->
        <div class="row">
            <div class="email-container">
                <div class="email">
                    <p><strong>Email: </strong>{{ email }}</p>
                    <button @click="edit_email"><span>Edit Email</span></button>
                </div>
            </div>
            <div class="user-type-container">
                <p><strong>UserType: </strong>{{ user_type }}</p>
            </div>
        </div>
    </div>
</template>

<script>
    import axios from 'axios';
    import NavBar from '../misc/NavBar.vue';

    export default {

        // Component name
        name: 'sponsor-profile',

        // Component specific variables and data
        data() {
            return {
                user_id: null,
                username: null,
                password: null,
                password_text: "***********************",
                email: null,
                user_type: "Sponsor",
                button_text: "Show Password",
                password_active: false,
                edit_username_active: false,
                edit_password_active: false,
                edit_email_active: false,
                drivers: [],
                allData: null,
                localhost_path: 'http://localhost:5000',
                production_path: "https://www.spacebarcowboys.com",
                path: null
            };
        },

        // Mounted function is used for doing operations right after the component
        // Is mounted and right before the component is shown to the user
        mounted() {

            // Preventing users from accessing the application without logging in
            if (sessionStorage.getItem('loggedIn') !== 'true') this.$router.push({name: 'login'});

            // Getting username from route URL and setting Axios API path to either
            // localhost or production
            this.username = this.$route.params.username;
            this.path = this.production_path;

            // Axios API call to python backend to get current user information
            axios.get(this.path + '/userinfo', {params: {username: this.username}})
                .then((res) => {
                    if (res.data.status === 'success') {
                        this.user_id = res.data.results[0][0];
                        this.password = res.data.results[0][1];
                        this.email = res.data.results[0][3];

                        if (sessionStorage.getItem('userID') !== this.user_id.toString()) this.$router.push({name: 'login'});

                        this.fetchDrivers();
                    }
                    else {
                        window.alert('Could not find this user, logging out now');
                        this.$router.push({name: 'login'});
                    }
                })
                .catch((error) => {
                    console.log(error);
                })
        },

        methods: {
            fetchAllData(){ //show records
                axios.get(this.path + '/', {params: {request: 'username'}})
                    .then(function(response){
                        this.allData = response.data.members;
                    });
            },

            // Method to get drivers associated with the sponsor from the database
            fetchDrivers() {

                // Axios API call to python backend to get driver associated with the 
                // sponsor from the databse
                axios.get(this.path + '/get-drivers', {params: {user_id: this.user_id}})
                    .then((res) => {
                        if (res.data.status === 'success') {
                            for (const name of res.data.results) {
                                this.drivers.push(name[0]);
                            }
                        }
                        else {
                            console.log('Unsuccessful');
                        }
                    })
                    .catch((error) => {
                        console.log(error);
                    })
            },
            
            // Method to show user password upon "view" button click
            // OR method to hide user password upon "hide" button click
            show_password() {
                if (this.password_active) {
                    this.password_text = "***********************";
                    this.password_active = false;
                    this.button_text = "Show Password";
                }
                else {
                    this.password_text = this.password;
                    this.password_active = true;
                    this.button_text = "Hide Password";
                }
            },

            // Method to change sponsor password
            edit_password() {

                // Getting new password from sponsor and checking to make sure it meets password requirements
                window.alert("Password must contain at least one upper and lower case letter, at least one number, and at least one special character.")
                let new_password = window.prompt("Enter new password");
                let update = false;
                let minMaxLength = /^[\s\S]{8,20}$/,
                    upper = /[A-Z]/,
                    lower  = /[a-z]/,
                    number = /[0-9]/,
                    special = /[ !"#$%&'()*+,\-./:;<=>?@[\\\]^_`{|}~]/;

                // Checking to see if a new password was inputted
                if (new_password.trim() === '' || new_password === null) {
                    window.alert("New password cannot be blank. Password must contain at least one upper and lower case letter, at least one number, and at least one special character.");
                    return;
                }

                // Checking password to make sure it meets all requirements
                if (minMaxLength.test(new_password) && upper.test(new_password) && lower.test(new_password) && number.test(new_password) && special.test(new_password)) {
                    update = true;
                }
                else {
                    update = false;
                }

                // Checking to see if new password is different from old password
                if(new_password === this.password){
                    window.alert("New password must be different than old password")
                    update = false;
                }

                if (!update) {
                    window.alert("New password must be at least 8 characters and no more than 20. Contain at least one upper and lower case letter, at least one number, and at least one special character.");
                    return;
                }

                // Axios API call to python backend to change password
                axios.post(this.path + '/edit', null, {params: {request: 'password', password: new_password, userid: 3}})
                    .then((res) => {
                        if (res.data.status === "success") {
                            this.password = new_password;
                            window.alert("Password change successful");
                        }
                        else {
                            window.alert("Cannot update password");
                        }
                        
                    })
                    .catch((error) => {
                        // esling-disable-next-line
                        console.log(error);
                    })
            },

            // Method to change sponsor email
            edit_email() {

                // Getting new email from sponsor
                let new_email = window.prompt("Enter new email");

                if (new_email.trim() === '' || new_email === null) {
                    window.alert("New email cannot be blank");
                    return;
                }

                // Axios API call to python backend to check for duplicate email
                axios.get(this.path + '/edit', {params: {request: 'email', email: new_email}})
                .then((res) => {
                    if (res.data.status === "success") {

                        // Axios API call to python backend to change sponsor email
                        axios.post(this.path + '/edit', null, {params: {request: 'email', email: new_email, userid: 3}})
                        .then((res) => {
                            if (res.data.status === "success") {
                                this.email = new_email;
                                window.alert("Email change successful");
                            }
                            else {
                                window.alert("Email change unsuccessful");
                            }
                        })
                        .catch((error) => {
                            // esling-disable-next-line
                            console.log(error);
                        })
                    }
                    else {
                        window.alert("That email is unavailable.");
                    }
                })
            },

            display_drivers() {
                let drivers_string = '';

                for (let i = 0; i < this.drivers.length - 1; i++) {
                    drivers_string += this.drivers[i] + ', ';
                }

                drivers_string += this.drivers[this.drivers.length - 1];

                return drivers_string;
            }
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
        background-color: #73bfb8;
        overflow-y: auto;
    }

    .password, .username, .email, .drivers {
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

    .user-id-container, .username-container, .password-container, .email-container, .user-type-container, .drivers-container {
        display: flex;
        width: 49%;
        border-style: solid;
        border-color: black;
        justify-content: start;
        align-items: center;
        overflow-x: auto;
    }

    button {
        margin-right: 5px;
        margin-left: auto;
        display: block;
        width: 150px;
        height: 30px;
        font-size: 13px;
        text-decoration: none;
        color: black;
        border: 2px solid black;
        letter-spacing: 1px;
        text-align: center;
        position: relative;
        transition: all 0.35s;
    }

    button span {
        position: relative;
        z-index: 2;
    }

    button::after {
        position: absolute;
        content: "";
        top: 0;
        left: 0;
        width: 0;
        height: 100%;
        background: darkorange;
        transition: all 0.35s;
    }

    button:hover {
        color: white;
        cursor: pointer;
    }

    button:hover::after {
        width: 100%;
    }

    .password-container .password .show-password .show-drivers{
        margin-left: 0;
    }

    p {
        margin-left: 5px;
    }
</style>
