

<template>
  <div id="app">
    <header>
      <h1>{{ sitename }}</h1>
      <div style="display:inline">
      <button @click="saveProductToDB">
        <font-awesome-icon icon="fa-solid fa-cart-shopping" />
        Test Save a Product to DB
      </button>
      <button @click="deleteAllCaches">
        <span class=" fas fa-save"></span>
        Delete All Caches
      </button>
      <button @click="reloadPage" class="test-elem">
        <font-awesome-icon icon="fa-solid fa-cart-shopping" />
        Reload Page
      </button>
      <strong> HTTPS Test:</strong>
      <a v-bind:href="serverURL" target="_blank">link
      </a>
      <button @click="unregisterAllServiceWorkers">
        <font-awesome-icon icon="fa-solid fa-cart-shopping" />
        Unregister All ServiceWorkers
      </button>
      <button v-if="cartEmpty" @click="toggleShowProduct">
        {{ this.cart.length }}
        <font-awesome-icon icon="fa-solid fa-cart-shopping" />
        Shopping cart
      </button>

      <div v-else>
        <button disabled class="disabled" @click="toggleShowProduct">
          {{ this.cart.length }}
          <font-awesome-icon icon="fa-solid fa-cart-shopping" />
          Shopping cart
        </button>
      </div>
      </div>
      <div class="sorty">
            Sort by:

            <button  @click="sortBySubject" class="same">Subject</button>
           
            <button @click="sortByPrice" class="same">Price</button>
            <button @click="sortByAvailability" class="same">Availability</button>
            <button margin-bottom="9em" @click="sortByLocation" class="samebreak">Location</button>

            <button @click="Ascending" class="same">Ascending</button>
            <button @click="Descending" class="same">Descending</button>
        </div>
    </header>

    <main>
      <component :is="currentView" :sortedProducts="sortedProducts" :cart="cart"
      @add-item-to-cart = "addItemToCart" @toggle-show-product = "toggleShowProduct" @remove-from-cart="removeFromCart" ></component>
      
    </main>
  </div>
</template>

<script>
import ProductList from './components/ProductList.vue'
import Checkout from './components/Checkout.vue'

