<template>
  <div class="root">
    <div id="snackbar">{{msg}}</div>
    <div class="form-container">
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
    </div>
    <br>
    <div class="list">
      <ul v-bind:class="{single: stocks.length <= 1, group: stocks.length > 1}">
        <h2 class="stock-placeholder" v-bind:class="{hide: stocks.length > 0}">No stocks yet!</h2>
        <li v-for="(stock, index) in stocks" :key="index" class="list-group-item" v-on:click="deleteStock(index)">
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
  </div>
</template>

<script>
import axios from "axios";
import '../css/styles.css';

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
        .catch(() => {
          this.warning("Invalid stock symbol, please try again.");
        });
      if (!this.symbols.includes(this.input.toUpperCase())) {
        this.symbols.push(this.resData.symbol);
        this.resData.symbol && this.stocks.push(this.resData);
      } else {
        this.warning("You're already watching that stock");
      }
      this.input = "";
      this.resData = {};
    },
    deleteStock(i) {
      this.stocks.splice(i,1);
      this.symbols.splice(i,1);
    },
    warning(str) {
      this.msg = str;
      var x = document.getElementById("snackbar");
      x.className = "show";
      setTimeout(function() {
        x.className = x.className.replace("show", "");
      }, 3000);
    }
  }
};
</script>