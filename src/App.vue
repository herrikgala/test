<script setup>
import { watch, ref, onMounted } from "vue";
import { debounce } from "@/utils/debounce.js";

const inputValue = ref("");
const responseArray = ref([]);
const activeIndex = ref(-1);
const requestTrial = ref(0);

async function handleChange(e) {
  inputValue.value = e.target.value;
  activeIndex.value = -1;
  if (inputValue.value) {
    debouncedRequest();
  } else {
    responseArray.value = [];
  }
}

const debouncedRequest = debounce(request, 250);


async function request() {
  const resRaw = await fetch(
    `https://rickandmortyapi.com/api/character?name=${inputValue.value}`
  );
  if(!resRaw.ok){
    requestTrial.value++
    if(requestTrial.value >= 3){
      requestTrial.value=0
      throw new Error('You request trials reached')
    }
    debouncedRequest();
  } else{
    requestTrial.value=0
  }
  const resJson = await resRaw.json();
  const results = resJson.results || [];
  responseArray.value = results.map((obj) => {
    return {
      ...obj,
      name: obj.name.replaceAll(
        inputValue.value,
        `<span class="yellow">${inputValue.value}</span>`
      ),
    };
  });
}

function extractContent(s){
  var span = document.createElement('span');
  span.innerHTML = s;
  return span.textContent || span.innerText;
}

function handleClick(obj, index) {
  inputValue.value = extractContent(obj.name);
  responseArray.value = [];
  activeIndex.value = index;
}

function handleNavigation({ code }) {
  if (code === "ArrowUp") {
    activeIndex.value--;
  } else if (code === "ArrowDown") {
    activeIndex.value++;
  }

  // acitveIndex Exsist
  if (activeIndex.value > -1 && code === "Enter") {
    handleClick(responseArray.value[activeIndex.value], activeIndex.value);
  }
}

// 1. Сделать поиск с подсказкой (suggest аля google)
//    - API для результатов поиска - https://rickandmortyapi.com/api/character;
//    - фильтровать можно по query "name";
//    - «Smith» — слово для поиска;
//    - cтилизовать поиск на своё усмотрение.
// 2. Подсказки:
//    - мы можем выбрать один из результатов;
//    - реализовать навигацию по списку с помощью таба.
//    - реализовать навигацию по списку с помощью клавиатуры (стрелки вверх/вниз);
//    - как только мы сделали выбор результата поиска или при событии input список должен исчезнуть;
//    - выбранный результат должен появиться в поле ввода.
// 3*. Подсветка результатов в списке:
//    - введенныйе символы в input должны выделяться жирным (<b></b>) в списке с результатами поиска
// 4*. Сделать запрос с повторами
//    - два повтора с интервалом 250 мс. после третьего показывать ошибку
</script>

<template>
  <div class="main">
    <div class="search">
      <input
        type="text"
        :value="inputValue"
        @input="handleChange"
        @keyup="handleNavigation"
      />
      <ul class="listWrapper">
        <li
          v-for="(obj, index) in responseArray"
          key="obj.id"
          @click="handleClick(obj, index)"
          :class="{ active: index === activeIndex }"
        >
          <span v-html="obj.name"></span>
          <!-- {{`${obj.status}` }} -->
        </li>
      </ul>
    </div>
    <!-- <button type="button">Найти!</button> -->
  </div>
</template>

<style>
.search {
  position: relative;
}

.active {
  background-color: tomato;
}

.yellow {
  background-color: yellow;
}

.listWrapper {
  padding: 0;
  position: absolute;
  list-style: none;
}

.listWrapper > li {
  padding: 0;
  cursor: pointer;
  width: max-content;
  padding: 5px 0;
}
</style>
