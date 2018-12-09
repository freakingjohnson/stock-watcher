<template>
  <div class="root">
    <div id="snackbar">Invalid stock symbol, please try again.</div>
    <div class="form-container">
      <h2>Stock Watcher</h2>
      <form>
        <input
          type="text"
          v-model="input"
          class="form-control"
          id="stockitem"
          placeholder="Enter stock symbol..."
        >
        <button type="button" v-on:click="add()">ADD</button>
      </form>
    </div>
    <br>
    <div class="list">
      <ul v-bind:class="{single: stocks.length <= 1, group: stocks.length > 1}">
        <h2 class="snack" v-bind:class="{hide: stocks.length > 0}">No stocks yet!</h2>
        <li v-for="(stock, index) in stocks" :key="index" class="list-group-item">
          <div class="left">
            <h3
              class="company-name"
            >{{stock.companyName.length > 19 ? `${stock.companyName.substring(0,19)}...` : stock.companyName}}</h3>
            <br>
            <div class="symbol">{{stock.symbol}}</div>
            <div class="price-container">
              <div class="change-percent">
                <span
                  class="triangle"
                  v-bind:class="{ up: stock.change > 0, down: stock.change < 0}"
                ></span>
                &nbsp;
                <span
                  class="percent"
                  v-bind:class="{green: stock.change > 0, red: stock.change < 0 }"
                >{{stock.change > 0 ? stock.change : Math.abs(stock.change)}}&nbsp;({{stock.changePercent.toFixed(2)}}%)</span>
              </div>
              <br>
              <h3 class="current-price">${{stock.latestPrice.toFixed(2)}}</h3>
            </div>
          </div>
          <div class="right">
            <div class="graph">
              <input
                class="slider"
                type="range"
                step=".01"
                :min="Math.min(stock.latestPrice, stock.low)"
                :max="Math.max(stock.latestPrice, stock.high)"
                :value="stock.latestPrice"
                disabled
              >
              <div class="hl high">${{stock.high.toFixed(2)}}</div>&nbsp;
              <div class="hl low">${{stock.low.toFixed(2)}}</div>
            </div>
          </div>
        </li>
      </ul>
    </div>
    <div id="snackbar">{{msg}}</div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "StockWatcher",
  data() {
    return {
      stocks: [],
      symbols: [],
      resData: {},
      input: "",
      msg: ""
    };
  },
  methods: {
    async add() {
      await axios
        .get(`https://api.iextrading.com/1.0/stock/${this.input}/quote`)
        .then(response => (this.resData = response.data))
        .catch(err => {
          this.warning("Invalid stock symbol, please try again.")
          console.log(this.msg)
          this.msg = ""
          });
      if (!this.symbols.includes(this.input.toUpperCase())) {
        this.symbols.push(this.resData.symbol);
        this.resData.symbol && this.stocks.push(this.resData);
        this.input = "";
        this.resData = {};
      }else{
        this.warning("You're already watching that stock")
        console.log(this.msg)
        this.msg = ""
        this.input = "";
        this.resData = {}
      }
    },
    warning(str) {
      this.msg = str
      var x = document.getElementById("snackbar");
      x.className = "show";
      setTimeout(function() {
        x.className = x.className.replace("show", "");
      }, 3000);
    }
  }
};
</script>

