<template>
    <div class="settings-container">
        <NavBar :usertype="user_type" :username="username"></NavBar>
        <form style="max-width:800px;margin:auto">
            <h1>New Driver</h1>
            <div>Select Sponsor:</div>

            <!-- Sponsor selection dropdown menu -->
            <select name = "selected" @change="onChange($event)" v-model="sponsor_selected" required>
                <option disabled value="">Please select one sponsor you would like to apply to</option>
                <option v-for="sponsor in sponsors" :key="sponsor">{{ sponsor[0] }}</option>
            </select>

            <br><br>

            <!-- First Name Input Container -->
            <div class="input-container">
                <input class="input-field" type="text" placeholder="First Name" name="first_name" v-model="first_name" required><br><br>
            </div>

            <!-- Last Name Input Container -->
            <div class="input-container">
                <input class="input-field" type="text" placeholder="Last Name" name="last_name" v-model="last_name" required><br><br>
            </div>

            <!-- Username Input Container -->
            <div class="input-container">
                <input class="input-field" type="text" placeholder="Username" name="username" v-model="driver_username" required><br><br>
            </div>

            <!-- Email Input Container -->
            <div class="input-container">
                <input class="input-field" type="text" placeholder="Email" name="email" v-model="email" required><br><br>
            </div>
            
            <!-- Password Input Container -->
            <div class="input-container">
                <input class="input-field" type="password" placeholder="Password" name="password" v-model="password" required><br><br>
            </div>

            <!-- Submit button to create a new driver -->
            <button type="button" class="btn" @click="create_driver" >Create</button> 
        </form>
    </div>

</template>

<script>
    import axios from 'axios';
    import NavBar from '../misc/NavBar.vue';


    export default {

        // Component name
        name: "new-driver",

        // Component specific variables and data
        data() {
            return {
                status: null,
                first_name: '',
                last_name:'',
                username: '',
                driver_username: '',
                password: '',
                email: '',
                user_type: '',
                sponsors: [],
                sponsor_selected: '',
                production_path: "https://www.spacebarcowboys.com",
                localhost_path: "http://localhost:5000",
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
                        this.user_type = res.data.results[0][2];

                        if (res.data.results[0][0].toString() !== sessionStorage.getItem('userID')) this.$router.push({name: 'login'});
                        
                        this.fetchSponsors();
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

        // Component specific methods
        methods: {

            // Method to get specific sponsor from dropdown menu
            onChange(e)
            {
                this.sponsor_selected=e.target.value
            },

            // Method to get all available sponsors from database
            fetchSponsors() {

                // Axios API call to python backend to get available sponsors from database
                axios.get(this.path + '/get-sponsors', {params: {user_id: this.user_id}})
                    .then((res) => {
                        if (res.data.status === 'success') {
                            this.sponsors = res.data.results;
                        }
                        else {
                            console.log('Unsuccessful');
                        }
                    })
                    .catch((error) => {
                        console.log(error);
                    })
            },

            // Method to add a new driver to the database
            create_driver() {                 

                // Checking to see if all fields are filled out before submission
                if (this.first_name.trim() !== '' && this.last_name.trim() !== '' && this.driver_username.trim() !== '' && this.email.trim() !== '' && this.password.trim() !== '' && this.sponsor_selected.trim() !== '') {

                // Axios API call to python backend to check for duplicate users
                    axios.get(this.path + '/new-user', {params: {username: this.driver_username, email: this.email}})
                        .then((res) => {
                            if (res.data.status === 'success') {

                                // If there are no duplicate users, then create a new driver
                                if (res.data.results.length === 0) {

                                    // Axios API call to python backend to add new driver to database
                                    axios.post(this.path + '/new-driver', null, {params: {email: this.email, first_name: this.first_name, last_name: this.last_name, username: this.driver_username, password: this.password, sponsor: this.sponsor_selected}}) 
                                        .then((res) => {
                                            if (res.data.status === "success") {
                                                window.alert("Driver successfully created");
                                                this.first_name = '';
                                                this.last_name = '';
                                                this.driver_username = '';
                                                this.email = '';
                                                this.password = '';
                                                this.sponsor_selected = '';
                                            }
                                            else {
                                                window.alert("Cannot create driver.");
                                            }
                                        })
                                        .catch((error) => {
                                            // esling-disable-next-line
                                            console.log(error);
                                        });
                                }
                                else if (res.data.results.length === 1) {
                                    window.alert(`${res.data.results[0]} is already taken`);
                                }
                                else if (res.data.results.length === 2) {
                                    window.alert(`${res.data.results[0]} and ${res.data.results[1]} are already taken`);
                                }
                            }
                            else {
                                console.log('Failure');
                            }
                        })
                        .catch((error) => {
                            console.log(error);
                        });
                }
                else {
                    window.alert("All fields must be filled out to create a new driver");
                }
            }
        },

        // Components used from external files
        components: { NavBar }
    }

</script>

<style scoped>
    * {box-sizing: border-box;}

    .settings-container {
        /* display: flex; */
        flex-direction: column;
        width: 98.5vw;
        height: 97.5vh;
        border-style: solid;
        border-color: black;
        gap: 1rem;
        background-color: #ff90b3;
        overflow-y: auto;
    }

    .input-container {
        display: flex;
        width: 100%;
        margin-bottom: 15px;
    }

    /* Style the input fields */
    .input-field {
        width: 100%;
        padding: 10px;
        outline: none;
    }

    .input-field:focus {
        border: 2px solid #8c72e0;
    }

    /* Set a style for the submit button */
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