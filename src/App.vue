<script setup lang="ts">
import { onMounted, ref, watch } from "vue";
import { GraphQLClient } from "graphql-request";

const collectionId = ref("");
const tokenId = ref("");
const errorMessage = ref("");

const client = new GraphQLClient("https://data.objkt.com/v3/graphql");

type Token = {
  name: string;
  supply: number;
  thumbnail_uri: string;
  holders: Holder[];
};

type Holder = {
  holder_address: string;
  quantity: number;
};

const token = ref<Token | null>(null);
watch([collectionId, tokenId], async ([collectionId, tokenId]) => {
  if (!collectionId || !tokenId) return;
  const data = await client.request(`
query Collectors {
  token(
    where: {token_id: {_eq: "61"}, fa_contract: {_eq: "KT1Bha1xqnWAddhkukzUyq11cs5AZ7o1VYs9"}}
  ) {
    name
    supply
    thumbnail_uri
    holders(order_by: {quantity: desc}, where: {quantity: {_gt: "0"}}) {
      holder_address
      quantity
    }
  }
}
`);
  if (data.token.length) {
    token.value = data.token[0];
    errorMessage.value = "";
  } else {
    errorMessage.value = "Token not found";
  }
});

const randomCollector = ref("");
const now = ref("");

const getRandomCollector = () => {
  if (!token.value) return;
  let collectors = token.value.holders
    .map((holder) => {
      return Array(holder.quantity).fill(holder.holder_address);
    })
    .flat();

  collectors = shuffleArray(collectors);

  randomCollector.value =
    collectors[Math.floor(Math.random() * collectors.length)];

  now.value = new Date().toLocaleString("en-GB", {
    day: "numeric",
    month: "short",
    year: "numeric",
    hour: "2-digit",
    minute: "2-digit",
    second: "2-digit",
  });
};

const shuffleArray = (array: Holder[]) => {
  for (let i = array.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    const temp = array[i];
    array[i] = array[j];
    array[j] = temp;
  }

  return array;
};

onMounted(() => {
  const urlParams = new URLSearchParams(window.location.search);
  const collection = urlParams.get("collection");
  const token = urlParams.get("token");
  if (collection && token) {
    collectionId.value = collection;
    tokenId.value = token;
  }
});
</script>

<template>
  <div class="min-h-screen bg-gray-200 flex flex-col">
    <main class="grid grid-cols-1 lg:grid-cols-2 gap-16 py-10 px-3">
      <div class="flex flex-col space-y-4 bg-white rounded-md shadow-md p-4">
        <h1 class="text-2xl font-bold">Find a token</h1>
        <div class="flex space-x-4 items-center">
          <label class="text-sm text-gray-800" for="collection-id"
            >Collection ID</label
          >
          <input
            class="bg-gray-200 border-0 shadow rounded px-4 py-1 flex-grow"
            id="collection-id"
            name="collection"
            type="text"
            v-model="collectionId"
          />
        </div>
        <div class="flex space-x-4 items-center">
          <label class="text-sm text-gray-800" for="token-id">Token ID</label>
          <input
            class="bg-gray-200 border-0 shadow rounded px-4 py-1 flex-grow"
            id="token-id"
            name="token"
            type="text"
            v-model="tokenId"
          />
        </div>
        <div class="text-red-500">
          <p>{{ errorMessage }}</p>
        </div>

        <div>
          <p class="text-gray-500 text-xs">
            <span class="text-gray-800 font-bold">Example:</span>
            https://objkt.com/asset/COLLECTION_ID/TOKEN_ID
          </p>
        </div>

        <div v-if="token">
          <h2 class="text-xl font-bold">Token info</h2>
          <div class="flex flex-col space-y-4 mt-2">
            <h3>{{ token.name }}</h3>
            <p class="text-xs">
              <span class="text-gray-800 font-bold">Supply:</span>
              {{ token.supply }}
            </p>
            <img
              class="aspect-square object-cover object-center rounded-md"
              :src="`https://ipfs.io/ipfs/${token.thumbnail_uri.replace(
                'ipfs://',
                ''
              )}`"
              alt="token thumbnail"
            />
          </div>
        </div>
      </div>

      <div
        v-if="token"
        class="flex flex-col space-y-4 bg-white rounded-md shadow-md p-4"
      >
        <h2 class="text-xl font-bold">Collectors</h2>
        <ul class="overflow-y-auto h-[50vh] bg-gray-100 p-4 rounded">
          <li class="px-2 py-1">
            <ul class="grid grid-cols-4 space-x-4">
              <li class="col-span-3">Address</li>
              <li class="text-right">Quantity</li>
            </ul>
          </li>
          <li
            v-for="holder in token.holders"
            :key="holder.holder_address"
            class="odd:bg-gray-50 even:bg-gray-200 px-2 py-1"
          >
            <ul class="grid grid-cols-4 space-x-4">
              <li class="font-mono col-span-3">{{ holder.holder_address }}</li>
              <li class="text-right">{{ holder.quantity }}</li>
            </ul>
          </li>
        </ul>

        <div class="flex w-full justify-end pt-4 space-x-4">
          <button
            class="bg-indigo-300 text-sm text-indigo-700 px-4 py-1 rounded-md shadow"
            @click="getRandomCollector"
          >
            Get a random Collector address
          </button>
        </div>
        <div class="flex flex-col space-y-2" v-if="randomCollector">
          <input
            class="bg-gray-200 border-0 shadow rounded px-4 py-1 flex-grow w-full"
            type="text"
            readonly
            v-model="randomCollector"
          />
          <p class="text-xs text-gray-500">
            <span class="text-gray-800 font-bold">Last updated:</span>
            {{ now }}
          </p>
        </div>
      </div>
    </main>

    <footer class="text-xs flex items-center justify-center mb-4 space-x-6">
      <div>
        made by
        <a class="underline" href="https://twitter.com/e29HDm" target="_blank"
          >@e29HDm</a
        >
      </div>
      <div>
        <a
          class="underline"
          href="https://github.com/e29HDm/random-collector"
          target="_blank"
          >Code on github</a
        >
      </div>
    </footer>
  </div>
</template>
