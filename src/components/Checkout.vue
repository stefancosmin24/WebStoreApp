<template>
    <div>
        <h1>Shopping Cart</h1>
        <div v-for="product in cart" class="box">
            <figure>
                <img v-bind:src="product.image" style="float:right" alt="" width="60px" height="60px">
            </figure>

            <p>Subject: {{ product.subject }}</p>
            <p>Location: {{ product.location }}</p>
            <p>Price: £{{ product.price }}</p>

            <button v-on:click="removeFromCart(product)">Remove</button>

        </div>
        <h2>Welcome to the Checkout Page</h2>
        <h2>
            <form @submit="checkForm" method="post">
                <strong>Name: </strong>
                <input type="text" name="name" id="name" v-model="order.name">
                <strong>Phone: </strong>
                <input type="tel" name="phone" v-model="order.phone">
                <input type="submit" value="Checkout">
                <p v-if="errors.length">
                    <b>Please correct the following error:</b>
                <ul>
                    <li v-for="error in errors">{{ error }}</li>
                </ul>
                </p>
            </form>
        </h2>
    </div>
</template>

<script>
export default {
    name: "Checkout",
    props: ["sortedProducts", "imagesBaseURL", "cart"],
    data() {
        return {
            order: {
                name: "",
                phone: "",
            },
            cartId: 0,
            errors: [],
            apiUrl : 'https://vuecookbook.netlify.app/.netlify/functions/product-name?name='
        }

    },
    methods: {
        removeFromCart: function (product) {
            this.$emit("remove-from-cart",product);
            if (this.cart.length === 0)
                {this.showProduct = this.showProduct ? false : true;
                    this.$emit("toggle-show-product");
                }
        },

        checkForm: function (e) {
            e.preventDefault();
            this.errors = [];
            var nameRegex = /^[a-z ,.'-]+$/i;
            var phoneRegex = /^[+]*[(]{0,1}[0-9]{1,4}[)]{0,1}[-\s\./0-9]*$/g;
            if (!(nameRegex.test(this.order.name))) {
                this.errors.push("Correct name is required.");
            }
            else if (!(phoneRegex.test(this.order.phone))) {
                this.errors.push("Correct phone number is required.");
            }
            else {
                fetch(this.apiUrl + encodeURIComponent(this.order.name))
                    .then(async res => {
                        if (res.status === 204) {
                            alert('Order has been submitted!')
                            //save order
                            const newOrder = {
                                "name": this.order.name,
                                "phoneNumber": this.order.phone,
                                "lessonID": this.cart,
                                "spaces": this.cart.length
                            }

                            fetch("https://webstoreapp-env.eba-azv4wsti.eu-west-2.elasticbeanstalk.com/collections/orders", {
                                method: "POST", //set the HTTP method as "POST"
                                headers: {
                                    "Content-Type": "application/json", //set the data type as JSON
                                },
                                body: JSON.stringify(newOrder) //need to stringigy the JSON
                            }).then(
                                function (response) {
                                    response.json().then(
                                        function (json) {
                                            //alert("Success: " + json.acknowledged);
                                            //console.log("Success: " + json.acknowledged);
                                            //webstore.products.push(newProduct);
                                        }
                                    )
                                }
                            );

                            //update spaces
                            //let item = this.cart[1];
                            //alert(item);
                            for (let i = 0; i < this.cart.length; i++) {
                                let item = this.cart[i];
                                //item.availableSpaces--;
                                const newItem = {

                                    "id": item.id,
                                    "subject": item.subject,
                                    "location": item.location,
                                    "price": item.price,
                                    "spaces": item.spaces,
                                    "availableSpaces": item.availableSpaces,
                                    "image": item.image,
                                }

                                fetch("https://webstoreapp-env.eba-azv4wsti.eu-west-2.elasticbeanstalk.com/collections/products/" + item._id, {
                                    method: "PUT", //set the HTTP method as "POST"
                                    headers: {
                                        "Content-Type": "application/json", //set the data type as JSON
                                    },
                                    body: JSON.stringify(newItem) //need to stringigy the JSON
                                }).then(
                                    function (response) {
                                        response.json().then(
                                            function (json) {
                                                alert("Success: " + json.acknowledged);
                                                console.log("Success: " + json.acknowledged);
                                                //webstore.products.push(newProduct);
                                            }
                                        )
                                    }
                                );
                            }


                        } else if (res.status === 400) {
                            let errorResponse = await res.json();
                            this.errors.push(errorResponse.error);
                        }
                    });
            }
        }
    }
}
</script>