<template>
  <div class="root">
    <div id="snackbar">{{msg}}</div>
    <header class="form-container">
      <h2>Stock Watcher</h2>
      <input
        type="text"
        v-model="input"
        class="form-control"
        id="stockitem"
        placeholder="Enter stock symbol..."
        v-on:keyup.enter="add()"
      >
      <button type="button" v-on:click="add()">ADD</button>
    </header>
    <br>
    <div class="list">
      <ul v-bind:class="{single: stocks.length <= 1}">
        <h2 class="stock-placeholder" v-bind:class="{hide: stocks.length > 0}">No stocks yet!</h2>
        <li
          v-for="(stock, index) in stocks"
          :key="index"
          class="list-group-item"
          v-on:click="deleteStock(index)"
        >
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
                  v-bind:class="{greentext: stock.change > 0, redtext: stock.change < 0 }"
                >{{stock.change > 0 ? stock.change : Math.abs(stock.change)}}&nbsp;({{stock.changePercent.toFixed(2)}}%)</span>
              </div>
              <br>
              <h3 class="current-price">${{stock.latestPrice.toFixed(2)}}</h3>
            </div>
          </div>
          <div class="right graph">
            <input
              class="slider"
              type="range"
              step=".01"
              :min="stock.low"
              :max="stock.high"
              :value="stock.latestPrice"
              disabled
            >
            <p class="hl high">${{stock.high.toFixed(2)}}</p>&nbsp;
            <p class="hl low">${{stock.low.toFixed(2)}}</p>
          </div>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import "../css/styles.css";

export default {
  name: "StockWatcher",
  data() {
    return {
      stocks: [],
      symbols: [],
      resData: {},
      input: "",
      msg: "",
      w: window.innerWidth
    };
  },
  methods: {
    async add() {
      if (this.input === "q" || this.input === "Q" || !this.input) {
        this.message("Invalid stock symbol, please try again.", "red");
      }
      this.input &&
        this.input !== "q" &&
        this.input !== "Q" &&
        (await axios
          .get(`https://api.iextrading.com/1.0/stock/${this.input}/quote`)
          .then(response => {
            this.resData = response.data;
            if (!this.symbols.includes(this.input.toUpperCase())) {
              this.symbols.push(this.resData.symbol);
              this.resData.symbol && this.stocks.push(this.resData);
              this.message("Stock added successfully!", "green");
            } else {
              this.message("You're already watching that stock", "orange");
            }
          })
          .catch(err => {
            this.message("Invalid stock symbol, please try again.", "red");
            console.log(err);
          }));

      this.input = "";
      this.resData = {};
    },
    deleteStock(i) {
      this.stocks.splice(i, 1);
      this.symbols.splice(i, 1);
      this.message("Stock removed successfully", "darkgray");
    },
    message(str, color) {
      this.msg = str;
      var x = document.getElementById("snackbar");
      x.className = `show ${color}`;
      setTimeout(function() {
        x.className = x.className.replace("show", "");
      }, 3000);
    }
  }
};
</script>