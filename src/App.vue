<script setup>
import { ref, onMounted } from "vue";

const appId = ref("");
const title = ref("");
const sortCondition = ref("standard");
const searchResults = ref(null);

onMounted(() => {
  if (localStorage.getItem("APP_ID")) {
    appId.value = localStorage.getItem("APP_ID");
  }
});

async function search() {
  searchResults.value = null;

  if (!validate()) return;

  const res = await fetch(generateURL());
  searchResults.value = await res.json();

  localStorage.setItem("APP_ID", appId.value);
}

function validate() {
  if (!appId.value || !title.value || !sortCondition.value) {
    return false;
  }
  return true;
}

function generateURL() {
  const searchURL = new URL(
    "https://app.rakuten.co.jp/services/api/BooksBook/Search/20170404"
  );
  searchURL.searchParams.append("applicationId", appId.value);
  searchURL.searchParams.append("title", title.value);
  searchURL.searchParams.append("sort", sortCondition.value);
  searchURL.searchParams.append("formatVersion", "2");
  return searchURL;
}
</script>

<template>
  <main class="container mx-auto p-6">
    <h1 class="text-xl font-semibold">書籍検索</h1>

    <div class="mt-2">楽天の書籍検索APIを利用して検索できます。</div>
    <div class="text-sm text-gray-500">
      ※検索には<a
        class="text-blue-400"
        href="https://webservice.rakuten.co.jp/guide"
        >アプリID</a
      >が必要です。
    </div>
    <form @submit.prevent="search" class="my-4">
      <fieldset>
        <label for="appId">アプリID</label>
        <input v-model="appId" id="appId" type="text" />
      </fieldset>
      <fieldset class="mt-2">
        <label for="title">タイトル</label>
        <input v-model="title" id="title" type="text" />
      </fieldset>
      <fieldset class="mt-2">
        <input
          type="radio"
          id="standard"
          value="standard"
          v-model="sortCondition"
        />
        <label for="standard">標準</label>
        <input
          type="radio"
          id="sales"
          value="sales"
          v-model="sortCondition"
          class="ml-2"
        />
        <label for="sales">売れている順</label>
        <input
          type="radio"
          id="-releaseDate"
          value="-releaseDate"
          v-model="sortCondition"
          class="ml-2"
        />
        <label for="-releaseDate">発売日が新しい順</label>
      </fieldset>

      <button class="btn">検索</button>
    </form>

    <p v-if="!searchResults || searchResults.Items.length == 0">
      一致する検索結果が見つかりませんでした。
    </p>
    <div v-else class="mt-2 overflow-auto shadow-md">
      <table class="table-fixed min-w-full">
        <thead class="bg-gray-100">
          <tr>
            <th class="px-2"></th>
            <th>タイトル / 著者</th>
            <th>発売日</th>
            <th>ISBN</th>
            <th>読書メーター</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="item in searchResults.Items" :key="item.itemUrl">
            <td class="pr-0">
              <img :src="item.smallImageUrl" :alt="item.title" class="h-12" />
            </td>
            <td>
              <p class="font-semibold text-gray-700">{{ item.title }}</p>
              <p class="mt-1 text-xs">{{ item.author }}</p>
            </td>
            <td>{{ item.salesDate }}</td>
            <td>{{ item.isbn }}</td>
            <td>
              <a
                :href="`https://bookmeter.com/search?keyword=${item.isbn}`"
                class="text-blue-600 font-semibold"
                target="_blank"
                rel="noopener noreferrer"
                >開く</a
              >
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </main>
</template>

<style>
.btn {
  @apply bg-blue-500 hover:bg-blue-700 text-white font-bold shadow border rounded text-xs py-2 px-6 mt-2;
}

input[type="text"] {
  @apply shadow border rounded text-sm py-1 px-2 ml-2 w-72;
}

a {
  @apply hover:underline;
}

th {
  @apply py-3 px-6 text-xs font-semibold tracking-wider text-left text-gray-700;
}
tr {
  @apply border-b hover:bg-gray-50;
}
td {
  @apply py-2 px-6 text-sm text-gray-500;
}
</style>
