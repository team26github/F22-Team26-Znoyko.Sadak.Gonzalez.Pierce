<template>
    <div class="profile-container">
        <form style="max-width:800px;margin:auto">
            <h1>Set Drivers Inactive</h1>
            <div>Select Driver:</div>

            <!-- Dropdown menu for driver selection -->
            <select name = "selected" @change="onChange($event)" required>
                <option disabled value="">Please select one driver to make inactive:</option>
                <option value="None">None</option>
                <option v-for="driver in drivers" :key="driver">{{ driver }}</option>
            </select>

            <br><br>

            <button type="submit" class="btn" @click="submit_inactivation" >Submit</button>
        </form>
        <div class="sponsor-dashboard-button">
            <button @click="go_to_sponsor_dashboard">Return to Dashboard</button>
        </div>
    </div>
</template>

<script>
    import axios from 'axios';
    
    export default {

        // Component name
        name: "set-inactive",

        // Component specific variables and data
        data() {
            return {
                username: null,
                user_id: null,
                drivers: [],
                production_path: "https://www.spacebarcowboys.com",
                localhost_path: "http://localhost:5000",
                path: null,
                driver_selected:''
            };
        },

        // Component specific methods
        methods: {

            // Method to change driver selection based on dropdown menu selection
            onChange(e)
            {
                this.driver_selected=e.target.value;
            },

            // Method to route user back to dashboard
            go_to_sponsor_dashboard() {
                this.$router.push({
                    name: 'sponsor-dashboard',
                    params: { username: this.username }
                });
            },

            // Method to fetch all active drivers
            fetchDrivers() {

                // Axios API call to python backend to get all active drivers
                axios.get(this.path + '/get-drivers', { params: { user_id: this.user_id } })
                    .then((res) => {
                        if (res.data.status === 'success') {
                            for (const name of res.data.results) {
                                this.drivers.push(name[0]);
                            }
                        }
                    })
                    .catch((error) => {
                        console.log(error);
                    });
            },

            // Method to change selected driver to inactive in database
            submit_inactivation() {                 
                if (this.driver_selected.trim() !== '') {
                    
                    // Axios API call to python backend to deactive selected driver
                    axios.post(this.path + '/deactivate-user', null, {params: {username: this.driver_selected}}) 
                        .then((res) => {
                            if (res.data.status === "success") {
                                window.log(`${this.driver_selected} set inactive successfully`);
                            }
                            else {
                                window.alert("Cannot add points.");
                            }
                        })
                        .catch((error) => {
                            // esling-disable-next-line
                            console.log(error);
                        });
                }
                else {
                    window.alert('Please select a driver to set inactive');
                }
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

                        this.fetchDrivers();
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
        background-color: #73bfb8;
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