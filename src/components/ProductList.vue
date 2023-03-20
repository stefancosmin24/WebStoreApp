<template>
    <div>
        <h1>Product List</h1>

        <div>
            <div v-for="product in sortedProducts" class="box">
                <figure>
                    <img v-bind:src="product.image" style="float:right" alt="" width="60px" height="60px">
                </figure>

                <p>Subject: {{ product.subject }}</p>
                <p>Location: {{ product.location }}</p>
                <p>Price: £{{ product.price }}</p>
                <p>Spaces: {{ product.availableSpaces }}</p>

                <button style=" width: 120px;" v-if="canAddToCart(product)" v-on:click="addItemToCart(product)">Add to the
                    Cart</button>
                <div v-else>
                    <button disabled class="disabled">Add to the Cart</button>
                    <p class="same2">No more available stock</p>
                </div>
            </div>
        </div>

    </div>
</template>

<script>
export default {
    name: "ProductList",
    props: ["sortedProducts", "imagesBaseURL", "cart"],
    data() {
        return{
        }

    },
    methods: {
        addItemToCart: function (product) {
           this.$emit("add-item-to-cart", product);
        },
        canAddToCart(product) {
            return product.availableSpaces > 0;
        },
    }
}
</script>