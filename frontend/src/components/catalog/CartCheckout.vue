<template>
    <div class="checkout">
        <NavBar :usertype="user_type" :username="username"></NavBar>
        <form style="max-width:800px;margin:auto">
            <h1>Checkout</h1>

            <!-- Section to display catalog items in the user's cart -->
            <div class="row">
                <div class="catalog-items" v-if="(items_total > 0)">
                    <ul><strong>Cart Items:</strong></ul>

                    <!-- A list item renders for each item in the cart -->
                    <li v-for="item in items" :key="item">&nbsp;{{ item }}</li>
                </div>
                <div class="catalog-items" v-else>
                    <p><strong>Cart Items: </strong>Nothing in your cart!</p>
                </div>
            </div>

            <p>Please enter the below information to complete your purchase.</p>

            <!-- Input field for First Name -->
            <div class="input-container">
                <input class="input-field" type="text" placeholder="First Name" name="first_name" v-model="first_name" required><br><br>
            </div>

            <!-- Input field for Last Name -->
            <div class="input-container">
                <input class="input-field" type="text" placeholder="Last Name" name="last_name" v-model="last_name" required><br><br>
            </div>

            <!-- Input field for Address -->
            <div class="input-container">
                <input class="input-field" type="text" placeholder="Address" name="address" v-model="address" required><br><br>
            </div>

            <!-- Input field for City -->
            <div class="input-container">
                <input class="input-field" type="text" placeholder="City" name="city" v-model="address_city" required><br><br>
            </div>

            <!-- Input field for State Abbreviation -->
            <div class="input-container">
                <input class="input-field" type="text" placeholder="State Abbreviation" name="state" v-model="address_state" required><br><br>
            </div>

            <!-- Input field for Zip Code -->
            <div class="input-container">
                <input class="input-field" type="text" placeholder="Zip Code" name="zip_code" v-model="address_zip_code" required><br><br>
            </div>

            <!-- Input field for Email -->
            <div class="input-container">
                <input class="input-field" type="text" placeholder="Email" name="email" v-model="email" required><br><br>
            </div>

            <!-- Submit button -->
            <button id="myBtn" class="btn" :disabled="!sufficient_balance" @click="submit_purchase()">
                Pay {{ this.points_total }} points
            </button>

            <br><br>

            <!-- Button to send user back to their dashboard -->
            <button id="myBtn" @click="return_to_dashboard()">
                Return to Driver Dashboard
            </button>

        </form>
    </div>

</template>

<script>
    import axios from 'axios';
    import NavBar from '@/components/misc/NavBar.vue';

    export default {

        // Component name
        name: "cart-checkout",

        // Component specific variables and data
        data() {
            return {
                status: null,
                first_name: '',
                user_id: null,
                sponsor_id: null,
                last_name:'',
                username: '',
                address: '',
                address_city: '',
                address_state: '',
                address_zip_code: '',
                email: '',
                user_type: '',
                points_balance: '',
                items: [],
                items_total: '',
                points_total: '',
                production_path: "https://www.spacebarcowboys.com",
                localhost_path: "http://localhost:5000",
                path: null,
                reason:'Purchase',
                sufficient_balance: false,
            };
        },

        // Mounted function is used for doing operations right after the component
        // Is mounted and right before the component is shown to the user
        mounted() {

            // Preventing users from accessing the application without logging in
            if (sessionStorage.getItem('loggedIn') !== 'true') this.$router.push({name: 'login'});

            // Getting username, cart items, and total cost from url
            this.username = this.$route.params.username;
            this.items = JSON.parse(this.$route.params.cart);
            this.points_total = this.$route.params.cost;

            // Getting the number of items in the cart
            this.items_total = this.items.length;

            // Setting Axios API path to either local host or production
            this.path = this.production_path;

            // Axios API call to python backend to get user information from the database
            axios.get(this.path + '/userinfo', {params: {username: this.username}})
                .then((res) => {
                    if (res.data.status === 'success') {
                        this.user_id = res.data.results[0][0];
                        this.user_type = res.data.results[0][2];
                        this.sponsor_id = res.data.results[0][7];
                        this.points_balance = res.data.results[0][11];

                        if (sessionStorage.getItem('userID') !== this.user_id.toString()) this.$router.push({name: 'login'});

                        // Determining if the user has enough points to purchase their items
                        if ( (this.points_balance < this.points_total) || (0 == this.points_total) ){
                            this.sufficient_balance = false;
                        }
                        else {
                            this.sufficient_balance = true;
                        }
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

        // Component specific methods
        methods: {

            return_to_dashboard() {
                this.$router.push({name:'driver-dashboard'})
            },

            submit_purchase() {
                if (this.items.length > 0) {

                    if ((this.first_name.trim() === '' || this.first_name === null) || 
                        (this.last_name.trim() === '' || this.last_name === null) || 
                        (this.address.trim() === '' || this.address === null) ||
                        (this.address_city.trim() === '' || this.address_city === null) ||
                        (this.address_state.trim() === '' || this.address_state === null) ||
                        (this.address_zip_code.trim() === '' || this.address_zip_code === null) ||
                        (this.email.trim() === '' || this.email === null)) {
                            window.alert("All fields must be filled out to submit a purchase");
                            return;
                    }

                    // Axios API call to python backend to add purchase information to the database
                    axios.post(this.path + '/submit-purchase', null, {params: {first_name: this.first_name, last_name: this.last_name, address: this.address, address_city: this.address_city, address_state: this.address_state, address_zip_code: this.address_zip_code, email: this.email, items: JSON.stringify(this.items), items_total: this.items_total, points_total: this.points_total, user_id: this.user_id, sponsor_id: this.sponsor_id }}) 
                        .then((res) => {
                            if (res.data.status === "success") {
                                window.alert("Purchase submitted successfully");
                            }
                            else {
                                window.alert("Cannot submit purchase.");
                            }
                        })
                        .catch((error) => {
                            // esling-disable-next-line
                            console.log(error);
                        });
                }
                else {
                    window.alert("There are no items in your cart!");
                }
            },

            // Method to get a user's point balance
            get_points_balance(){

                // Axios API call to python backend to get the user's point balance
                axios.get(this.path + '/userinfo', {params: {request: 'username'}})
                    .then(function(response){
                        this.points_balance = response.data.results[0][11];
                    });
            },
        },

        // Components used from external files
        components: { NavBar }
    }

</script>

<style scoped>
    * {
        box-sizing: border-box;
    }

    .checkout{
        /* display: flex; */
        flex-direction: column;
        width: 98.5vw;
        height: 97.5vh;
        border-style: solid;
        border-color: black;
        gap: 1rem;
        background-color: #0096c7;
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

    .btn:disabled{
        background-color: #acacaf;
        color: white;
        padding: 15px 20px;
        border: none;
        cursor: pointer;
        width: 100%;
        opacity: 0.9;
    }

    .catalog-items ul {
        margin-left: -2.5rem;
    }
</style>