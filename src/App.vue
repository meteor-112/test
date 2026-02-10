<!-- 這裡寫js -->
<!-- 加上 setup 後，就不需要寫return -->
<script setup>
import { ref, onMounted } from "vue";
import Swal from "sweetalert2";

let data = ref([]); //空陣列
const nameInput = ref(""); //輸入框預設清空
const levelInput = ref("none"); //輸入框預設清空
const priorityInput = ref("none"); //輸入框預設清空
const remarkInput = ref(""); //輸入框預設清空
const hasError = ref(false); //輸入框提醒預設不出現
const getTasks = JSON.parse(localStorage.getItem("todoList")); //取得待辦清單

//載入畫面時顯示既有的清單
onMounted(() => get());

//點擊送出按鈕時
const submit = () => {
  //未輸入文字時不送出,並跳出紅字提示
  if (
    nameInput.value.trim() === "" ||
    levelInput.value === "none" ||
    priorityInput.value === "none"
  ) {
    return (hasError.value = true);
  } else {
    //加入陣列
    data.value.push({
      name: nameInput.value.trim(), //名稱
      level: Number(levelInput.value), //難度(數字)
      priority: Number(priorityInput.value), //優先度(數字)
      remark: remarkInput.value.trim(), //備註
      status: false, //未完成
    });
    // console.log(data.value);
    close(); //送出後，關閉視窗並清空input
    sortByStatus(); //送出後進行一次排序，確保新增事項在已完成事項上方
    saveTasks(); //存入清單
  }
};

//點擊「移除」從陣列中刪除事項
const deleteItem = (index) => {
  data.value.splice(index, 1);
  // console.log(data.value);
  saveTasks(); //將變化存入清單
};

//待辦事項-完成狀態切換
const toggleStatus = (item) => {
  if (item.status === false) {
    item.status = true;

    // 彈跳視窗
    let timerInterval;
    Swal.fire({
      title: "Good Job!",
      timer: 1500,
      timerProgressBar: true,
      showConfirmButton: false,
      icon: "success",
      didOpen: () => {
        const timer = Swal.getPopup().querySelector("b");
        timerInterval = setInterval(() => {
          timer.textContent = `${Swal.getTimerLeft()}`;
        }, 100);
      },
      willClose: () => {
        clearInterval(timerInterval);
      },
    });
  } else {
    item.status = false;
  }
  sortByStatus(); //進行一次排序
  saveTasks(); //將變化存入清單
};

//排序
const sortByStatus = () => {
  data.value.sort((a, b) => {
    if (a.status !== b.status) {
      return a.status - b.status; // status false(未完成)在前，true(完成)在後
    } else {
      return b.priority - a.priority; // status 相同時，按 優先度 排序
    }
  });
};

//視窗默認關閉
const modal = ref(false);
//點擊新增按鈕，開啟視窗
const addItem = () => (modal.value = true);
//點擊關閉按鈕，關閉視窗，並清空input
const close = () => {
  modal.value = false;
  nameInput.value = "";
  levelInput.value = "none";
  priorityInput.value = "none";
  remarkInput.value = "";
  hasError.value = false;
};
// 儲存清單於local
const saveTasks = () => {
  localStorage.setItem("todoList", JSON.stringify(data.value));
};
//檢查有無現存清單，有則載入
const get = () => {
  if (!getTasks || getTasks.length === 0) {
    return;
  } else {
    data.value = getTasks;
    // console.log(data.value);
  }
};
</script>

