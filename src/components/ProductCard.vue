<template>
    <transition name="listComp">
    <div id="card" class="card">
        <b-container class="w-100 h-100 bv-example-row">
            <b-row class="m-0">
                <b-col cols="7">
                    <transition name="slideRight" mode="out-in">
                        <img :src="product.images[selectedcolor]" :key="selectedcolor">
                    </transition>
                </b-col>
                <transition name="slideLeft" mode="out-in">
                    <b-col cols="5" class="pt-4 pr-2" :key="selectedcolor">
                        <p class="head1 mb-0" :class="selectedcolor">{{product.category}}</p>
                        <p class="head2 mb-2">{{product.title}}</p>
                        <div style="position:relative">
                            <span class="price" :class="[discounted?'discount':'', selectedcolor]">${{product.price}}</span>
                            <span v-if="discounted" class="price newprice ml-4" :class="[selectedcolor]">${{calcPrice}}</span>        
                        </div>
                        <p class="description mt-2">{{product.summary}}</p>
                    </b-col>
                </transition>
            </b-row>
            <b-row align-v="center" class="d-flex flex-row align-content-center ">

                <b-col cols="3" class="d-flex flex-row align-items-center m-0">
                    <span v-for="(s1,i) in product.rating" :key="'1'+i" class="fa fa-star checked" :class="selectedcolor"></span>
                    <span v-for="(s2,j) in (5-product.rating)" :key="'2'+j" class="fa fa-star-o" :class="selectedcolor"></span>
                </b-col>

                <b-col v-if="stars" cols="4" class="d-flex flex-row align-items-center m-0">
                    <b-form-group class="d-flex flex-row align-items-center m-0">
                        <span role="button" v-for="(color,index) in product.colors" :key="index" class="fa fa-circle fa-xs my-0 mx-2 p-0 align-items-center" :class="[color]" @click="colorState?[selectedcolor=color,loadstars]:''"></span>
                    </b-form-group>
                </b-col>

            </b-row>
            <b-row class="m-0">
                <b-col cols="7" class="m-0 p-0 d-flex align-items-center">
                    <p class="radiogroup" :class="selectedcolor">CHOOSE SIZE</p>
                </b-col>
                <b-col cols="5" class="d-flex align-items-center justify-content-center w-100">
                    <span role="button" class="fa fa-minus mx-3" :class="selectedcolor" aria-hidden="true" @click="quantity--,quantity<=1?quantity=1:''"></span>
                    <b-badge variant="light" size="lg" :class="selectedcolor" style="font-size:18px; background-color:#F4F4F4">{{quantity}}</b-badge>
                    <span role="button" class="fa fa-plus mx-3" :class="selectedcolor" aria-hidden="true" @click="quantity++"></span>
                </b-col>
            </b-row>
            <b-row>
                <b-col cols="7">
                    <b-form-group v-slot="{ ariaDescribedby }" class="d-flex align-items-center">
                        <b-form-radio 
                         ref="sizeRadios"
                         button
                         button-variant="none" 
                         class="r-button" 
                         :class="[selectedcolor, selectedsize==size?bgColor:'']" 
                         v-for="(size,index) in product.sizes" 
                         :key="index" 
                         v-model="selectedsize" 
                         :aria-describedby="ariaDescribedby" 
                         name="size-radios" 
                         :value="size"
                        >
                            <p class="size-radio" :class="[selectedsize==size?bgColor:selectedcolor]">{{size}}</p>
                        </b-form-radio>
                    </b-form-group>                
                </b-col>
                <b-col cols="5">
                    <b-button class="w-100 m-0 addbutton" :class="bgColor" @click="addToCart">Add To Cart</b-button>
                </b-col>
            </b-row>
            <transition name="fade">
                <div v-show="warn" class="warn" :class="selectedcolor">Choose a Size</div>
            </transition>
        </b-container>
    </div>
    </transition>
</template>

