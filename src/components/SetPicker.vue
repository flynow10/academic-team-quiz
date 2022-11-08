<script setup lang="ts">
import { computed, reactive, ref } from "vue";
import { sets } from "../../public/data/sets";

const open = ref(false);
const displayNames: { [key: string]: string } = {
  "american-government-and-economics": "American Government and Economics",
  "american-history": "American History",
  "american-literature": "American Literature",
  "fine-arts": "Fine Arts",
  geography: "Geography",
  "life-science": "Life Science",
  math: "Math",
  "physical-science": "Physical Science",
  "world-history": "World History",
  "world-literature": "World Literature",
};

const questionGroups = reactive({
  hard: true,
  all: true,
});

const setList = reactive(
  [...new Set(sets.map((set) => set.replace(/-\(.*\)/, "")))].map((set) => ({
    id: set,
    displayName: displayNames[set],
    selected: true,
  }))
);
const selectedSets = computed(() => {
  var finalSetList: string[] = [];
  for (const set of setList) {
    if (set.selected) {
      if (questionGroups.hard) {
        finalSetList.push(set.id + "-(hard)");
      }
      if (questionGroups.all) {
        if (set.id === "american-history") {
          finalSetList.push(set.id + "-(old)");
        } else {
          finalSetList.push(set.id + "-(all)");
        }
      }
    }
  }
  return finalSetList;
});

const toggle = (value: boolean | null = null) => {
  if (value === null) {
    open.value = !open.value;
  } else {
    open.value = value;
  }
};

const emit = defineEmits(["update"]);

defineExpose({
  selectedSets,
  toggle,
});
</script>
<template>
  <div class="sidebar" :class="open ? 'open' : 'closed'" @keypress.escape="open = false" @click.exact="open = !open">
    <!--An arrow pointing in the direction of next movement-->
    <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24"
      :class="open ? 'arrow arrow-open' : 'arrow arrow-closed'">
      <path d="M0 0h24v24H0z" fill="none" />
      <path d="M16.59 8.59L12 13.17 7.41 8.59 6 10l6 6 6-6z" />
    </svg>
    <div class="sidebar-sticky" @click.stop="">
      <h3>Configuration</h3>
      <div class="question-group group">
        <span class="group-name">Question Types</span>
        <div class="group-item" v-for="(questionGroup, key) in questionGroups" :key="key">
          <input type="checkbox" :id="key" v-model="questionGroups[key]" @change="emit('update')" :disabled="
            questionGroup &&
            (questionGroups.all ? !questionGroups.hard : questionGroups.hard)
          " />
          <label :for="key">{{
              key[0].toUpperCase() + key.substring(1)
          }}</label>
        </div>
      </div>
      <div class="set-list group">
        <span class="group-name">Question Sets</span>
        <div class="group-item" v-for="set in setList" :key="set.id">
          <input type="checkbox" :id="set.id" v-model="set.selected" @change="emit('update')" :disabled="
            set.selected &&
            setList.reduce(
              (count, set) => count + (set.selected ? 1 : 0),
              0
            ) === 1
          " />
          <label :for="set.id">{{ set.displayName }}</label>
        </div>
      </div>
    </div>
  </div>
</template>