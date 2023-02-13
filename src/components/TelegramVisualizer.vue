<template>
  <div
    class="code"
    v-for="(code, i) in table"
    v-bind:key="i"
    v-text="i + ' ' + code"
  ></div>
  <br />
  <label>Cursor Position: </label>
  <input v-model="input_x" />
  <br />
  <label>Text to be decoded: </label>
  <input v-model="input_data" />
  <br />
  <label>Decoded sequence: </label>
  <input readonly v-bind:value="decoded_sequence" />
  <br />
  <label>Space rate: </label>
  <input v-model="space_rate" />
  <br />
  <label>Ticks per second: </label>
  <input v-model="time_rate" />
  <br />
  <button
    @click="timer ? UnloadTimer() : loadTimer()"
    v-text="timer ? 'Unload' : 'Load'"
  ></button>
  <button @click="mounted_time = time">Refresh</button>
  <br />
  <svg>
    <line class="now" v-bind:x1="x" v-bind:x2="x" y1="0" y2="200" />
    <line
      class="point"
      v-for="(point, i) in time_sequence"
      v-bind:key="i"
      v-bind:x1="
        x + point * space_rate - (deltaTime * time_rate * space_rate) / 1000
      "
      v-bind:x2="
        x + point * space_rate - (deltaTime * time_rate * space_rate) / 1000
      "
      y1="0"
      y2="200"
    ></line>
  </svg>
</template>

<script lang="ts">
import { defineComponent } from "vue";
import { encode } from "morse-converter";

// a-z
const alphabetTable: Array<Array<0 | 1>> = [
  [0, 1], // a
  [1, 0, 0, 0],
  [1, 0, 1, 0],
  [1, 0, 0],
  [0], // e
  [0, 0, 1, 0],
  [1, 1, 0],
  [0, 0, 0, 0],
  [0, 0], // i
  [0, 1, 1, 1],
  [1, 0, 1],
  [0, 1, 0, 0],
  [1, 1],
  [1, 0],
  [1, 1, 1], // o
  [0, 1, 1, 0],
  [1, 1, 0, 1],
  [0, 1, 0],
  [0, 0, 0],
  [1],
  [0, 0, 1], // u
  [0, 0, 0, 1],
  [0, 1, 1],
  [1, 0, 0, 1],
  [1, 0, 1, 1],
  [1, 1, 0, 0],
];

const useMyOwnCode = false;

export default defineComponent({
  methods: {
    timeupdate() {
      this.time = Date.now();
    },
    loadTimer() {
      this.mounted_time = Date.now() - this.deltaTime;
      this.timer = setInterval(this.timeupdate);
    },
    UnloadTimer() {
      if (this.timer) {
        clearInterval(this.timer);
        this.timer = 0;
      }
    },
  },
  data: () => {
    return {
      input_x: "0",
      input_data: "Test text",
      timer: 0,
      mounted_time: 0,
      time: 0,
      space_rate: 100,
      time_rate: 2,
    };
  },
  unmounted() {
    this.UnloadTimer();
  },
  computed: {
    deltaTime: function (): number {
      return this.time - this.mounted_time;
    },
    x: function (): number {
      return parseFloat(this.input_x) || 0;
    },
    style: function (): string {
      return "x:" + this.x + ";";
    },
    table: function (): { [_: string]: string } {
      const res: { [_: string]: string } = {};
      alphabetTable.forEach((char: (0 | 1)[], index: number): void => {
        const key = String.fromCharCode(index + "a".charCodeAt(0));

        const value = char.reduce((pre: string, cur: number): string => {
          return pre + (cur ? "-" : ".");
        }, "");
        res[key] = value;
      });
      return res;
    },
    decoded_sequence: function (): string {
      if (useMyOwnCode) {
        let res = "";
        for (const c of this.input_data) {
          if (c === " ") {
            res += "      ";
          } else {
            res += this.table[c.toLowerCase()];
            res += " ";
          }
        }
        return res;
      } else {
        try {
          return encode(this.input_data);
        } catch (e) {
          console.error(e);
          return "?";
        }
      }
    },
    time_sequence: function (): number[] {
      let current = 0;
      const res: number[] = [];
      for (const c of this.decoded_sequence) {
        res.push(current);
        switch (c) {
          case "/":
          case "-":
            current += 3;
            break;
          case " ":
          case ".":
            current += 1;
            break;
          default:
            console.error(
              c +
                ' is not unexpected in "' +
                this.decoded_sequence +
                '", which is parsed from "' +
                this.input_data +
                '"'
            );
            break;
        }
      }
      return res;
    },
  },
});
</script>

<style scoped>
* {
  font-size: 2vw;
}

.code {
  width: 8em;
  text-align: left;
  display: inline-block;
}

label {
  width: 20vw;
  text-align: left;
  display: inline-block;
}

input {
  width: 40vw;
}

svg {
  width: 100%;
  height: 200;
  border: 5px black;
  background-color: grey;
}

line.now {
  stroke: red;
  stroke-width: 10;
}

line.point {
  stroke: black;
  stroke-width: 4;
}
</style>
