<script setup>
// по мотивам https://www.brainbell.com/javascript/making-resizable-table-js.html

import { watch, computed, reactive, onMounted } from "vue";
import { Notify } from "quasar";

const props = defineProps({
  rows: {
    type: Array,
    default: () => [],
    required: true,
  },
  columns: {
    type: Array,
    default: () => [],
    required: true,
  },
  resizible: {
    type: Boolean,
    default: false,
  },
  title: String,
  width: String,
});
// переопределяем массив для внутренней манипуляции
let mycolumns = reactive(props.columns);
const uniqueId =
  Date.now().toString(36) + Math.random().toString(36).substring(2);

let table;

onMounted(() => {
  // получаем указатель на нашу таблицу в DOM
  table = document.getElementsByClassName(uniqueId)[0].firstChild;
});

let curCol, //текщий столбец
  nxtCol, //следующий столбец
  pageX, //стартовая позиция по горизонтале
  curColWidth, //ширина текщего столбца
  nxtColWidth; //ширина следующего столбца

let sortableColumns = []; //список имен столбцов, которые сортируются

// отображение нотификатора
function showNotif(message, color, icon) {
  Notify.create({
    message,
    color,
    icon,
  });
}

// очищаем параметры при отжатии кнопки
function mouseUp(e) {
  curCol = nxtCol = pageX = nxtColWidth = curColWidth = undefined;
}
// инициализация стартовых данных при нажатии кнопки мыши
function mouseDown(e) {
  curCol = e.target.parentElement;
  nxtCol = curCol.nextElementSibling;
  pageX = e.pageX;

  var padding = paddingDiff(curCol);

  curColWidth = curCol.offsetWidth - padding;
  if (nxtCol) nxtColWidth = nxtCol.offsetWidth - padding;
}

// двигаем мышкой, вычисляем координаты, изменяем ширину столбца
function mouseMove(e) {
  if (curCol) {
    var diffX = e.pageX - pageX;

    if (nxtCol) nxtCol.style.width = nxtColWidth - diffX + "px";

    curCol.style.width = curColWidth + diffX + "px";
  }
}
// восстанавливаем сортировку
function sortableTable() {
  mycolumns.forEach((el) => (el.sortable = sortableColumns.includes(el.name)));
}
// выключаем сортировку на столбцах
function unsortableTable() {
  sortableColumns = [];
  mycolumns.forEach((el) => {
    if (!(el.sortable === undefined) && el.sortable) {
      sortableColumns.push(el.name); //запоминаяем столбцы, которые имели сортировку
    }
  });
  mycolumns = mycolumns.map((e) => {
    if (e.sortable) e.sortable = false;
    return e;
  });
}
// создаем вертикальную линию, за которую будем двигать
function createDiv(h) {
  var div = document.createElement("div");
  div.style.height = `${h}px`;
  div.className = "columnSelector";
  setListeners(div);
  return div;
}
// устанавливаем слушаетли событий на нажатие кнопки мыши
function setListeners(div) {
  div.addEventListener("mousedown", mouseDown, false);
  div.addEventListener("mouseup", mouseUp, false);
}
// убираем слушатели событий
function unsetListeners(div) {
  div.removeEventListener("mousedown", mouseDown, false);
  div.removeEventListener("mouseup", mouseUp, false);
}
// включаем режим изменения столбцов
function resizeableTable() {
  // выключаем сортировку
  unsortableTable();
  // подключаем слушателя на заголовок таблицы
  // функция обнуления изменений ширины

  table
    .getElementsByTagName("thead")[0]
    .addEventListener("dblclick", resetWith);

  // перемещение мышкой, непосредственно изменение столбцов
  document.addEventListener("mousemove", mouseMove);

  const cols = table.getElementsByTagName("th");
  // перебираем все столбцы и добавляем вертикальную линию для управления мышью

  for (var i = 0; i < cols.length; i++) {
    var div = createDiv(table.clientHeight);
    cols[i].appendChild(div);
    cols[i].style.position = "relative";
  }
  showNotif("Включено изменение ширины столбцов", "green");
}

// выключаем изменение размеров
function unresizeableTable() {
  table
    .getElementsByTagName("thead")[0]
    .removeEventListener("dblclick", resetWith);

  document.removeEventListener("mousemove", mouseMove);
  sortableTable();
  const cols = table.getElementsByTagName("th");

  for (var i = 0; i < cols.length; i++) {
    const divs = cols[i].getElementsByClassName("columnSelector");
    for (var j = 0; j < divs.length; j++) {
      unsetListeners(divs[j]);
      cols[i].removeChild(divs[j]);
    }
  }
  showNotif("Выключено изменение ширины столбцов", "red");
}

// сброс внесенных изменений ширины столбцов
function resetWith() {
  const cols = table.getElementsByTagName("th");
  for (var i = 0; i < cols.length; i++) {
    cols[i].style.width = "";
  }
  showNotif("Сброшены изменения ширины столбцов", "purple");
}

// вычисление разницы отступов соседних столбцов
function paddingDiff(col) {
  if (getStyleVal(col, "box-sizing") == "border-box") {
    return 0;
  }

  var padLeft = getStyleVal(col, "padding-left");
  var padRight = getStyleVal(col, "padding-right");
  return parseInt(padLeft) + parseInt(padRight);
}
// вычисляем реальные размеры
function getStyleVal(elm, css) {
  return window.getComputedStyle(elm, null).getPropertyValue(css);
}

const resizible = computed(() => props.resizible);

// вызываем функцию вкл или выкл режима, в зависимости от состнояния флага
watch(resizible, (value) => {
  if (value) resizeableTable();
  else unresizeableTable();
});
</script>

<template>
  <q-table
    :width="width"
    :title="title"
    :rows="rows"
    :columns="mycolumns"
    row-key="name"
    :table-class="uniqueId"
  ></q-table>
</template>

<style>
.columnSelector {
  top: 0;
  right: 0;
  width: 2px;
  position: absolute;
  cursor: col-resize;
  user-select: none;
  background-color: silver;
  z-index: 1000;
}
.columnSelector:hover {
  background-color: blue !important;
}
</style>
