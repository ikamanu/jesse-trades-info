<template>
  <section class="container">
    <Uploader/>

    <h1 class="text-capitalize">
      Backtest:
      <span class="mono-font font-size-16">
        {{ backtestName }}
      </span>
    </h1>

    <h2>Trades</h2>

    <client-only>
      <OHLC
        :orders="orders"
        :ohlc="candles"
        class="mb-10"/>
    </client-only>

    <TradesList
      :trades="trades"/>
  </section>
</template>

<style scoped lang="scss">
</style>

<script>

import OHLC from '@/components/OHLC'
import TradesList from '@/components/TradesList'
import Uploader from '@/components/Uploader'

export default {
  name: 'Backtest',
  components: {
    OHLC,
    TradesList,
    Uploader
  },
  async asyncData ({ $axios, route }) {
    let trades = []

    await $axios
      .$get('/api/read-file', {
        params: {
          fileName: route.params.backtest
        }
      })
      .then((result) => {
        trades = result.trades
      })

    // Get orders to draw them on chart
    let orders = trades.map((item) => {
      return [...item.orders]
    })

    orders = orders.flat()

    let candles = []

    if (trades.length > 0) {
      await $axios
        .$get('/api/candles', {
          params: {
            symbol: trades[0].symbol,
            exchange: trades[0].exchange,
            entryTimestamp: trades[0].opened_at,
            exitTimestamp: trades[trades.length - 1].closed_at
          }
        })
        .then((result) => {
          candles = result.data
        })
    }

    return {
      trades,
      candles,
      orders
    }
  },
  data () {
    return {
      orders: [],
      candles: [],
      trades: []
    }
  },
  computed: {
    backtestName () {
      return this.$route.params.backtest
    }
  },
  mounted () {
  },
  methods: {},
  head () {
    return {
      title: `Trades :: ${this.backtestName.charAt(0).toUpperCase() + this.backtestName.slice(1)}`
    }
  }
}
</script>
