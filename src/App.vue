<template>
  <div id="calculator-mcu">
    <!-- Первая полоса -->
    <div class="head-container">
    <!-- Левая часть с типом лицензии -->
      <div class="configuration-part">
      
        <span class="configuration-title">Тип лицензии:</span>
        <select v-model="currentLicense">
          <option v-for="lic in licenses" :value="lic" :key="lic.id">
            {{ lic.name }}
          </option>
        </select>

        <span>Количество портов:</span>
        <div class="calculator-input__container" @click="clickBlockFeature('changePort')">
          <input type="number" v-bind="portNumber" @change="validateInput" @blur="portNumber = portNumber === ' ' ? 1" :disabled="currentLicense.id == 'free'" >
          <p>{{this.portNumber}}</p>
          <div class="calculator-counter__container" v-if="currentLicense.id != 'free'">
            <button class="calculator-counter plus" v-if="portNumber < currentLicense.ports.max" @click="portNumber++">+</button>
            <button class="calculator-counter minus" v-if="portNumber > currentLicense.ports.min" @click="portNumber--">-</button>
          </div>
        </div>
      </div>

      <!-- Цена или бесплатно -->
      <div class="calculator-block__cost">
        <p class="price-port calculator-price"> 
          <span>{{ currentLicense.id === 'full' ? portPrice.toLocaleString("ru-RU") + "₽" : 'Бесплатно' }}</span>
        </p>
      </div>
    </div>
    
    <div class="calculator-body__container">
      <div class="main-info">
        <div class="calculator-info__container">
          <div class="calculator-subtitle__container">
            <p class="calculator-body__subtitle"><b>Стандартные возможности</b></p>
            <p class="calculator-body__subinfo calculator-block__cost">Включено</p>
          </div>

          <ul>
            <li class="calculator-standart__item" v-for="(option, id) in currentLicense.defOps" :key="id">
              <div class="calculator-item__icon standart-icon"></div>
              <p v-html="option.title"></p>
            </li>
          </ul>
        </div>

        <div class="calculator-info__container" v-if="currentLicense.restrictions.length != 0">
          <div class="calculator-subtitle__container">
            <p class="calculator-body__subtitle"><b>Ограничения</b></p>
          </div>
          <ul>
            <li class="calculator-standart__item" v-for="(item, id) in currentLicense.restrictions" :key="id" @click="clickBlockFeature('restriction')">
              <div class="calculator-item__icon restriction-icon"></div>
              <p v-html="item.title"></p>
            </li>
            <li class="calculator-switcher__item"><button class="calculator-link switcher" v-if="currentLicense.id === 'free'" @click="changeToFull">Переключить на полную лицензию →</button></li>
          </ul>
        </div>

        <div class="additional-features" v-if="selectedAddOps.length != 0">
          <div class="calculator-subtitle__container">
            <p class="calculator-body__subtitle"><b>Дополнительные возможности</b></p>
          </div>
          <ul>
            <li class="calculator-subtitle__container" v-for="(option, id) in selectedAddOps" :key="id">
              <div class="calculator-body__subtitle calculator-block__content calculator-standart__item" @click="removeCurrentAddOps(option,id)">
                <div class="calculator-item__icon plus-icon"></div>
                <p>{{ option.title }}</p>
              </div> 
              <p class="calculator-body__subinfo calculator-block__cost">{{ option.price.toLocaleString("ru-RU") + "₽" }}</p>
            </li>
          </ul>

        </div>

      </div>
    </div>

    <div class="footer-line">
      <div class="calculator-block__content bottom-text">
        <p>Общая стоимость: </p>
      </div>
      <div class="calculator-price calculator-block__cost">
        <p><b>{{ currentLicense.id === 'full' ? totalPrice.toLocaleString("ru-RU") + "₽" : 'Бесплатно' }}</b></p>
      </div>
    </div>

    <div v-if="addOpsList.length">
      <h2>Добавьте дополнительные опции</h2>
      <div class="head-container"></div>
      <div class="calculator-body__container">
        <div class="main-info">
          <ul>
            <li class="calculator-subtitle__container" v-for="(option, id) in addOpsList" :key="id">
              <div class="calculator-body__subtitle calculator-block__content calculator-standart__item" @click="addCurrentAddOps(option,id)">
                <img class="calculator-item__icon" src="./assets/plus.svg" alt="plus">
                <p>{{ option.title }}</p>
              </div>
              <p class="calculator-body__subinfo calculator-block__cost">{{ option.price.toLocaleString("ru-RU") + "₽"}}</p>
            </li>
          </ul>
        </div>
      </div>
    </div>
    <button class="cart btn-large light-blue darken-1 waves-effect white_text btn_cart span intro__btn" @click="showCart()">
      <span>{{ currentLicense.id != 'free' ? 'Оформить заказ' : 'Скачать' }}</span>
    </button>

    <!-- Popup -->
    <div class="popup-overlay" v-if="popupFlag" @click="popupFlag = false">
      <div class="calculator-popup">
        <div class="exit-button" @click="popupFlag = false">
          ×
        </div>
        <div class="calculator-popup__container">
          <div class="calculator-popup__container">
            <p class="h2">
              Переключиться на полную лицензии
            </p>
            <p>{{ popupMsg }}</p>
          </div>
          <div class="calculator-button__container">
            <button class="modal-action modal-close waves-effect waves-light btn-flat lighten-5 sprt-btn-calc" @click="changeToFull"><b>Переключиться на полную лицензию</b></button>
          </div>
        </div>
      </div>
    </div>

  </div>