export default {
  name: 'app',
  data() {
    return {
      sitename: "Coursework Website",
      showProduct: "false",
      cart: [],
      errors: [],
      serverURL: "https://webstoreapp-env.eba-azv4wsti.eu-west-2.elasticbeanstalk.com/collections/products",
      //products: products,
      products: [],
      order: {
        name: "",
        phone: "",
      },
      cartId: 0,
      sortby: "price",
      clicked: true,
      ascend: 1,
      searchQuery: "",
      searchQueryEmp: true,
      currentView: ProductList
    }
  },
  components: { ProductList, Checkout },
  
    created: function () {
      if ("serviceWorker" in navigator) {
        navigator.serviceWorker.register("service-worker.js");
      }
     let webstore = this;
     
      fetch("https://webstoreapp-env.eba-azv4wsti.eu-west-2.elasticbeanstalk.com/collections/products").then(
        function (response) {
          response.json().then(
            function (json) {
  
              webstore.products = json;
            }
          )
        }
      )
    },




  methods: {
    deleteAllCaches() {
      caches.keys().then(function (names) {
        for (let name of names)
          caches.delete(name);
      })
      console.log("All Caches Deleted");
    },
    reloadPage() {
      window.location.reload();
    },
    unregisterAllServiceWorkers() {
      navigator.serviceWorker.getRegistrations().then(function (registrations) {
        for (let registration of registrations) {
          registration.unregister()
        }
      });
      console.log("ServiceWorkers Unregistered");
    },

    addItemToCart: function (product) {
      this.cart.push(product);
      product.availableSpaces--;
    },
    

    toggleShowProduct() {
      //this.showProduct = this.showProduct ? false : true;
      if (this.currentView === ProductList){
        return this.currentView = Checkout;
      } else {
        return this.currentView = ProductList;
        }

    },
    canAddToCart(product) {
      return product.availableSpaces > 0;
    },
    cartCount(id) {
      let count = 0;
      for (let i = 0; i < this.cart.length; i++) {
        if (this.cart[i] === id) { count++; }

      }
      return count;
    },

    isClicked() {
      this.clicked = this.clicked ? false : true;
    },
    sortByPrice() {
      this.sortby = "price";
    },
    sortByAvailability() {
      this.sortby = "availability";
    },
    sortByLocation() {
      this.sortby = "location";
    },
    sortBySubject() {
      this.sortby = "subject";
    },
    Ascending() {
      this.ascend = 1;
    },
    Descending() {
      this.ascend = -1;
    },



    searchQueryEmpty: function () {
      if (this.searchQuery === '' || this.searchQuery === null || this.searchQuery === 0)
        this.searchQueryEmp = true;
      this.searchQueryEmp = false;
    },

    saveProductToDB() {
      const newProduct = {
        "id": 1022,
        "subject": 'English2',
        "location": 'Oxford',
        "price": 100,
        "spaces": 5,
        "availableSpaces": 5,
        "image": 'icon2.png',
      }

      fetch("https://webstoreapp-env.eba-azv4wsti.eu-west-2.elasticbeanstalk.com/collections/products", {
        method: "POST", //set the HTTP method as "POST"
        headers: {
          "Content-Type": "application/json", //set the data type as JSON
        },
        body: JSON.stringify(newProduct) //need to stringigy the JSON
      }).then(
        function (response) {
          response.json().then(
            function (json) {
              alert("Success: " + json.acknowledged);
              console.log("Success: " + json.acknowledged);
              webstore.products.push(newProduct);
            }
          )
        }
      );
    },

    removeFromCart: function (product) {
            for (let i = 0; i < this.cart.length; i++)
                if (this.cart[i] === product) {
                    var temp = this.cart[i];
                    this.cart[i] = this.cart[this.cart.length - 1];
                    this.cart[this.cart.length - 1] = temp;
                }
            this.cart.pop();
            product.availableSpaces++;},

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
        fetch(apiUrl + encodeURIComponent(this.order.name))
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
  },
  computed: {
    totalItemsInTheCart: function () {
      return this.cart.length || "";
    },
    resultQuery: function () {
      if (this.searchQuery) {
        return this.products.filter((product) => {
          return this.searchQuery.toLowerCase().split(' ').every(v => this.product.subject.toLowerCase().includes(v))
        })
      } else {
        return this.products;
      }
    },


    sortedProducts() {

      if (this.sortby === "price") {

        if (this.ascend === 1) {
          function comparebyprice(a, b) {
            //this.comparebyprice = function (a, b) {
            if (a.price > b.price) return 1;
            if (a.price < b.price) return -1;
            return 0;
          }
          let products = this.products.filter(product => {

            return (product.location
              .toLowerCase() + product.subject.toLowerCase())
              .indexOf(this.searchQuery.toLowerCase()) != -1

          });
          products.sort(comparebyprice);

          return products;

        }
        else {
          function comparebyprice(a, b) {
            if (a.price > b.price) return -1;
            if (a.price < b.price) return 1;
            return 0;
          }
          let products = this.products.filter(product => {

            return (product.location
              .toLowerCase() + product.subject.toLowerCase())
              .indexOf(this.searchQuery.toLowerCase()) != -1

          });
          products.sort(comparebyprice);

          return products;
        }


      }

      //////////////////
      if (this.sortby === "subject") {

        if (this.ascend === 1) {
          function comparebysubject(a, b) {
            //this.comparebyprice = function (a, b) {
            if (a.subject > b.subject) return 1;
            if (a.subject < b.subject) return -1;
            return 0;
          }
          let products = this.products.filter(product => {

            return (product.location
              .toLowerCase() + product.subject.toLowerCase())
              .indexOf(this.searchQuery.toLowerCase()) != -1

          });
          products.sort(comparebysubject);

          return products;

        }
        else {
          function comparebysubject(a, b) {
            if (a.subject > b.subject) return -1;
            if (a.subject < b.subject) return 1;
            return 0;
          }
          let products = this.products.filter(product => {

            return (product.location
              .toLowerCase() + product.subject.toLowerCase())
              .indexOf(this.searchQuery.toLowerCase()) != -1

          });
          products.sort(comparebysubject);

          return products;
        }


      }
      //////////
      if (this.sortby === "availability") {
        if (this.ascend === 1) {
          function comparebyavailability(a, b) {
            if (a.availableSpaces > b.availableSpaces) return 1;
            if (a.availableSpaces < b.availableSpaces) return -1;
            return 0;
          } return this.products.sort(comparebyavailability);
        }
        else {
          function comparebyavailability(a, b) {
            if (a.availableSpaces > b.availableSpaces) return -1;
            if (a.availableSpaces < b.availableSpaces) return 1;
            return 0;
          }
          return this.products.sort(comparebyavailability);
        }
      }

      if (this.sortby === "location") {
        if (this.ascend === 1) {
          function comparebylocation(a, b) {
            if (a.location > b.location) return 1;
            if (a.location < b.location) return -1;
            return 0;
          }
          let products = this.products.filter(product => {

            return (product.location
              .toLowerCase() + product.subject.toLowerCase())
              .indexOf(this.searchQuery.toLowerCase()) != -1

          });
          products.sort(comparebylocation);

          return products;
        }
        else {
          function comparebylocation(a, b) {
            if (a.location > b.location) return -1;
            if (a.location < b.location) return 1;
            return 0;
          }
          let products = this.products.filter(product => {

            return (product.location
              .toLowerCase() + product.subject.toLowerCase())
              .indexOf(this.searchQuery.toLowerCase()) != -1

          });
          products.sort(comparebylocation);

          return products;
        }


      }


    },

    cartEmpty: function () {
      if (this.cart.length === 0)
        return false;
      else return true;
    }
  }


}

</script>



