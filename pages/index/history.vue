<template>
  <div>
    <h1 class="text-center">Burning history</h1>
    <transition name="fade" mode="out-in">
      <div v-if="state === 'loading'" key="loading" class="text-center">
        <loader />
        <p>Loading burning history...</p>
      </div>
      <div v-if="state === 'loaded' && history.length > 0" key="loaded">
        <div class="table">
          <div class="tr">
            <div class="th date">Date</div>
            <div class="th amount">Amount</div>
            <div class="th tx">Transaction</div>
          </div>

          <div v-for="burn in history" :key="burn.ref" class="tr">
            <div class="td date" v-text="burn.completedAt" />
            <div class="td amount">{{ parseBalance(burn.amount) }} ZXC</div>
            <div class="td tx ellipsis">
              <a :href="`https://etherscan.io/tx/${burn.txHash}`">
                {{ burn.txHash }}
              </a>
            </div>
          </div>
        </div>
        <div class="text-center">
          <n-link class="button mt-2" to="/">Go back</n-link>
        </div>
      </div>
      <div v-if="state === 'loaded' && history.length < 1" class="text-center">
        <p>There's no burning events recorded so far.</p>
        <n-link class="button mt-2" to="/">Go back</n-link>
      </div>
      <div v-if="state === 'error'" key="error" class="text-center">
        <h2>Something went wrong</h2>
        <p>{{ error }}</p>
      </div>
    </transition>
  </div>
</template>

<script>
import Loader from '~/components/Loader'
export default {
  components: {
    Loader
  },
  data() {
    return {
      state: 'loading',
      history: null,
      error: '',
      headers: ['Date', 'Amount', 'transaction']
    }
  },
  created() {
    this.fetchHistory()
  },
  methods: {
    async fetchHistory() {
      try {
        const data = await this.$axios
          .get('/stats/burns')
          .then((res) => res.data.data)
        this.history = data
        this.state = 'loaded'
      } catch (error) {
        this.error = error
        this.state = 'error'
      }
    }
  }
}
</script>

<style lang="scss" scoped>
.table {
  margin-top: 2rem;

  .tr {
    display: flex;

    &:nth-child(even) {
      background: rgba(0, 0, 0, 0.4);
    }
  }

  .td,
  .th {
    font-size: 16px;
    padding: 10px;
  }

  .th {
    font-weight: 500;
  }

  .tx {
    flex-basis: 44%;
    justify-self: flex-end;
  }

  .date {
    flex-basis: 28%;
  }

  .amount {
    flex-basis: 20%;
    flex-grow: 1;
  }
}

.ellipsis {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}
</style>