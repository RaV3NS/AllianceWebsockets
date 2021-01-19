<template>
    <div>
        <div class="loader-wrapper" v-if="loader">
            <span>Loading...</span>
            <v-btn
                text
                loading
            ></v-btn>
        </div>

        <v-row>
            <v-col lg="4" class="mt-10" cols="12">
                <v-row>
                    <v-col
                        v-for="(rate, index) in rates"
                        :key="index"
                        cols="6"
                    >
                        <v-card
                            elevation="2"
                        >
                            <div class="card-wrapper" style="cursor: pointer" @click="changeChart(index)">
                                <h4>{{ index }}</h4>
                                <span>{{ rate.toFixed(2) }} $</span>
                            </div>
                        </v-card>
                    </v-col>
                </v-row>
            </v-col>

            <v-col lg="8" cols="12">
                <h1 class="currency_heading" v-if="rates.BTC">1 {{ symbol }} = {{ rates[symbol] }}$</h1>
                <v-sparkline
                    :fill="false"
                    :gradient="['#07d0e6']"
                    :line-width="0.5"
                    :padding="5"
                    :smooth="0"
                    :value="values"
                    auto-draw
                    style="overflow: visible; width: 80%; margin: auto"
                >
                    <template v-slot:label="item">
                        $ {{ item.value }}
                    </template>
                </v-sparkline>
            </v-col>
        </v-row>

    </div>
</template>

<script>
    export default {
        data: function() {
            return {
                connection: null,
                symbol: 'BTC',
                cryptos: ['BTC', 'LTC', 'ETH', 'XRP', 'BCH', 'XLM', 'XMR', 'LINK', 'ADA', 'YFI', 'NEO', 'ATOM', 'OMG', 'BUSD', 'BNB', 'XTZ'],
                rates: {},
                data_source: 'Binance',
                currency: 'USDT',
                chart_limit: 10,
                values_data: {},
                values: [],
                loader: true
            }
        },
        computed: {
            getChartLimit() {
                return this.chart_limit - 1;
            }
        },
        mounted() {
            const api_key = '0083a36ca0d058fa61f569c473211920e4e5deaec753b55d6ba8a31c230f6dd9'
            this.connection = new WebSocket('wss://streamer.cryptocompare.com/v2?api_key=' + api_key);

            this.connection.onmessage = (event) => {
                this.onMessage(event);
            }

            this.subscribe();
        },
        methods: {
            onMessage(event) {
                let data = JSON.parse(event.data);

                // if price exist (not heartbeat or smth)
                if (data.PRICE) {
                    // push to chart data
                    if (data.FROMSYMBOL === this.symbol) {
                        if (this.values.length > this.getChartLimit) this.values.shift();
                        this.values.push(data.PRICE);
                    }

                    // currency history
                    if (this.values_data[data.FROMSYMBOL].length > this.getChartLimit) this.values_data[data.FROMSYMBOL].shift();
                    this.values_data[data.FROMSYMBOL].push(data.PRICE);
                    this.rates[data.FROMSYMBOL] = data.PRICE;
                }
            },
            changeChart(symbol) {
                this.symbol = symbol;
                this.values = this.values_data[symbol];
            },
            subscribe() {
                if (this.connection.readyState === 1) {
                    this.loader = false;

                    let subs = [];
                    this.cryptos.map((el) => {
                        this.$set(this.rates, el, 0);
                        this.$set(this.values_data, el, []);
                        subs.push(`2~${this.data_source}~${el}~${this.currency}`);
                    });

                    this.connection.send(JSON.stringify({ action: 'SubAdd', subs: subs }));
                } else {
                    setTimeout(() => {
                        this.subscribe();
                    }, 1000);
                }
            }
        },
        beforeRouteLeave (to, from, next) {
            // close websocket connection after route leave
            this.connection.close();
            next();
        }
    }
</script>

<style lang="scss" scoped>
    .currency_heading {
        text-align: center;
        margin-bottom: 2rem;
        margin-top: 2rem
    }

    .card-wrapper {
        display: flex;
        justify-content: space-between;
        align-items: center;
        padding: 1rem;

        h4 {
            font-weight: 400;
        }
    }

    .loader-wrapper {
        display: flex;
        justify-content: center;
        align-items: center;
        height: 80vh;
        flex-direction: column;
    }
</style>