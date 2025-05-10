<template>
  <div class="mx-auto w-fit mt-2 flex flex-col">
    <div class="bg-gray-100 py-3 px-2 flex justify-between gap-2">
      <input
        :model-value="searchQuery"
        type="text"
        placeholder="Filter by username"
        class="px-1 border-gray-300 rounded-md bg-white focus:outline-none focus:ring-2 focus:ring-gray-700"
        @input="(e) => debounceSeachQuery(e.target?.value || '')"
      />
      <select
        v-model="maxNumUsers"
        @change="initializeUsers()"
        class="px-1 border-gray-300 rounded-md bg-white focus:outline-none focus:ring-2 focus:ring-gray-700"
      >
        <option :value="100">100</option>
        <option :value="200">200</option>
        <option :value="300">300</option>
        <option :value="400">400</option>
        <option :value="500">500</option>
      </select>
      <button
        class="bg-blue-500 text-white px-2 rounded hover:bg-blue-600"
        @click="initializeUsers()"
      >
        Generate New Users
      </button>
    </div>
    <table>
      <thead class="bg-gray-100 sticky top-0">
        <tr>
          <th class="px-4 py-2 text-left text-sm font-semibold text-gray-700">
            #
          </th>
          <TableHeader :header="TableHeaderTitle.RANK" />
          <TableHeader
            :header="TableHeaderTitle.USERNAME"
            :sort-asc="TableHeaderTitle.USERNAME === sortKey ? sortAsc : null"
            has-sorting
            @asc="(sortKey) => changeTableSorting(sortKey)"
          />
          <TableHeader
            :header="TableHeaderTitle.SCORE"
            :sort-asc="TableHeaderTitle.SCORE === sortKey ? sortAsc : null"
            has-sorting
            @asc="(sortKey) => changeTableSorting(sortKey)"
          />
          <th
            class="px-4 py-2 text-left text-sm font-semibold text-gray-700"
          ></th>
        </tr>
      </thead>
      <tbody class="bg-white divide-y divide-gray-200">
        <TableRow
          v-for="(user, index) in displayedUsers"
          :key="index"
          :id="user.id"
          :index="index + 1"
          :rank="user.rank"
          :previous-rank="user.previousRank"
          :user-name="user.userName"
          :score="user.score"
          :points-gained="user.pointsGained"
          @change-score="({ id, points }) => changeScore(id, points)"
        />
      </tbody>
    </table>
  </div>
</template>

<script setup lang="ts">
import TableRow from "./components/TableRow.vue";
import TableHeader from "./components/TableHeader.vue";
import { TableHeaderTitle } from "./types/Table";
import { generateNames } from "./utils/nameGenerator";
import { ref, onMounted, onBeforeUnmount, computed } from "vue";
import type { User } from "./types/User";

/* Initialization */
const initialUsers = ref<User[]>([]);
const maxNumUsers = ref<number>(100);
const displayedUsers = computed(() => {
  let userList = [...initialUsers.value];
  if (searchQuery.value) {
    userList = userList.filter((user) =>
      user.userName.toLowerCase().startsWith(searchQuery.value)
    );
  }

  if (sortAsc.value !== false || sortKey.value !== TableHeaderTitle.SCORE) {
    userList = changeSortBy(userList, sortKey.value, sortAsc.value);
  }

  return userList;
});

onMounted(() => {
  initializeUsers();

  // Start automatic score giver function for ever 10 second
  autoScoreTimer.value = setInterval(() => {
    addRandomScores();
  }, 10000);
});

function initializeUsers() {
  let usersList = generateNames(maxNumUsers.value);
  let rankedUserList = applyRankings(usersList, true);
  initialUsers.value = rankedUserList;
}

/* Sorting based on score and username */
const sortKey = ref<TableHeaderTitle>(TableHeaderTitle.SCORE);
const sortAsc = ref<boolean>(false);

function changeTableSorting(key: TableHeaderTitle): void {
  if (sortKey.value === key) {
    sortAsc.value = !sortAsc.value;
  } else {
    sortKey.value = key;
    sortAsc.value = true;
  }
}

function changeSortBy(
  userList: User[],
  sortByKey: string,
  sortAsc: boolean
): User[] {
  if (sortByKey === TableHeaderTitle.SCORE) {
    // Firstly sort by score. If the scores equal the same, then sort by username
    return userList.sort((a, b) => {
      const scoreDiff = sortAsc ? a.score - b.score : b.score - a.score;
      if (scoreDiff !== 0) return scoreDiff;
      return a.userName.localeCompare(b.userName);
    });
  }
  if (sortByKey === TableHeaderTitle.USERNAME) {
    return userList.sort((a, b) => {
      return sortAsc
        ? a.userName.localeCompare(b.userName)
        : b.userName.localeCompare(a.userName);
    });
  }
  return userList;
}

/* Score handling logic */
function changeScore(id: number, points: number): void {
  const user = initialUsers.value.find((user) => user.id === id);
  if (user) {
    user.score = Math.max(0, user.score + points);
    user.pointsGained = points;
    applyRankings(initialUsers.value);
  }
}

function applyRankings(usersList: User[], inititalApply = false): User[] {
  if (usersList.length) {
    usersList = changeSortBy(usersList, TableHeaderTitle.SCORE, false);
    let newRank = 1;
    let lastScore = usersList[0].score;

    for (let user of usersList) {
      if (user.score !== lastScore) {
        newRank++;
        lastScore = user.score;
      }

      inititalApply
        ? (user.previousRank = newRank)
        : (user.previousRank = user.rank);

      user.rank = newRank;
    }
  }
  return usersList;
}

/* Automatic score update logic */
const autoScoreTimer = ref<number | null>(null);

onBeforeUnmount(() => {
  autoScoreTimer.value = null;
});

function addRandomScores(): void {
  initialUsers.value.map((user) => (user.pointsGained = 0));

  // Randomly change scores for 10 users
  for (let i = 0; i < 10; i++) {
    const index = Math.floor(Math.random() * initialUsers.value.length);
    const points = Math.floor(Math.random() * 21) - 10; // -10 to +10
    initialUsers.value[index].pointsGained = points;
    initialUsers.value[index].score = Math.max(
      0,
      initialUsers.value[index].score + points
    );
  }
  applyRankings(initialUsers.value);
}

/* Filtering based on search query */
const searchQuery = ref<string>("");
let debounceTimeout: number | null = null;

function debounceSeachQuery(queryValue: string): void {
  if (debounceTimeout !== null) {
    clearTimeout(debounceTimeout);
  }
  debounceTimeout = setTimeout(() => {
    searchQuery.value = queryValue;
  }, 500);
}
</script>

<style scoped></style>
