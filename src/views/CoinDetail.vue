<template>
  <div class="flex-col">
    <div class="text-center">
      <SyncLoader :loading="isLoading" :color="'#68D391'" :size="10" />
    </div>
    <template v-if="!isLoading">
      <div class="flex flex-col sm:flex-row justify-around items-center">
        <div class="flex flex-col items-center">
          <img
            class="w-20 h-20 mr-5"
            :src="`https://static.coincap.io/assets/icons/${asset.symbol.toLowerCase()}@2x.png`"
            :alt="asset.name"
          />
          <h1 class="text-5xl">
            {{ asset.name }}
            <small class="sm:mr-2 text-gray-500"> {{ asset.symbol }}</small>
          </h1>
        </div>

        <div class="my-10 flex flex-col">
          <ul>
            <li class="flex justify-between">
              <b class="text-gray-600 mr-10 uppercase">Ranking</b>
              <span># {{ asset.rank }} </span>
            </li>
            <li class="flex justify-between">
              <b class="text-gray-600 mr-10 uppercase">Precio actual</b>
              <span> {{ asset.priceUsd | dollar }} </span>
            </li>
            <li class="flex justify-between">
              <b class="text-gray-600 mr-10 uppercase">Precio más bajo</b>
              <span> {{ min | dollar }} </span>
            </li>
            <li class="flex justify-between">
              <b class="text-gray-600 mr-10 uppercase">Precio más alto</b>
              <span> {{ max | dollar }} </span>
            </li>
            <li class="flex justify-between">
              <b class="text-gray-600 mr-10 uppercase">Precio Promedio</b>
              <span> {{ avg | dollar }} </span>
            </li>
            <li class="flex justify-between">
              <b class="text-gray-600 mr-10 uppercase">Variación 24hs</b>
              <span> {{ asset.changePercent24Hr | percent }} </span>
            </li>
          </ul>
        </div>

        <div class="my-10 sm:mt-0 flex flex-col justify-center text-center">
          <button
            class="bg-green-500 hover:bg-green-700 text-white font-bold py-2 px-4 rounded"
            @click="toggleConverter"
          >
            {{ fromUsd ? `USD a ${asset.symbol} ` : `${asset.symbol} a USD ` }}
          </button>

          <div class="flex flex-row my-5">
            <label class="w-full" for="convertValue">
              <input
                id="convertValue"
                type="number"
                class="text-center bg-white focus:outline-none focus:shadow-outline border border-gray-300 rounded-lg py-2 px-4 block w-full appearance-none leading-normal"
                v-model="convertValue"
                :placeholder="
                  fromUsd ? `USD a ${asset.symbol} ` : `${asset.symbol} a USD `"
              />
            </label>
          </div>

          <span class="text-xl">
            {{ convertResult }}
            {{ fromUsd ? `USD a ${asset.symbol} ` : `${asset.symbol} a USD ` }}
          </span>
        </div>
      </div>

      <line-chart
        class="my-10"
        :color="['red']"
        :min="min"
        :max="max"
        :data="history.map((h) => [h.date, parseFloat(h.priceUsd).toFixed(2)])"
      />

      <h3 class="text-xl my-10">Mejores Ofertas de Cambio</h3>
      <table>
        <tr
          class="border-b"
          v-for="m in martkets"
          :key="`${m.exchangeId}-${m.priceUsd}`"
        >
          <td>
            <b> {{ m.exchangeId }} </b>
          </td>
          <td>{{ m.priceUsd | dollar }}</td>
          <td>{{ m.baseSymbol }} / {{ m.quoteSymbol }}</td>
          <td>
            <px-button
              :is-loading="m.isLoading || false"
              @custom-click="getWebsite(m)"
              v-if="!m.url"
            >
              <slot>Obtener Link</slot>
            </px-button>
            <a
              v-else
              class="hover:underline text-green-600 cursor-pointer"
              target="_blanck"
              :href="m.url"
            >
              {{ m.url }}
            </a>
          </td>
        </tr>
      </table>
    </template>
  </div>
</template>
<script>
import api from '@/api.js'
import PxButton from '@/components/PxButton.vue'
export default {
  name: 'CoinDetail',
  components: {
    PxButton,
  },
  data() {
    return {
      asset: {},
      history: [],
      isLoading: false,
      martkets: [],
      fromUsd: true,
      convertValue: null,
    }
  },
  computed: {
    convertResult() {
      if (!this.convertValue) {
        return 0
      }
      const result = this.fromUsd
        ? this.convertValue / this.asset.priceUsd
        : this.convertValue * this.asset.priceUsd

      return result.toFixed(4)
    },
    min() {
      return Math.min(
        ...this.history.map((h) => parseFloat(h.priceUsd).toFixed(2))
      )
    },
    max() {
      return Math.max(
        ...this.history.map((h) => parseFloat(h.priceUsd).toFixed(2))
      )
    },
    avg() {
      return Math.abs(
        ...this.history.map((h) => parseFloat(h.priceUsd).toFixed(2))
      )
    },
  },
  watch: {
    $route() {
      this.getCoin()
    },
  },
  created() {
    this.getCoin()
  },
  methods: {
    toggleConverter() {
      this.fromUsd = !this.fromUsd
    },
    getWebsite(exchange) {
      this.$set(exchange, 'isLoading', true)

      return api
        .getExchange(exchange.exchangeId)
        .then((res) => {
          this.$set(exchange, 'url', res.exchangeUrl)
        })
        .finally(() => {
          this.$set(exchange, 'isLoading', false)
        })
    },
    getCoin() {
      this.isLoading = true
      // obtener el id de la url
      const id = this.$route.params.id
      Promise.all([
        api.getAsset(id),
        api.getAssetHistory(id),
        api.getMarkerts(id),
      ])
        .then(([asset, history, martkets]) => {
          this.asset = asset
          this.history = history
          this.martkets = martkets
        })
        .finally(() => {
          this.isLoading = false
        })
    },
  },
}
</script>
<style scoped>
td {
    padding: 10px;
    text-align: center;
}
</style>