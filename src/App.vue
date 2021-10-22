<template>
  <div id="app" >
      <div id="navbar">
        <b-navbar toggleable="lg" type="dark">
            <b-navbar-brand style="font-weight:bold">ShoeShop</b-navbar-brand>
            <b-collapse id="nav-collapse" is-nav>
            <b-navbar-nav class="ml-auto">
                <transition name="fade">
                  <div v-show="alert" class="alert">New Product Added</div>
                </transition>
                <b-nav-item-dropdown right role="">
                  <template #button-content>
                      <span class="fa fa-shopping-cart fa-2x" style="color:white"></span>
                      <b-badge class="mx-3" size="lg" variant="light">{{allItem}}</b-badge>
                  </template>
                  <b-dropdown-form @click.prevent="" v-for="(item,index) in purchased" :key="index" class="d-flex align-items-center">
                    <div class="row">
                      <div class="col"><img :src="item.image"></div>
                      <div class="col"><small>Size</small>{{item.size}}</div>
                      <div class="col col-quant">
                        <span role="button" class="fa fa-minus mx-3" aria-hidden="true" @click="item.quantity--,item.quantity<=1?item.quantity=1:''"></span>
                        {{item.quantity}}
                        <span role="button" class="fa fa-plus mx-3" aria-hidden="true" @click="item.quantity++"></span>
                      </div>
                      <div class="col col-price">
                        ${{item.price*item.quantity}}
                      </div>
                      <div class="col-delete">
                        <span role="button" class="fa fa-times fa-lg" @click="deleteItem(index)"></span>
                      </div>
                    </div>
                  </b-dropdown-form>
                  <div class="total">
                    Total: ${{totalPrice}}
                  </div>
                </b-nav-item-dropdown>
            </b-navbar-nav>
            </b-collapse>
        </b-navbar>
    </div>
    <ProductCard  v-for="(product,index) in products" :key="index" :product="product" @cart="addToCart"/>
  </div>
</template>

<script>
import ProductCard from './components/ProductCard.vue'
import {items} from "./assets/products.js"

export default {
  name: 'App',

  components: {
    ProductCard,
  },

  data(){
    return{
      items:items, //ürünleri dosyadan alıyoruz
      products:[], //ürünleri transition için sırayla bu değişkene atıyoruz
      purchased:[], //sepete atılmış ürünler
      alert:false, //sepete yeni bir ürün eklendiğini belirtiyoruz
    }
  },

  mounted(){
    this.loadProductCards() //ürün kartlarını sırayla yüklemek için
  },

  computed:{

    //sepetteki toplam ürün sayısı hesaplanıyor
    allItem(){
      let counter=0
      this.purchased.forEach(item => {
        counter+=item.quantity
      });
      return counter
    },

    //spetteki toplam fiyat bulunuyor
    totalPrice(){
      if(this.purchased.length>0){
        return this.purchased.map(item=>item.price*item.quantity).reduce((a,b)=>a+b)
      }else{
        return 0
      }
    }

  },

  methods:{

    //child componentlerden gelen ürünler sepete ekleniyor
    addToCart(e){
      let isFound=null
      isFound = this.purchased.find(item=> item.id==e.id && item.color==e.color && item.size==e.size)
      if(!isFound){
        this.purchased.push(e)
      }else{
        isFound.quantity+=e.quantity
      }
      this.alert=true
      setTimeout(() => {
        this.alert=false
      }, 2000);
    },

    //sepetteki ürünleri tek tek silmek için
    deleteItem(index){
      this.purchased.splice(index,1)
    },

    //ürün kartlarını sırayla yüklemek için
    loadProductCards(){
      let r=0
      let intrvl = setInterval(() => {
        this.products.push(this.items[r])
        r++
        if(r>=this.items.length){
          clearInterval(intrvl)
        }
      }, 250);
    }

  },
}
</script>

<style scoped>
  #app {
    height: 100%;
    width: 100%;
    min-height: 100vh;
    background-color: rgb(213, 213, 213);
    display: flex;
    flex-direction: row;
    align-items: flex-start;
    flex-wrap: wrap;
    padding-top: 70px;
    padding-inline: 75px;
    font-family: 'Mako', sans-serif;
    justify-content: space-between;
  }
  #navbar{
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      background-color: rgb(86, 117, 196);
      z-index: 2;
      position: fixed;
      box-shadow: 0 10px 7px -2px #ababab
  }
  #button-content{
    display: flex;
    align-items: center;
  }
  .badge{
    background-color: rgb(86, 117, 196);
    color: white;
    font-size: 17px;
    font-weight: 600;
  }
  .row{
    width: 350px;
    height: 50px;
    padding:0;
    ;
  }
  .row img{
    height: 100%;
    width: 100%;
    object-fit: cover;
  }
  .col{
    height: 100%;
    width: 50px;
    overflow: hidden;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    font-size: 12px;
    color:rgb(86, 117, 196);
  }
  .col-quant{
    font-size:15px;
    font-weight:bold;
    flex-direction:row;
  }
  .col-price{
    font-size:15px;
    font-weight:600;
    flex-direction:row;
  }
  .col-delete{
    height: 100%;
    width: 20px;
    overflow: hidden;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    font-size: 12px;
    color:rgb(86, 117, 196);
  }
  .total{
    font-size:14px;
    font-weight:600;
    display: flex;
    flex-direction:row;
    justify-content: flex-end;
    margin-top: 10px;
    margin-right:55px ;
    color:rgb(86, 117, 196);
  }
  .alert{
    height: 20px;
    width: fit-content;
    font-size: 12px;
    display: flex;
    align-items: center;
    justify-content: center;
    color: white;
    background-color: rgb(83, 150, 99);
    position: absolute;
    top: 65px;
    right: 0px;
    border-radius: 0;
    z-index: -2;
  }

  .fade-enter-active {
    animation: toDown .3s ease-out;
  }
  .fade-leave-to{
    animation: toUp .3s ease-in;
  }
  @keyframes toDown {
    from{
      transform: translateY(-100%);
      opacity: 0;
    }
    to{
      transform: translateY(0);
      opacity: 1;
    }
  }
  @keyframes toUp {
    from{
      transform: translateY(0);
      opacity: 1;
    }
    to{
      transform: translateY(-100%);
      opacity: 0;
    }
  }

</style>