</template>

<script>
import { licenses, addOps } from './data'

const PORT_PRICE = 40000

export default {
  name: 'App',
  data: () => ({
    portNumber: 0,
    mail: '',
    currentLicense: licenses[1],
    addOpsList: [],
    selectedAddOps: [],
    popupFlag: false,
    msg:{
      default: 'Данная возможность недоступна в бесплатной лицензии... ',
      restriction: 'Данные ограничения нельзя убрать для бесплатной версии...',
      changePort: 'В бесплатной лицензии нельзя изменять количество портов'
    },
    popupMsg: ''
  }),
  methods: {
    changeToFull () {
      const prevLic = this.currentLicense;
      const fullLic = licenses.find(item => {
        if (item.id === 'full') return true;
      })
      if(prevLic == fullLic){
        return true
      }else{
        this.currentLicense = fullLic;
        this.popupFlag = this.popupFlag ?? !this.popupFlag;
      }
      // this.currentLicense = findFull ? findFull : prevLic;
    },
    showCart() {
      console.log(this.cart);
    },
    addCurrentAddOps(addOps,id){
      if(this.currentLicense.id == 'free'){
        this.clickBlockFeature();
      }else{
        this.addOpsList.splice(id);
        this.selectedAddOps.push(addOps);
      } 
    },
    removeCurrentAddOps(addOps,id){
      this.selectedAddOps.splice(id);
      this.addOpsList.push(addOps);
    },
    test(testVar){
      console.log("Переключаем переменную" + testVar);
    },
    clickBlockFeature(errorMsg){
      if(this.currentLicense.id == 'free'){
        this.popupMsg = errorMsg ? this.msg[errorMsg] : this.msg['default'];
        this.popupFlag = !this.popupFlag ?? !this.popupFlag;
      }
    },
    validateInput(e){
      // const number = e.target.value

      // console.log('test', number)
      // // if(number > this.currentLicense.ports.max){
      // //   console.log("max "+ number);
      // //   this.portNumber = this.currentLicense.ports.max;
      // // }else if (number < this.currentLicense.ports.min){
      // //   console.log("else "+ number);
      // //   this.portNumber = this.currentLicense.ports.min;
      // // }
      // // if (number >= this.currentLicense.ports.min && number < this.currentLicense.ports.max) {
      // //   console.log(123);
      // // }
      // console.log(number / this.currentLicense.ports.max);
      // if ( number / this.currentLicense.ports.max > 1 ) {
      //   return this.portNumber = this.currentLicense.ports.max
      // }
      // if ( number / this.currentLicense.ports.max < 0) {
      //   return this.portNumber = this.currentLicense.ports.min
      // }
      // this.portNumber = number

      
    }
  },
  computed: {
    licenses: () => licenses,
    portPrice: ({portNumber}) => PORT_PRICE * portNumber,
    totalPrice: ({portPrice, currentLicense, selectedAddOps}) => {
      if(currentLicense.id === 'free'){
        return 'Бесплатно';
      }else{
        if(selectedAddOps){
          let selectedAddOpsPrice = 0;
          selectedAddOps.forEach(element => {
            selectedAddOpsPrice += element.price;
          })
          return portPrice + selectedAddOpsPrice;
        }else{
          return portPrice;
        }
      }
    },
    cart: ({currentLicense , portNumber, selectedAddOps, totalPrice}) => {
      let str = "Тип лицензии:" + currentLicense.name + " Количество портов:" + portNumber + "Полная стоимость: " + totalPrice;
      if(selectedAddOps.length){
        str += " Дополнительные возможности:";
        selectedAddOps.forEach(element => {
          str += element.title;
        });
      }
      return str;
    },
  },
  watch: {
    currentLicense (v) {
      this.portNumber = v.ports.min
      if(this.currentLicense.id === 'full'){
        this.addCurrentAddOps(this.addOpsList[0]);
      }else{
        this.selectedAddOps.forEach(element => {
          element.id == 0 ? this.removeCurrentAddOps(element) : '';
        });
      }
    },
    portNumber(v){
      console.log('value', isFinite(parseInt(v)));
      const number = v


      // console.log(number / this.currentLicense.ports.max);
      if ( number / this.currentLicense.ports.max > 1 ) {
        return this.portNumber = this.currentLicense.ports.max
      }
      if ( number / this.currentLicense.ports.max < 0) {
        return this.portNumber = this.currentLicense.ports.min
      }
      this.portNumber = number
      console.log(typeof number, number);
      if (isNaN(number)) {
        return this.portNumber = 13
      }
    }
  },
  created () {
    this.currentLicense = licenses[1]
    this.portNumber = 10;
    this.selectedAddOps = addOps;
    this.popupMsg = this.msg.default;
  }
};
</script>

