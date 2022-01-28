<template>
  <div class="home">
    <div v-if="selecting">
      <b-button variant="success" @click="admin=true;selecting=false;">I AM ADMIN</b-button>

      <b-button variant="info" @click="admin=false;selecting=false;">I AM PARTICIPANT</b-button>
    </div>
    <div v-else>
      <div v-if="admin">
        <b-button
          variant="info"
          @click="callBingoNumber()"
          :disabled="bingo.callCounter==75 || verifyWinnerOrLose===true"
        >Bingo Numbers</b-button>

        <div class="text-center">
          <div class="ball" v-if="getLastNumberBingo!==null">
            <div class="text">{{ getLastNumberBingo }}</div>
          </div>
          <div class>
            Last Balls
            <br />
            <span v-for="(n,k) in bingo.numbersCalled" :key="k">
              {{ n }}
              <span>&emsp;</span>
            </span>
          </div>
        </div>
      </div>
      <div class="bingo">
        <div class="winner" v-if="verifyWinnerOrLose===true">
          <div class="text">
            YOU ARE WINNER
            <br />🤩🤩🤩
          </div>
        </div>
        <div class="looser" v-if="verifyWinnerOrLose===false">
          <div class="text">
            NOT THE WINNER
            <br />😩😩😩
          </div>
        </div>
        <div class="card" v-if="verifyWinnerOrLose===null">
          <div class="headers">
            <div>
              <span>B</span>
            </div>
            <div>
              <span>I</span>
            </div>
            <div>
              <span>N</span>
            </div>
            <div>
              <span>G</span>
            </div>
            <div>
              <span>O</span>
            </div>
          </div>
          <div class="container-numbers">
            <div class="column 1" v-for="(c,k) in bingo.cards" :key="k">
              <div
                class="number"
                :class="markNumber(nn)"
                v-for="(nn,kk) in c"
                :key="kk"
                @click="addOrRemove(nn)"
              >
                <span>{{nn}}</span>
              </div>
            </div>
          </div>
          <div v-if="!admin" class="text-center">
            <b-button variant="danger" @click="forceLose=true">END GAME</b-button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
// @ is an alias to /src