<script>
export default {

    props:["product"],

    data() {
      return {
        selectedsize:null, //seçilen size
        selectedcolor:this.product.colors[0], //seçilen renk
        quantity:1, //adet
        price:null, //sepete atılan price değeri
        warn:false, //size seçimini unutmamak için uyarı
        rating:null, //yıldız yüklemek için kullanılan bir değişken
        stars:false, //yıldızları renk değiştiğinde tekrar yüklemek için
        colorState:true //yıldızlar dolmadan yeni renk seçimini engellemek için
      }
    },

    mounted(){
        //başlangıçta yıldızları yüklemek için
        this.rating=this.product.rating
        this.loadstars()
    },
    
    computed:{

        //butonlar için ayakkabı renngine göre arka plan class'ı belirleme
        bgColor(){
            return `bg-${this.selectedcolor}`
        },

        //indirimli fiyat hesaplama
        calcPrice(){
            if(this.selectedsize==this.product.sizes[0]||this.selectedsize==this.product.sizes[this.product.sizes.length-1]){
                return this.product.price*0.8
            }else if(this.selectedsize==this.product.sizes[1]||this.selectedsize==this.product.sizes[this.product.sizes.length-2]){
                return this.product.price*0.9
            }else{
                return this.product.price
            }
        },

        //farklı numaralarda indirim olup olmadığı kontrol ediliyor
        discounted(){
            return this.calcPrice<this.product.price
        }

    },

    watch:{

        //renk değişimi takip ediliyor
        selectedcolor(){                            
            this.colorState=false
            this.rating=this.product.rating
            this.loadstars()
        }
    },

    methods:{

        //ürünlerin sepete atılması
        addToCart(){
            if(this.selectedsize==null){
                this.warn=true
                setTimeout(() => {
                    this.warn=false
                }, 1000);                         //size seçilmediyse 1 saniyelik uyarı veriyoruz 
            }else{
                let obj={
                    id:this.product.id,
                    color:this.selectedcolor,
                    size:this.selectedsize,
                    quantity:this.quantity,
                    price:this.calcPrice,
                    image:this.product.images[this.selectedcolor]
                }
                this.$emit("cart",obj)
            }

        },

        //yıldızları (rating) yükleme fonksiyonu
        loadstars(){
            this.stars=false //state değiştirerek tekrardan yüklenmesini sağlıyoruz çünkü renge göre farklı rating vermedim
            this.stars=true
            const lim = this.rating
            let rat=0
            let intrvl = setInterval(() => {
                this.product.rating=rat
                rat++
                if (rat>lim){
                    clearInterval(intrvl)
                    this.rating=null
                    this.colorState=true
                }
            }, 400/lim);
        }

    }

}
</script>