<!-- html -->
<template>
  <div class="w-full min-h-screen bg-gray">
    <main class="w-[90%] sm:w-[80%] lg:w-[60%] flex-col mx-auto py-4">
      <!-- 上方標題與新增按鈕 -->
      <header
        class="flex flex-row justify-between items-center px-3 py-2 mb-4 rounded-t-md bg-navdark border-b-5 border-indigo-500 shadow-md shadow-gray-500"
      >
        <div class="text-4xl text-white">待辦清單</div>
        <button
          type="button"
          class="bg-gray-200 hover:bg-gray"
          @click="addItem"
        >
          新增
        </button>
      </header>
      <!-- 清單內容 -->
      <section class="flex flex-col gap-4">
        <!-- 待辦事項 -->
        <div
          class="relative flex flex-col lg:flex-row gap-4 justify-between px-3 sm:px-6 pt-4 pb-3 lg:py-4 shadow-md shadow-gray-500 rounded-md bg-[#f4f4f4]"
          v-for="(item, index) in data"
          :key="index"
        >
          <!-- 左側資訊 -->
          <div class="flex flex-col gap-3 flex-1 min-w-0">
            <div class="text-2xl">
              名稱：
              <span class="break-words">{{ item.name }}</span>
            </div>
            <div class="grid lg:grid-cols-2 gap-3 w-full text-2xl">
              <div>
                緊急度：
                <span v-if="item.priority === 1" class="tag bg-green-400"
                  >不緊急</span
                >
                <span v-else-if="item.priority === 2" class="tag bg-yellow"
                  >一般</span
                >
                <span v-else-if="item.priority === 3" class="tag bg-red-500"
                  >緊急</span
                >
              </div>
              <div>
                重要度：
                <span v-if="item.level === 1" class="tag bg-green-400"
                  >不重要</span
                >
                <span v-else-if="item.level === 2" class="tag bg-yellow"
                  >一般</span
                >
                <span v-else-if="item.level === 3" class="tag bg-red-500"
                  >重要</span
                >
              </div>
            </div>
            <div v-show="item.remark" class="text-2xl">
              備註：
              <span class="break-words">{{ item.remark }}</span>
            </div>
          </div>
          <!-- 右側按鈕區 -->
          <div class="buttons grid grid-cols-2 lg:flex lg:flex-col lg:justify-center gap-3 z-10">
            <button
              type="button"
              class="bg-primarycolor hover:bg-sky-600 text-white"
              @click="toggleStatus(item)"
            >
              {{ item.status ? "回復" : "完成" }}
            </button>
            <button
              type="button"
              class="bg-dark hover:bg-gray-600 text-white"
              @click="deleteItem(index)"
            >
              移除
            </button>
          </div>
          <!-- 遮罩（完成時顯示） -->
          <div
            v-show="item.status"
            class="absolute inset-0 bg-gray-500/60 flex items-center justify-center rounded-md cursor-default"
          >
            <span class="text-white text-2xl font-bold bg-dark rounded-md px-3 py-2">
              已完成
            </span>
          </div>
        </div>
      </section>

      <!-- 陣列無內容時(長度為0)，顯示提示 data.length===0 -->
      <!-- data.length=0 是false 等同於 !data.length-->
      <div
        v-show="!data.length"
        class="flex flex-col items-center absolute top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2 text-gray-800"
      >
        <i class="bi bi-cup-hot-fill text-5xl mb-2"></i>
        <div class="text-2xl font-bold text-center">當前沒有待辦事項</div>
      </div>

      <!-- 輸入視窗 -->
      <div v-show="modal" class="modal-on text-2xl">
        <div
          class="modal-content w-[80%] lg:w-150 bg-gray p-5 rounded-lg flex flex-col gap-3"
        >
          <!-- 待辦事項 -->
          <label for="name" class="space-y-1">
            <p>待辦事項：</p>
            <input
              type="text"
              name="name"
              id="name"
              class="w-full bg-amber-50"
              placeholder="請輸入待辦事項"
              v-model="nameInput"
            />
          </label>

          <!-- 緊急度 -->
          <label for="priority" class="space-y-1">
            <p>緊急度：</p>
            <select
              name="priority"
              id="priority"
              class="w-full h-10 bg-amber-50"
              v-model="priorityInput"
            >
              <option value="none" hidden>--請選擇--</option>
              <option value="1">不緊急</option>
              <option value="2">一般</option>
              <option value="3">緊急</option>
            </select>
          </label>

          <!-- 重要度 -->
          <label for="level" class="space-y-1">
            <p>重要度：</p>
            <select
              name="level"
              id="level"
              class="w-full h-10 bg-amber-50"
              v-model="levelInput"
            >
              <option value="none" hidden>--請選擇--</option>
              <option value="1">不重要</option>
              <option value="2">一般</option>
              <option value="3">重要</option>
            </select>
          </label>
          <!-- 備註 -->
          <label for="remark" class="space-y-1">
            <p>備註：</p>
            <input
              type="text"
              name="remark"
              id="remark"
              class="w-full bg-amber-50"
              placeholder="請填寫備註(非必填)"
              v-model="remarkInput"
            />
          </label>
          <div v-if="hasError" class="text-red-500">請填寫所有必填欄位</div>
          <!-- 下方按鈕區(確認/取消) -->
          <div class="grid grid-cols-2 justify-center gap-4 mt-2">
            <button
              type="button"
              class="bg-primarycolor hover:bg-sky-600 text-white"
              @click="submit"
            >
              新增
            </button>
            <button
              type=" button"
              class="bg-dark hover:bg-gray-600 text-white"
              @click="close"
            >
              取消
            </button>
          </div>
        </div>
      </div>
    </main>
  </div>
</template>

<!-- css -->
<!-- 加上scoped後，這裡寫的css 只會屬於這支檔案，不會影響到其他檔案 -->
<style scoped>

.modal-on {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 20;
}

input {
  padding: 4px 8px;
}
select {
  padding: 4px;
}

span.tag {
  color: white;
  padding: 4px 10px;
  border-radius: 18px;
}
</style>
