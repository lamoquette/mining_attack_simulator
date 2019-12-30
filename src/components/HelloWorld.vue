<template>
  <v-container fluid class="grey lighten-5">
    <v-row no-gutters>
      <v-col cols="12" md="9">
        <apexchart type="line" :options="chartOptions" :series="series" />
      </v-col>
      <v-col cols="12" md="3">
        <div class="slidecontainer2">
          <p>Parameters:</p>
          <p>
            hashrate (q) in %: <span id="demo">{{ hashrate }}</span>
          </p>
          <v-slider type="range"
            thumb-label
            min="0.5"
            max="50"
            v-model="hashrate"
            step="0.5"></v-slider>

          <p>
            number of confirmations (z) for coinbase:
            <span id="confirmations-label">{{ number_of_confirmations }}</span>
          </p>
          <v-slider type="range"
            thumb-label
            min="3"
            max="50"
            v-model="number_of_confirmations"
            step="0.5"></v-slider>

          <p>
            maximum authorized delay (A):
            <span id="delay-label">{{ delay }}</span>
          </p>
          <v-slider type="range"
            thumb-label
            min="1"
            max="50"
            v-model="delay"
            step="1"></v-slider>

          <p>
            number of attacks:
            <span id="attacks-label">{{ number_of_attacks }}</span>
          </p>
          <v-slider type="range"
            thumb-label
            min="10"
            max="1000"
            v-model="number_of_attacks"
            step="10"></v-slider>

          <p>
            number of cycles per attack:
            <span id="cycles-label">{{ number_of_cycles }}</span>
          </p>
          <v-slider type="range"
            thumb-label
            min="10"
            max="500"
            v-model="number_of_cycles"
            step="1"></v-slider>

          <v-btn v-on:click="launch_simulation()" block color="primary">Simulation</v-btn>
        </div>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
export default {
  name: "home",
  created() {
    this.launch_simulation();
  },
  data: function() {
    return {
      hashrate: 0.5,
      media: 5,
      number_of_confirmations: 5,
      delay: 20,
      number_of_cycles: 100,
      number_of_attacks: 60,
      honest_cycles_result: [],
      attacker_cycles_result: [],
      series: [
        {
          name: "Honest Miners",
          data: this.honest_cycles_result
        },
        {
          data: this.attacker_cycles_result,
          name: "Attacker"
        }
      ],
      chartOptions: {
        chart: {
          zoom: {
            enabled: false
          }
        },
        legend: {
          position: 'top',
        },
        responsive: [
          {
            breakpoint: 1000,
            options: {
              plotOptions: {
                bar: {
                  horizontal: false
                }
              }
            }
          }
        ],
        dataLabels: {
          enabled: false
        },
        stroke: {
          curve: "straight"
        },
        title: {
          text: "Income ratio by hashrate",
          align: "left"
        },
        grid: {
          row: {
            colors: ["#f3f3f3", "transparent"], // takes an array which will be repeated on columns
            opacity: 0.5
          }
        },
        xaxis: {
          categories: this.honest_cycles_result,
          labels: {
            style: {
              colors: [],
              fontSize: "12px",
              fontFamily: "Helvetica, Arial, sans-serif",
              cssClass: "apexcharts-xaxis-label"
            },
            formatter: function(value) {
              return value.toFixed(3);
            }
          }
        },
        yaxis: {
          labels: {
            formatter: function(value) {
              return value.toFixed(4);
            }
          }
        }
      }
    };
  },
  methods: {
    start_attack: function(hashrate) {
      // medium hashrate
      var q = hashrate; // (between 0 et 50)

      // nombre de confirmations
      var z = this.number_of_confirmations;

      // spend attack (bitcoin)
      var v = 10;

      // maximum authorized delay
      var A = this.delay;

      // block premined
      var premined_blocks = 1 / q;

      // nombre d'attaque / bloc miné max par cycle
      var number_of_cycles = this.number_of_cycles;

      // Miner's revenue at the end of the attack cycle
      var reward = 0;

      function total_rewards() {
        console.log("reward: " + reward);
        console.log("premined_blocks: " + premined_blocks);
        return reward / premined_blocks;
      }

      function start_cycle() {
        // number of block mined by honest miners
        var honest_miners_blocks_validated = 0;

        // number of block mined by attacker miner
        var attacker_blocks_validated = 0;

        // retard entre la blockchain officiel et la blockchain fork. True si dépassé, false sinon.
        function delay_exceeded() {
          return (
            A <
            Math.abs(honest_miners_blocks_validated - attacker_blocks_validated)
          );
        }

        // nombre de block z en avance par rapport aux attaquants
        function attacker_block_ahead() {
          return (
            attacker_blocks_validated > honest_miners_blocks_validated &&
            honest_miners_blocks_validated >= z
          );
        }

        // 1 : mined by honest miner - 0 : mined by attacker
        function block_mined() {
          var random = Math.random() * (100 - 0) + 0;

          if (q >= random) {
            attacker_blocks_validated++;
          } else {
            honest_miners_blocks_validated++;
          }
        }

        function add_rewards() {
          reward += attacker_blocks_validated + v;
        }

        while (!delay_exceeded() && !attacker_block_ahead()) {
          block_mined();
          premined_blocks++;
        }
        /*
        var cycle_result = {
          hashrate: hashrate,
          total_rewards: attacker_block_ahead() ? total_rewards() : 0,
          attacker_blocks_validated: attacker_blocks_validated,
          honest_miners_blocks_validated: honest_miners_blocks_validated,
          rentabilite: attacker_block_ahead() ? total_rewards() : 0
        };*/

        //console.log(cycle_result)

        if (attacker_block_ahead()) add_rewards();

        //return cycle_result.rentabilite;
      }

      for (var i = 0; i < this.number_of_cycles; i++) start_cycle();

      return total_rewards();
    },
    launch_simulation: function() {
      var hashrate_step = 50 / this.number_of_attacks;
      this.attacker_cycles_result = [];
      this.honest_cycles_result = [];

      for (var j = this.hashrate / 100; j <= 50; j += hashrate_step) {
        this.attacker_cycles_result.push(this.start_attack(j));
        this.honest_cycles_result.push(j / 100);
      }

      console.log(this.attacker_cycles_result);
      console.log(this.honest_cycles_result);

      this.series = [];
      this.chartOptions.xaxis.categories = [];
      this.series.push({
        data: this.attacker_cycles_result,
        name: "Attacker"
      });
      this.series.push({
        data: this.honest_cycles_result,
        name: "Honest Miners"
      });
      this.chartOptions = {
        ...this.chartOptions,
        ...{
          chart: {
            zoom: {
              enabled: false
            }
          },
          xaxis: {
            categories: this.honest_cycles_result,
            labels: {
              formatter: function(value) {
                return value.toFixed(3);
              }
            }
          }
        }
      };

      //create_chart(honest_cycles_result, attacker_cycles_result);
      //this.renderChart(data, options);
    }
  }
};
</script>

<style>
.slidecontainer2 {
  width: 100%;
  height: 100%;
}

.slider {
  -webkit-appearance: none;
  width: 100%;
  height: 25px;
  background: #d3d3d3;
  outline: none;
  opacity: 0.7;
  -webkit-transition: 0.2s;
  transition: opacity 0.2s;
}

.slider:hover {
  opacity: 1;
}

.slider::-webkit-slider-thumb {
  -webkit-appearance: none;
  appearance: none;
  width: 25px;
  height: 25px;
  background: #87619e;
  cursor: pointer;
}

.slider::-moz-range-thumb {
  width: 25px;
  height: 25px;
  background: #87619e;
  cursor: pointer;
}
</style>
