<script setup>
import { reactive, ref, computed } from "vue";
import { FwbSelect, FwbToggle, FwbInput, FwbButton } from "flowbite-vue";

const GameTypes = reactive([
  { value: "Classic", name: "Classic" },
  { value: "Complex", name: "Complex" },
]);
const Teaming = ref(true); // team by default
const GameType = ref("Classic");
const GameRounds = computed(() => {
  let rounds = {
    Classic: 5,
    Complex: 2,
  };
  return rounds[GameType.value];
});
const TeamPlayers = reactive(["player 1", "player 2", "player 3", "player 4"]);
let gameInSession = ref(false);
let timeGameStarted = ref(null);
let timeGameEnded = ref(null);
function startGame() {
  gameInSession.value = true;
  timeGameStarted.value = new Date();
}
function endGame() {
  gameInSession.value = false;
  timeGameEnded.value = null;
  timeGameStarted.value = null;
  records.splice(0, records.length);
  summaries = 0;
}

const GameSummary = computed(() => records.filter((i) => i.type == "Round").reduce((a, b) => a + parseInt(b.point), 0));

let recordPoint = ref("");
const RecordTypes = computed(() => {
  let types = {
    Classic: ["King of Hearts", "Queens", "Diamonds", "Lutoosh", "Trix"],
    Complex: ["Complex", "Trix"],
  };
  types = types[GameType.value];
  let roundRecord = records.length % types.length;
  roundRecord = records.slice(records.length - roundRecord);
  roundRecord = roundRecord.map((record) => record.type);
  return types.filter((i) => !roundRecord.includes(i)).map((type) => ({ value: type, name: type }));
});
const recordType = ref(null);
const recordTypeComputed = computed({
  get() {
    console.log(RecordTypes.value);
    return recordType.value ?? RecordTypes.value[0]?.value;
  },
  set(value) {
    recordType.value = value;
  },
});
let summaries = 0;
const records = reactive([]);
function addRecord() {
  records.unshift({
    type: recordTypeComputed.value,
    point: !parseInt(recordPoint.value) ? 0 : parseInt(recordPoint.value),
  });
  if ((records.length - summaries) % GameRounds.value == 0) {
    records.unshift({
      type: "Round",
      point: records
        .slice(0, GameRounds.value)
        .reduce((a, b) => a + (b.type == "Trix" ? parseInt(b.point) : -parseInt(b.point)), 0),
    });
    summaries++;
    if (summaries == 4) {
      timeGameEnded.value = new Date();
    }
  }
  recordPoint.value = "";
  recordType.value = null;
}
</script>

<template>
  <main class="container p-2">
    <h1 class="my-3 text-gray-700 text-center text-5xl">Trix</h1>

    <div v-if="gameInSession">
      <p class="text-center text-l">
        <b>{{ GameType }}</b> Game In Session
      </p>
      <h1
        class="text-2xl my-10 text-center"
        :class="{ 'text-green-400': GameSummary > 0, 'text-red-500': GameSummary < 0 }"
      >
        <b>{{ GameSummary }}</b> Points
      </h1>
      <form v-if="summaries < 4" class="my-5" @submit.prevent.stop="addRecord">
        <fwb-select v-model="recordTypeComputed" :options="RecordTypes" label="Select a Game Type" />
        <fwb-input class="my-1 py-1" pattern="[0-9]*" type="number" v-model="recordPoint" label="Points" />
        <div class="my-5 flex justify-center">
          <fwb-button color="dark" type="submit">Add Record</fwb-button>
        </div>
      </form>
      <div v-else>
        <p class="text-center">
          <!-- human readable time in minutes or hours -->
          Duration: {{ Math.round((timeGameEnded - timeGameStarted) / 60000) }} minutes
        </p>
        <div class="my-5 flex justify-center">
          <fwb-button color="red" @click="endGame">End Game</fwb-button>
        </div>
      </div>

      <div class="mt-4">
        <h1 class="text-xl">Records</h1>
        <table class="w-full text-sm text-left rtl:text-right text-gray-500" aria-label="Game Records">
          <thead class="my-10 text-xs text-gray-700 uppercase bg-gray-300">
            <tr>
              <th scope="col" class="px-6 py-3">Type</th>
              <th scope="col" class="px-6 py-3">Points</th>
            </tr>
          </thead>
          <tbody>
            <tr
              v-for="record in records"
              class="border-b"
              :class="{
                'bg-white': record.type != 'Round',
                'bg-green-300': record.type == 'Round' && record.point > 0,
                ' bg-red-300': record.type == 'Round' && record.point < 0,
                ' bg-blue-300': record.type == 'Round' && record.point == 0,
              }"
            >
              <th scope="row" class="px-6 py-4 font-medium text-gray-900 whitespace-nowrap dark:text-white">
                {{ record.type }}
              </th>
              <td class="px-6 py-4">
                {{ record.point }}
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
    <div v-else>
      <fwb-select v-model="GameType" :options="GameTypes" label="Select a Game Type" />
      <fwb-toggle class="mt-4" v-model="Teaming" label="Teaming" />

      <div v-if="!Teaming">
        <label for="Players">Team 1</label>
        <fwb-input class="my-1 py-1" v-model="TeamPlayers[0]" placeholder="Player 1" size="sm" />
        <fwb-input class="my-1 py-1" v-model="TeamPlayers[1]" placeholder="Player 2" size="sm" />
        <label for="Players">Team 2</label>
        <fwb-input class="my-1 py-1" v-model="TeamPlayers[2]" placeholder="Player 3" size="sm" />
        <fwb-input class="my-1 py-1" v-model="TeamPlayers[3]" placeholder="Player 4" size="sm" />
      </div>

      <div class="text-center">
        <fwb-button color="dark" @click="startGame">Start A Game</fwb-button>
      </div>
    </div>
  </main>
</template>

<style scoped>
.read-the-docs {
  color: #888;
}
</style>
