<script setup>
// по мотивам https://www.brainbell.com/javascript/making-resizable-table-js.html

import { ref, reactive, watch } from "vue";
import { Notify } from "quasar";

let curCol, //текщий столбец
  nxtCol, //следующий столбец
  pageX, //стартовая позиция по горизонтале
  curColWidth, //ширина текщего столбца
  nxtColWidth; //ширина следующего столбца

const enableResizible = ref(false); //Флаг включения или нет ресайзинга

// Параметры столбцов
let columns = reactive([
  {
    name: "desc",
    required: true,
    label: "Dessert (100g serving)",
    align: "left",
    field: (row) => row.name,
    format: (val) => `${val}`,
    sortable: true,
  },
  {
    name: "calories",
    align: "center",
    label: "Calories",
    field: "calories",
    sortable: true,
  },
  { name: "fat", label: "Fat (g)", field: "fat", sortable: true },
  { name: "carbs", label: "Carbs (g)", field: "carbs" },
  { name: "protein", label: "Protein (g)", field: "protein" },
  { name: "sodium", label: "Sodium (mg)", field: "sodium" },
  {
    name: "calcium",
    label: "Calcium (%)",
    field: "calcium",
    sortable: true,
    sort: (a, b) => parseInt(a, 10) - parseInt(b, 10),
  },
  {
    name: "iron",
    label: "Iron (%)",
    field: "iron",
    sortable: true,
    sort: (a, b) => parseInt(a, 10) - parseInt(b, 10),
  },
]);
let sortableColumns = []; //список имен столбцов, которые сортируются

// данные для таблицы
const data = reactive([
  {
    name: "Frozen Yogurt",
    calories: 159,
    fat: 6.0,
    carbs: 24,
    protein: 4.0,
    sodium: 87,
    calcium: "14%",
    iron: "1%",
  },
  {
    name: "Ice cream sandwich",
    calories: 237,
    fat: 9.0,
    carbs: 37,
    protein: 4.3,
    sodium: 129,
    calcium: "8%",
    iron: "1%",
  },
  {
    name: "Eclair",
    calories: 262,
    fat: 16.0,
    carbs: 23,
    protein: 6.0,
    sodium: 337,
    calcium: "6%",
    iron: "7%",
  },
  {
    name: "Cupcake",
    calories: 305,
    fat: 3.7,
    carbs: 67,
    protein: 4.3,
    sodium: 413,
    calcium: "3%",
    iron: "8%",
  },
  {
    name: "Gingerbread",
    calories: 356,
    fat: 16.0,
    carbs: 49,
    protein: 3.9,
    sodium: 327,
    calcium: "7%",
    iron: "16%",
  },
  {
    name: "Jelly bean",
    calories: 375,
    fat: 0.0,
    carbs: 94,
    protein: 0.0,
    sodium: 50,
    calcium: "0%",
    iron: "0%",
  },
  {
    name: "Lollipop",
    calories: 392,
    fat: 0.2,
    carbs: 98,
    protein: 0,
    sodium: 38,
    calcium: "0%",
    iron: "2%",
  },
  {
    name: "Honeycomb",
    calories: 408,
    fat: 3.2,
    carbs: 87,
    protein: 6.5,
    sodium: 562,
    calcium: "0%",
    iron: "45%",
  },
  {
    name: "Donut",
    calories: 452,
    fat: 25.0,
    carbs: 51,
    protein: 4.9,
    sodium: 326,
    calcium: "2%",
    iron: "22%",
  },
  {
    name: "KitKat",
    calories: 518,
    fat: 26.0,
    carbs: 65,
    protein: 7,
    sodium: 54,
    calcium: "12%",
    iron: "6%",
  },
]);

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
  columns.forEach((el) => (el.sortable = sortableColumns.includes(el.name)));
}
// выключаем сортировку на столбцах
function unsortableTable() {
  sortableColumns = [];
  columns.forEach((el) => {
    if (!(el.sortable === undefined) && el.sortable) {
      sortableColumns.push(el.name); //запоминаяем столбцы, которые имели сортировку
    }
  });
  columns = columns.map((e) => {
    if (e.sortable) e.sortable = false;
    return e;
  });
}
// создаем вертикальную линию, за которую будем двигать
function createDiv(h) {
  var div = document.createElement("div");
  div.style.height = `${h}px`;
  div.className = "resizibleDiv";
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
// выключаем изменение размеров
function unresizeableTable() {
  document
    .querySelector(".q-table thead")
    .removeEventListener("dblclick", resetWith);
  document.removeEventListener("mousemove", mouseMove);
  document.removeEventListener("mouseup", mouseUp);
  sortableTable();
  const cols = document.querySelectorAll(".q-table th");

  for (var i = 0; i < cols.length; i++) {
    const divs = cols[i].getElementsByTagName("div");
    for (var j = 0; j < divs.length; j++) {
      unsetListeners(divs[j]);
      cols[i].removeChild(divs[j]);
    }
  }
  showNotif("Выключено изменение ширины столбцов", "red");
}
// включаем режим изменения столбцов
function resizeableTable() {
  // выключаем сортировку
  unsortableTable();
  // подключаем слушатели на заголовок таблицы
  document
    .querySelector(".q-table thead")
    .addEventListener("dblclick", resetWith);
  document.addEventListener("mousemove", mouseMove);

  const table = document.querySelectorAll(".q-table"),
    cols = document.querySelectorAll(".q-table th");
  // перебираем все столбцы и добавляем вертикальную линию для управления мышью
  for (var i = 0; i < cols.length; i++) {
    var div = createDiv(table[0].clientHeight);
    cols[i].appendChild(div);
    cols[i].style.position = "relative";
  }
  showNotif("Включено изменение ширины столбцов", "green");
}
// сброс внесенных изменений ширины столбцов
function resetWith() {
  const cols = document.querySelectorAll(".q-table th");
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

// вызываем функцию вкл или выкл режима, в зависимости от состнояния флага
watch(enableResizible, (value) => {
  if (value) resizeableTable();
  else unresizeableTable();
});
</script>
<template>
  <div class="q-pa-md">
    <div>
      <q-btn-toggle
        v-model="enableResizible"
        no-caps
        toggle-color="primary"
        color="white"
        text-color="black"
        :options="[
          { label: 'Unresizible', value: false },
          { label: 'Resizible', value: true },
        ]"
      />
    </div>
    <q-table
      width="500"
      ref="mytable"
      title="Treats"
      :rows="data"
      :columns="columns"
      row-key="name"
    ></q-table>
  </div>
</template>
<style>
.resizibleDiv {
  top: 0;
  right: 0;
  width: 3px;
  position: absolute;
  cursor: col-resize;
  user-select: none;
  background-color: silver;
  z-index: 1000;
}
.resizibleDiv:hover {
  background-color: blue !important;
}
</style>
