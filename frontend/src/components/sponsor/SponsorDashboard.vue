<template>
    <div class="profile-container">
        <nav-bar :usertype="user_type" :username="username"></nav-bar>
        <br><br>
        <div class="row">
            <div class="add-point-container">
                <button @click="go_to_add_points"> Add Points to Driver</button>
            </div>
        </div>
        <div class="row">
            <div class="remove-points-button">
                <button @click="go_to_remove_points">Deduct Points from Driver</button>
            </div>
        </div>
        <div class="row">
            <div class="pending-applications-button">
                <button @click="go_to_pending_applications">Pending Applications</button>
            </div>
        </div>
        <div class="row">
            <div class="set-inactive-button">
                <button @click="go_to_set_inactive">Set Drivers Inactive</button>
            </div>
        </div>
    </div>
</template>

<script>
    import NavBar from '../misc/NavBar.vue';
    export default {
        name: 'SponsorDashboard',

        data() {
            return {
                dashboard_name: `${this.username}'s Dashboard`,
                username: null,
                user_type: 'sponsor',
                active: false
            }
        },

        methods: {
            go_to_pending_applications() {
                this.$router.push({
                    name: 'pending-applications',
                    params: { username: this.username }
                });
            },
            go_to_set_inactive() {
                this.$router.push({
                    name: 'set-inactive',
                    params: { username: this.username }
                });
            },
            go_to_add_points() {
                this.$router.push({
                    name: 'add-points',
                    params: { username: this.username }
                });
            },
            go_to_remove_points() {
                this.$router.push({
                    name: 'remove-points',
                    params: { username: this.username }
                });
            }
        },

        mounted() {
            this.username = this.$route.params.username;
        },

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
    }
    .user-id-container, .username-container, .password-container, .email-container, .user-type-container, .drivers-list-container {
        display: flex;
        width: 49%;
        border-style: solid;
        border-color: black;
        justify-content: center;
        align-items: center;
    }
    .password, .username, .email {
        width: 100%;
    }
    .catalog-item-points, .catalog-items {
        display: flex;
        flex-direction: row;
        align-items: center;
    }
</style>