<style lang="sass">
$menu-color: #edf2f6

ul,p
  margin-bottom: 0

button
  background: none
  color:#0097a7

.bold
  font-weight: 700

#calculator-mcu
  font-size: 14px
  color: #555

.head-container,.footer-line
  display: flex
  justify-content: space-between
  align-items: center
  padding: 0 15px
  background-color: $menu-color
  min-height: 40px
  font-size: 16px

.calculator-counter__container
  display: flex

.calculator-input__container
  display: flex

.main-info
  padding: 10px 15px
  position: relative
  font-size: 16px
  &::after
    content: ""
    position: absolute
    border-right: 1px solid $menu-color
    height: 100%
    top: 0
    right: 10%

.calculator-standart__item
  display: flex
  align-items: center
  padding-left: 0

.calculator-item__icon
  max-width: 20px
  width: 100%
  height: 30px
  margin-right: 10px
  background-repeat: no-repeat
  background-position: center

.standart-icon
  background-image: url("./assets/check.svg")

.restriction-icon
  background-image: url("./assets/restriction.svg")

.plus-icon
  background-image: url("./assets/plus.svg")
  transition: 0.2s

.additional-features
  &:hover .plus-icon
    background-image: url("./assets/plus-cropped.svg")
    transform: rotate(45deg)
    transition: 0.2s

.calculator-switcher__item
  padding-left: 45px
  
// .calculator-link.switcher
//   margin-top: 5px

.calculator-subtitle__container
  display: flex
  justify-content: space-between
  align-items: center
  padding-left: 0px

.calculator-block__content
  max-width: 91%
  width: 100%
  cursor: pointer
  &:hover
    text-decoration: underline

.calculator-block__cost
  max-width:8%
  width: 100%
  text-align: center

.calculator-body__subinfo
  text-align: center

.calculator-info__container
  margin: 0 0 5px 0

.input-container,.configuration-part
  display: flex
  align-items: center
  position: relative

.calculator-button__container
  display: flex
  justify-content: flex-end
  margin-top: 20px

.configuration-part
  & select
      display: block!important
      width: 140px
      height: 35px
  & input[type=number]
      width:35px
      height: 30px
      margin: 0
      border: 1px solid #d1d1d1
      background: #FFF

.calculator-body
  padding-bottom: 15px
  &__container
    border:
      left: 1px solid $menu-color
      right: 1px solid $menu-color
      bottom: 1px solid $menu-color
    li
      margin: 10px 0
      background: none
    li.standart
      cursor: default
  &__title
    font-weight: bold
    height: 50px
    line-height: 50px

.configuration-part *
  margin-right: 5px

.bottom-text
  text-align: right

button.cart
  margin: 20px auto
  display: block

.calculator-price
  font-size: 16px
  font-weight: bold

.popup-overlay
  position: absolute
  top: 0
  left: 0
  background: rgba(0,0,0,0.1)
  width: 100%
  height: 100%
  z-index: 99

.exit-button
  position: absolute
  right: 20px
  top: 10px
  font-size: 30px
  cursor: pointer

.calculator-popup
  position: relative
  max-width: 800px
  width: 100%
  margin: 50px auto 0
  padding: 30px 40px 30px
  background: rgb(255,255,255)
  box-shadow: 0 8px 10px 1px rgb(0 0 0 / 14%), 0 3px 14px 2px rgb(0 0 0 / 12%), 0 5px 5px -3px rgb(0 0 0 / 30%)
  z-index: 100
  &__container
  & .h2
    margin-top: 0

.calculator-counter 
  border: 1px solid #d1d1d1
  margin: 0
  padding: 0px 10px
  font-size: 18px
  background: #00c0ce
  color: #fff
  font-weight: 900

input::-webkit-outer-spin-button, input::-webkit-inner-spin-button 
  -webkit-appearance: none
  margin: 0 

input[type=number]
  -moz-appearance: textfield
  text-align: center

input[type=number]:disabled
  color: inherit

//.calculator-body
//  &__container
//    border:
//      left: 1px solid $menu-color
//      right: 1px solid $menu-color
//  &__description
//    width: 100%
//    border:
//      right: 1px solid $menu-color
</style>
