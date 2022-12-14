<template>
    <div class="profile-container">

        <!-- Form to fill out for removing points from a driver as an admin -->
        <form style="max-width:800px;margin:auto">
            <h1>Remove Points from Drivers</h1>
            <div>Select Driver:</div>

            <!-- Dropdown menu to select driver to remove points from -->
            <select name = "selected" @change="onChange($event)" required>
                <option disabled value="">Please select a driver:</option>
                <option value="None">None</option>
                <option v-for="driver in drivers" :key="driver">{{ driver }}</option>
            </select>

            <br><br>

            <!-- Number of points to remove container -->
            <div class="input-container">
                <input class="input-field" type="text" placeholder="Number of points to remove" name="num_points" v-model="num_points" required><br><br>
            </div>

            <!-- Reason for deduction container -->
            <div class="input-container">
                <input class="input-field" type="text" placeholder="Reason for deduction" name="reason" v-model="reason" required><br><br>
            </div>

            <!-- Button for points deduction submission -->
            <button type="submit" class="btn" @click="submit_application" >Submit</button>
        </form>

        <!-- Button to route user back to admin dashboard -->
        <div class="sponsor-admin-button">
            <button @click="go_to_admin_dashboard">Return to Dashboard</button>
        </div>
    </div>
</template>

<script>
    import axios from 'axios';

    export default {

        // Component name
        name: "remove-points-admin",

        // Component specific variables and data
        data() {
            return {
                username: null,
                user_id: null,
                drivers: [],
                production_path: "https://www.spacebarcowboys.com",
                localhost_path: "http://localhost:5000",
                path: null,
                num_points: '',
                reason: '',
                driver_selected:''
            };
        },

        // Component specific methods
        methods: {

            // Method to get selected driver from dropdown menu
            onChange(e)
            {
                this.driver_selected=e.target.value;
            },

            // Method to route user back to admin dashboard
            go_to_admin_dashboard() {
                this.$router.push({
                    name: 'admin-dashboard',
                    params: { username: this.username }
                });
            },

            // Method to get all available drivers from database
            fetchDrivers() {

                // Axios API call to python backend to get all available drivers from database
                axios.get(this.path + '/get-drivers-admin', { params: { user_id: this.user_id } })
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

            // Method to update database with new points value for selected driver
            submit_application() {                 

                if ((this.num_points.trim() === '' || this.num_points === null) ||
                    (this.reason.trim() === '' || this.reason === null) ||
                    (this.driver_selected.trim() === '' || this.driver_selected === null)) {
                        window.alert("Please fill out all fields to remove points from a driver");
                        return;
                    }

                // Axios API call to python backend to update database with new points
                // value for selected driver
                axios.post(this.path + '/remove_points', null, {params: {num_points: this.num_points, reason: this.reason, driver: this.driver_selected, sponsor: this.user_id}}) 
                    .then((res) => {
                        if (res.data.status === "success") {
                            window.alert(`Successfully removed ${Math.abs(this.num_points)} to ${this.driver_selected}`)
                        }
                        else {
                            window.alert("Cannot remove points from driver");
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
    * {box-sizing: border-box;}

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