<style scoped>
    #card{
        height: 380px;
        width: 550px;
        border-radius: 10px;
        box-shadow: 10px 10px 10px -2px #ababab, -2px -2px 10px -2px #ababab;
        background-color: #F4F4F4;
        margin-inline: 60px;
        margin-block: 40px;
        overflow: hidden;
        position: relative;
    }
    .card:nth-child(odd){
        animation: left .5s ease-out;
    }
    .card:nth-child(even){
        animation: right .5s ease-out;
    }
    img{
        width: 100%;
    }
    .head1{
        font-size: 13px;
        text-transform: uppercase;
    }
    .head2{
        font-size: 20px;
        font-weight: 600;
        background-color: #F4F4F4;
        text-transform: uppercase;
    }
    .price{
        font-size: 25px;
        margin-block: 10px;
        font-weight: 700;
        color: #CCA9A8;
    }
    .newprice{
        position:absolute;
        top: -25%;
        left: 25%;
        animation: right .2s linear;
    }
    .description{
        font-size: 11px;
        font-weight: 600;
    }
    .sizes{
        width: 100%;
        height: 100%;
    }
    .radiogroup{
        font-size: 10px;
        font-weight: 600;
        color: #CCA9A8;
        margin: 0;
    }
    .r-button{
        background-color: none;
        border: 1px solid #CCA9A8;
        margin-right: 3px;
        color: #CCA9A8;
        padding: 0;
        display: flex;
        align-items: center;
        justify-content: center;
        text-align: center;
        outline: none;
    }
    .size-radio{
        padding: 0;
        margin: 0;
        font-size: 13px;
        color: #CCA9A8;
    }
    .color-r-btn{
        margin: 0;
        padding: 0;
        display: flex;
        align-items: center;
        justify-content: center;
        height: 20px;
    }
    .addbutton{
        background-color: #F4F4F4;
        border: 1px solid;
    }
    .pink{
        color: #CCA9A8;
        border-color: #CCA9A8;
    }
    .mint{
        color: #98C8CA;
        border-color: #98C8CA;
    }
    .blue{
        color: #4EA8CA;
        border-color: #4EA8CA;
    }
    .grey{
        color: grey;
        border-color: grey;
    }
    .green{
        color: #3bbf8e;
        border-color: #3bbf8e;
    }
    .red{
        color: #e02023;
        border-color: #e02023;
    }
    .orange{
        color:#9C7D22;
        border-color: #9C7D22;
    }
    .aqua{
        color:#32c3ca;
        border-color: #32c3ca;
    }
    .smoke{
        color: #7E7B76;
        border-color: #7E7B76;
    }
    .pblue{
        color: #1C72C9;
        border-color: #1C72C9;
    }
    .burgundy{
        color: #714B53;
        border-color: #714B53;
    }
    .purple{
        color: #4B4CA9;
        border-color: #4B4CA9;
    }
    .bg-pink{
        background-color: #CCA9A8;
        color: #F4F4F4;
    }
    .bg-mint{
        background-color: #98C8CA;
        color: #F4F4F4;
    }
    .bg-blue{
        background-color: #4EA8CA;
        color: #F4F4F4;
    }
    .bg-grey{
        background-color: grey;
        color: #F4F4F4;
    }
    .bg-green{
        background-color: #3bbf8e;
        color: #F4F4F4;
    }
    .bg-red{
        background-color: #e02023;
        color: #F4F4F4;
    }
    .bg-orange{
        background-color: #9C7D22;
        color: #F4F4F4;
    }
    .bg-aqua{
        background-color: #32c3ca;
        color: #F4F4F4;
    }
    .bg-smoke{
        background-color: #7E7B76;
        color: #F4F4F4;
    }
    .bg-pblue{
        background-color: #1C72C9;
        color: #F4F4F4;
    }
    .bg-purple{
        background-color: #4B4CA9;
        color: #F4F4F4;
    }
    .bg-burgundy{
        background-color: #714B53;
        color: #F4F4F4;
    }
    .warn{
        height: 80px;
        width: 550px;
        border-radius: 0 0 10px 10px;
        font-size: 20px;
        font-weight: 600;
        display: flex;
        align-items: center;
        justify-content: center;
        position: absolute;
        top: 300px;
        left: 0;
        background-color:#F4F4F4;;
    }
    .discount{
        position: relative;
        height: 100%;
        width: 100%;
    }
    .discount::before{
        content: "";
        position: absolute;
        left: 0;
        top: 0;
        height: 85%;
        width: 110%;
        border-bottom: 4px solid rgb(57, 57, 57);
        transform-origin: 0 70%;
        animation: draw .2s linear;
        animation-fill-mode: forwards;
    }

    .slideRight-enter-active {
        animation: toRight .3s ease-in-out;
    }
    @keyframes toRight {
        from{
        transform: translateX(-50%);
        opacity: 0;
        }
        to{
        transform: translateY(0);
        opacity: 1;
        }
    }
    .slideLeft-enter-active {
        animation: toLeft .3s ease-in-out;
    }
    @keyframes toLeft {
        from{
        transform: translateX(50%);
        opacity: 0;
        }
        to{
        transform: translateY(0);
        opacity: 1;
        }
    }

    .fade-enter-active {
        animation: toUp .15s ease-out;
    }
    .fade-leave-to{
        animation: toDown .15s ease-in;
    }
    @keyframes toUp {
        from{
        transform: translateY(100%);
        opacity: 0;
        }
        to{
        transform: translateY(0);
        opacity: 1;
        }
    }
    @keyframes toDown {
        from{
        transform: translateY(0);
        opacity: 1;
        }
        to{
        transform: translateY(100%);
        opacity: 0;
        }
    }

    .listComp-enter-active {
        animation: occour 3s ease-in-out;
    }
    @keyframes occour {
        from{
        opacity: 0;
        }
        to{
        opacity: 1;
        }
    }

    @keyframes draw {
        0% {
            width: 0;
            transform: rotateZ(-15deg) scaleX(0);

        }
        100% {
            width: 100%;
            transform:rotateZ(-15deg) scaleX(1);
        }
    }

    @keyframes right {
        0% {
            opacity:0;
            transform: translateX(-150%);

        }
        100% {
            opaticty:1;
            transform:translateX(0);
        }
    }
    @keyframes left {
        0% {
            opacity:0;
            transform: translateX(150%);

        }
        100% {
            opaticty:1;
            transform:translateX(0);
        }
    }
</style>