<style scoped>
ul {
  list-style-type: none;
  padding: 0;
  display: grid;
  grid-template-columns: auto auto;
  grid-gap: 10px;
  align-items: center;
  margin: auto;
  width: 70%;
}
button {
  height: 42px;
  width: 100px;
  border: none;
  border-radius: 2px;
  box-shadow: 1px 1px 5px #000;
  background-color: #72b118;
  color: #fff;
  font-weight: 600;
  cursor: pointer;
  outline: none;
}
button:hover {
  background-color: #48700f;
}
button:active {
  box-shadow: 0px 0px 5px #000;
  transform: translateY(4px);
  transform: translateX(4px);
}
.root {
  margin-bottom: 50px;
}
.list {
  width: 80%;
  margin: auto;
}
.form-control {
  height: 40px;
  width: 30%;
  margin-right: 10px;
  border: 1px solid grey;
  border-radius: 2px;
  padding-left: 20px;
}
.form-container {
  text-align: center;
  width: 90%;
  margin: auto;
}
.list-group-item {
  border: 1px solid gray;
  border-radius: 2px;
  display: flex;
  padding: 15px;
  margin: 0;
  background-color: #fff;
  max-width: 400px;
}
.company-name {
  font-size: 1em;
  margin: 0;
  position: absolute;
}
.symbol {
  padding-top: 2px;
  margin-bottom: 10px;
  position: relative;
  color: gray;
  font-size: 0.9em;
}
.change-percent {
  position: relative;
  top: 15px;
  right: 0;
}
.triangle {
  position: relative;
  width: 0;
  height: 0;
  border-left: 5px solid transparent;
  border-right: 5px solid transparent;
  top: 25%;
}
.percent {
  margin-left: 10px;
}
.up {
  position: absolute;
  border-bottom: 10px solid green;
}
.down {
  position: absolute;
  border-top: 10px solid red;
}
.green {
  color: green;
}
.red {
  color: red;
}
.current-price {
  position: relative;
  margin: 0;
}
.left {
  position: relative;
  /* margin-right: 10%; */
  width: 230px;
}
.right {
  display: inline-block;
  left: 65px;
  bottom: 6px;
  position: relative;
}
.hl {
  font-size: 10px;
}
.high {
  position: relative;
  bottom: 10px;
}
.low {
  position: relative;
  top: 55px;
}
.graph {
  height: 100px;
  padding-left: 5px;
}
.slider {
  position: relative;
  transform: rotate(270deg);
  height: 2px;
  right: 60px;
  top: 47px;
}
.single {
  grid-template-columns: auto;
  width: 35%;
}
input[type="range"] {
  -webkit-appearance: none;
  width: 100px;
  background: transparent;
}
input[type="range"]::-webkit-slider-thumb {
  -webkit-appearance: none;
  width: 0;
  height: 0;
  border-top: 10px solid gray;
  border-left: 5px solid transparent;
  border-right: 5px solid transparent;
  position: relative;
  bottom: 10px;
}
input[type="range"]::-webkit-slider-runnable-track {
  height: 2px;
  cursor: default;
  background: gray;
}
.snack {
  text-align: center;
  width: auto;
  color: darkgray;
}
.hide {
  display: none;
}
@media screen and (max-width: 1025px) {
  ul {
    width: 90%;
  }
  .list {
    width: 90%;
    margin: auto;
  }
}
@media screen and (max-width: 900px) {
  ul {
    width: 100%;
  }
  .list {
    width: 100%;
    margin: auto;
  }
}
@media screen and (max-width: 500px) {
  ul {
    grid-template-columns: auto;
    width: 90%;
  }
  .list-group-item {
    height: 30px;
  }
  .list {
    width: 100%;
    margin-left: -10px;
  }
  .graph {
    display: none;
  }
  .price-container {
    display: inline-block;
    left: 220px;
    bottom: 62px;
    position: relative;
  }
  .current-price {
    font-size: 0.8em;
  }
  .percent {
    font-size: 0.6em;
  }
  .company-name {
    font-size: 0.8em;
  }
  .symbol {
    font-size: 0.7em;
  }
  .form-control {
    padding: 0;
    font-size: 0.7em;
  }
}
@media screen and (max-width: 374px) {
  .price-container {
    display: inline-block;
    left: 153px;
  }
  .form-control {
    font-size: 0.54em;
  }
}
#snackbar {
  visibility: hidden;
  width: 80%;
  margin: auto;
  background-color: darkred;
  color: #fff;
  text-align: center;
  border-radius: 2px;
  padding: 16px;
  position: relative;
  z-index: 1;
  bottom: 30px;
}
#snackbar.show {
  visibility: visible;
  animation: fadein 0.5s, fadeout 0.5s 2.5s;
}

@keyframes fadein {
  from {
    left: -3000px;
    opacity: 1;
  }
  to {
    left: 30px;
    opacity: 1;
  }
}

@keyframes fadeout {
  from {
    left: 30px;
    opacity: 1;
  }
  to {
    left: -2000px;
    opacity: 1;
  }
}
</style>