export default {
  name: "Home",
  components: {},
  data() {
    return {
      bingo: {
        config: {
          start: 1,
          end: 75,
          height: 5,
          width: 5,
          limits: [
            { column: "B", lower_bound: 1, uper_bound: 15 },
            { column: "I", lower_bound: 16, uper_bound: 30 },
            { column: "N", lower_bound: 31, uper_bound: 45 },
            { column: "G", lower_bound: 46, uper_bound: 60 },
            { column: "O", lower_bound: 61, uper_bound: 75 }
          ],
          middleBlank: true
        },
        callCounter: 0,
        cards: [],
        bingoNumbers: [],
        numbersCalled: []
      },
      selecting: true,
      admin: false,
      forceLose: false
    };
  },
  computed: {
    getLastNumberBingo() {
      const numbers = this.bingo.numbersCalled;
      const last = numbers.length - 1;
      if (last === -1) {
        return null;
      }
      return numbers[last];
    },
    verifyWinnerOrLose() {
      const numbers = this.bingo.numbersCalled;
      const size = numbers.length;
      if (this.forceLose) {
        return false;
      }
      if (size == this.bingo.config.end) {
        return false;
      }
      return this.verifyAllBingoNumbers(this.bingo.cards, numbers);
    }
  },
  mounted() {
    this.generateBingo();
  },
  methods: {
    addOrRemove(nn) {
      if (!this.admin) {
        var numbers = this.bingo.numbersCalled;
        var exist = null;
        numbers.forEach((el1, k) => {
          if (el1 == nn) {
            exist = k;
          }
        });
        if (exist !== null) {
          this.bingo.numbersCalled.splice(exist);
        } else {
          this.bingo.numbersCalled.push(nn);
        }
      }
    },
    markNumber(n) {
      var exist = false;
      this.bingo.numbersCalled.forEach(el1 => {
        if (el1 == n) {
          exist = true;
        }
      });
      return exist ? "mark" : "";
    },
    callBingoNumber() {
      const config = this.bingo.config;
      this.bingo.callCounter++;
      var numbers = this.bingo.bingoNumbers;

      const random = this.between(config.start, config.end, numbers);
      const numberSelected = numbers[random];
      console.log([numberSelected, numbers, random]);
      this.bingo.numbersCalled.push(numberSelected);
      this.bingo.bingoNumbers.splice(random, 1);
    },
    generateBingo() {
      const config = this.bingo.config;
      var genCards = [];
      //generate number in base of start and end
      var listNumbers = this.generateNumbers(config.start, config.end);
      this.bingo.bingoNumbers = this.generateNumbers(config.start, config.end);
      //recursive for to base height
      for (let index = 1; index <= config.height; index++) {
        // recursive for to base width
        for (let index2 = 1; index2 <= config.width; index2++) {
          //fix index to base Array
          const indexHeight = index - 1;
          if (
            config.middleBlank &&
            this.isMiddleCard(indexHeight, index2 - 1)
          ) {
            genCards[indexHeight].push("FREE");
          } else {
            //get Limit in base of index
            const limits = config.limits[indexHeight];
            //get random number in base of limits, when exists in the list Number
            const random = this.between(
              limits.lower_bound,
              limits.uper_bound,
              listNumbers
            );
            //if is the first empty array, fix set [] to init push and others methods
            if (!genCards[indexHeight]) {
              genCards[indexHeight] = [];
            }
            //push the var
            genCards[indexHeight].push(listNumbers[random]);
            listNumbers.splice(random, 1);
          }
        }
      }
      this.bingo.cards = genCards;
    },
    generateNumbers(start, end) {
      var list = [];
      for (let index = start; index <= end; index++) {
        list.push(index);
      }
      return list;
    },
    between(min, max, array) {
      let fixMax = null;
      let fixMin = null;
      let first = false;
      for (let index = 0; index < array.length; index++) {
        const element = array[index];
        if (element <= max) {
          if (element >= min && !first) {
            fixMin = index;
            first = true;
          }
          fixMax = index;
        }
      }
      if (fixMax === null || fixMin === null) {
        return 0;
      }
      return Math.floor(Math.random() * (fixMax - fixMin) + fixMin);
    },
    isMiddleCard(index, index2) {
      const config = this.bingo.config;
      const total = config.height * config.width;
      let counter = index * config.width;
      counter = counter + index2 + 1;
      var middle = total / 2;
      if (counter == middle.toFixed(0)) {
        return true;
      } else {
        return false;
      }
    },
    verifyAllBingoNumbers(cards, numbers) {
      const config = this.bingo.config;
      const middle = config.middleBlank ? -1 : 0;
      var size = config.height * config.width;
      size = size + middle;
      var counterBingo = 0;
      numbers.forEach(el1 => {
        if (this.existNumberCard(cards, el1)) {
          counterBingo++;
        }
      });
      console.log([size, counterBingo]);
      return counterBingo == size ? true : null;
    },
    existNumberCard(cards, number) {
      var exist = false;
      cards.forEach(el1 => {
        el1.forEach(el2 => {
          if (el2 == number) {
            exist = true;
          }
        });
      });
      return exist;
    }
  }
};
</script>
<style lang="scss">
.ball {
  border-radius: 100%;
  width: 100px;
  height: 100px;
  display: flex;
  align-items: center;
  text-align: center;
  border: solid;
  margin: 0 auto;
  .text {
    text-align: center;
    width: 100%;
  }
}
.bingo {
  $border: 1px solid #000;
  $width: 600px;
  $bg: #446444;
  $hugeText: 60px;
  $bigText: 42px;
  $smallText: 32px;
  $white: #fff;
  min-height: 500px;

  position: relative;

  .container-numbers {
    margin: 0 auto;
  }
  .clear {
    width: 100%;
    clear: both;
  }
  .card {
    margin: 0 auto;
    width: $width;
    height: auto;
    background-color: $bg;
    border-radius: 10px;
    padding: 10px;
  }
  button {
    float: left;
  }
  .headers > div {
    float: left;
    width: ($width/5)- 10px;
    text-align: center;
  }
  .headers > div span {
    font-size: $bigText;
    color: #fff;
    font-weight: bold;
  }
  .column {
    float: left;
    width: ($width / 5) - 10px;
    text-align: center;
  }
  .number {
    padding: 20px 0;
    border: $border;
    background-color: $white;
  }
  .number span {
    color: #000;
    font-size: $smallText;
  }
  .number span:hover {
    text-shadow: 0 0 5px rgba(0, 0, 0, 0.5);
  }
  .number {
    &.mark {
      background-color: yellow;
    }
  }
  .winner,
  .looser {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    z-index: 1;
    height: 100%;
    color: white;
    font-weight: bold;
    font-size: 88px;
    display: flex;
    align-items: center;
    .text {
      width: 100%;
      text-align: center;
    }
  }
  .winner {
    background-color: yellow;
  }

  .looser {
    background-color: black;
  }
}
</style>
