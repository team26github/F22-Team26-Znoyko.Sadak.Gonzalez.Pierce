<template>
    <div class="profile-container">
        <nav-bar :usertype="user_type" :username="username"></nav-bar>
        <br><br>

        <!-- Button to route user to add points to a driver as a sponsor page -->
        <div class="row">
            <div class="add-point-container">
                <button @click="go_to_add_points"><span>Add Points to Driver</span></button>
            </div>
        </div>

        <!-- Button to route user to remove points from a driver as a sponsor page -->
        <div class="row">
            <div class="remove-points-button">
                <button @click="go_to_remove_points"><span>Deduct Points from Driver</span></button>
            </div>
        </div>

        <!-- Button to route user to pending applications acceptance/rejection page -->
        <div class="row">
            <div class="pending-applications-button">
                <button @click="go_to_pending_applications"><span>Pending Applications</span></button>
            </div>
        </div>

        <!-- Button to route user to set a driver inactive as a sponsor page -->
        <div class="row">
            <div class="set-inactive-button">
                <button @click="go_to_set_inactive"><span>Set Drivers Inactive</span></button>
            </div>
        </div>
    </div>
</template>

<script>
    import NavBar from '../misc/NavBar.vue';
    import axios from 'axios';

    export default {

        // Component name
        name: 'SponsorDashboard',

        // Component specific variables and data
        data() {
            return {
                dashboard_name: `${this.username}'s Dashboard`,
                username: null,
                user_type: 'sponsor',
                active: false,
                production_path: 'https://www.spacebarcowboys.com',
                localhost_path: 'http://localhost:5000',
                path: null
            }
        },

        // Component specific methods
        methods: {

            // Method to route sponsor to pending applications acceptance/rejection page
            go_to_pending_applications() {
                this.$router.push({
                    name: 'pending-applications',
                    params: { username: this.username }
                });
            },

            // Method to route sponsor to set driver inactive page
            go_to_set_inactive() {
                this.$router.push({
                    name: 'set-inactive',
                    params: { username: this.username }
                });
            },

            // Method to route sponsor to add points to a driver page
            go_to_add_points() {
                this.$router.push({
                    name: 'add-points',
                    params: { username: this.username }
                });
            },

            // Method to route sponsor to remove points from a driver page
            go_to_remove_points() {
                this.$router.push({
                    name: 'remove-points',
                    params: { username: this.username }
                });
            }
        },

        // Mounted function is used for doing operations right after the component
        // Is mounted and right before the component is shown to the user
        mounted() {

            // Preventing users from accessing the application without logging in
            if (sessionStorage.getItem('loggedIn') !== 'true') this.$router.push({name: 'login'});

            this.username = this.$route.params.username;
            this.path = this.production_path;

            axios.get(this.path + '/userinfo', {params: {username: this.username}})
                .then((res) => {
                    if (res.data.status !== 'success') {
                        window.alert('Could not find this user, logging out now');
                        this.$router.push({name: 'login'});
                    }
                    else if (res.data.results[0][0].toString() !== sessionStorage.getItem('userID')) this.$router.push({name: 'login'});
                })
                .catch((err) => {
                    console.log(err);
                });
        },

        // Components used from external files
        components: {
            "nav-bar": NavBar
        }
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

    .row {
        width: 100%;
        height: 45px;
        display: flex;
        flex-direction: row;
        justify-content: center;
        gap: 1rem;
    }
    
    button {
        margin-right: 5px;
        margin-left: 5px;
        display: block;
        width: 200px;
        height: 40px;
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
</style>
