<script setup>
// по мотивам https://www.brainbell.com/javascript/making-resizable-table-js.html

import { watch, computed, reactive, onMounted } from "vue";

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
  resizable: {
    type: Boolean,
    default: false,
  },
  title: String,
  width: String,
});

const uniqueId =
  Date.now().toString(36) + Math.random().toString(36).substring(2);

let table;
let sortableColumns;
let MyColumns = reactive(props.columns);
onMounted(() => {
  // получаем указатель на нашу таблицу в DOM
  table = document.getElementsByClassName(uniqueId)[0].firstChild;
});

let curCol, //текущий столбец
  nxtCol, //следующий столбец
  pageX, //стартовая позиция по горизонтали
  curColWidth, //ширина текущего столбца
  nxtColWidth; //ширина следующего столбца

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

// двигаем мышкой, вычисляем координаты, изменяем ширину столбцов
function mouseMove(e) {
  if (curCol) {
    var diffX = e.pageX - pageX;

    if (nxtCol) nxtCol.style.width = nxtColWidth - diffX + "px";

    curCol.style.width = curColWidth + diffX + "px";
  }
}

// создаем вертикальную линию, за которую будем двигать
function createDiv() {
  var div = document.createElement("div");
  div.style.height = `${table.clientHeight}px`;
  div.className = "columnSelector";
  setListeners(div);
  return div;
}
// устанавливаем слушатели событий на нажатие кнопки мыши
function setListeners(div) {
  div.addEventListener("mousedown", mouseDown, false);
  div.addEventListener("mouseup", mouseUp, false);
}
// убираем слушатели событий
function unsetListeners(div) {
  div.removeEventListener("mousedown", mouseDown, false);
  div.removeEventListener("mouseup", mouseUp, false);
}
// включаем сортировку столбцов
function sortableTable() {
  MyColumns.forEach((el) => (el.sortable = sortableColumns.includes(el.name)));
}
// выключаем сортировку столбцов
function unsortableTable() {
  sortableColumns = [];
  props.columns.forEach((el) => {
    if (!(el.sortable === undefined) && el.sortable) {
      sortableColumns.push(el.name); //запоминаяем столбцы, которые имели сортировку
    }
  });
  MyColumns = props.columns.map((e) => {
    if (e.sortable) e.sortable = false;
    return e;
  });
}

// включаем режим изменения столбцов
function resizableTable() {
  unsortableTable(); //отключаем сортировку
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
    var div = createDiv();
    cols[i].appendChild(div);
    cols[i].style.position = "relative";
  }
}

// выключаем изменение размеров
function unresizableTable() {
  sortableTable(); //возвращаем сортировку
  table
    .getElementsByTagName("thead")[0]
    .removeEventListener("dblclick", resetWith);

  document.removeEventListener("mousemove", mouseMove);

  const cols = table.getElementsByTagName("th");

  for (var i = 0; i < cols.length; i++) {
    const divs = cols[i].getElementsByClassName("columnSelector");
    for (var j = 0; j < divs.length; j++) {
      unsetListeners(divs[j]);
      cols[i].removeChild(divs[j]);
    }
  }
}

// сброс внесенных изменений ширины столбцов
function resetWith() {
  const cols = table.getElementsByTagName("th");
  for (var i = 0; i < cols.length; i++) {
    cols[i].style.width = "";
  }
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

const resizable = computed(() => props.resizable);

// вызываем функцию вкл или выкл режима, в зависимости от состнояния флага
watch(resizable, (value) => {
  if (value) resizableTable();
  else unresizableTable();
});
</script>

<template>
  <q-table
    :width="width"
    :title="title"
    :rows="rows"
    :columns="MyColumns"
    row-key="name"
    :table-class="uniqueId"
  ></q-table>
</template>

<style>
.columnSelector {
  top: 0;
  right: 0;
  display: block;
  width: 7px;
  position: absolute;
  cursor: col-resize;
  user-select: none;
  border-color: silver;
  border-right: 1px dashed;
  z-index: 1000;
}
.columnSelector:hover {
  border-color: blue !important;
}
</style>
