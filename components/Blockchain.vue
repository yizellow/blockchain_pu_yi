<template>
  <div class="p-6 bg-gray-100 min-h-screen">
    <h1 class="text-2xl font-bold mb-4">🧱 區塊鏈模擬器（Vue 3）</h1>

    <div class="flex gap-2 mb-4">
      <input v-model="newData" type="text" placeholder="輸入資料"
             class="border p-2 rounded w-1/2" />
      <button @click="addBlock" class="bg-blue-500 text-white px-4 py-2 rounded">新增區塊</button>
      <button @click="checkValidity" class="bg-green-500 text-white px-4 py-2 rounded">驗證鏈</button>
    </div>

    <div v-for="block in chain" :key="block.index" class="bg-white p-4 rounded shadow mb-4">
      <h2 class="font-bold">區塊 {{ block.index }}</h2>
      <p><strong>時間：</strong>{{ block.timestamp }}</p>
      <p><strong>資料：</strong>{{ block.data }}</p>
      <p><strong>雜湊：</strong><pre class="break-all whitespace-pre-wrap">{{ block.hash }}</pre></p>
      <p><strong>前一雜湊：</strong><pre class="break-all whitespace-pre-wrap">{{ block.previousHash }}</pre></p>
    </div>
  </div>
</template>

<script setup>
import { reactive, ref } from 'vue'

function sha256(str) {
  const encoder = new TextEncoder()
  const data = encoder.encode(str)
  return crypto.subtle.digest('SHA-256', data).then(buf => {
    return Array.from(new Uint8Array(buf))
      .map(b => b.toString(16).padStart(2, '0'))
      .join('')
  })
}

class Block {
  constructor(index, timestamp, data, previousHash = '') {
    this.index = index
    this.timestamp = timestamp
    this.data = data
    this.previousHash = previousHash
    this.hash = ''
  }

  async calculateHash() {
    const str = this.index + this.previousHash + this.timestamp + JSON.stringify(this.data)
    this.hash = await sha256(str)
  }
}

class Blockchain {
  constructor() {
    this.chain = []
  }

  async init() {
    const genesis = new Block(0, new Date().toISOString(), 'Genesis Block', '0')
    await genesis.calculateHash()
    this.chain.push(genesis)
  }

  getLatestBlock() {
    return this.chain[this.chain.length - 1]
  }

  async addBlock(data) {
    const prev = this.getLatestBlock()
    const newBlock = new Block(
      prev.index + 1,
      new Date().toISOString(),
      data,
      prev.hash
    )
    await newBlock.calculateHash()
    this.chain.push(newBlock)
  }

  async isValid() {
    for (let i = 1; i < this.chain.length; i++) {
      const curr = this.chain[i]
      const prev = this.chain[i - 1]

      const hash = await sha256(curr.index + curr.previousHash + curr.timestamp + JSON.stringify(curr.data))
      if (curr.hash !== hash || curr.previousHash !== prev.hash) return false
    }
    return true
  }
}

// Reactive chain instance
const newData = ref('')
const chain = reactive([])

const blockchain = new Blockchain()
await blockchain.init()
chain.splice(0, chain.length, ...blockchain.chain)

async function addBlock() {
  if (!newData.value) return
  await blockchain.addBlock(newData.value)
  chain.splice(0, chain.length, ...blockchain.chain)
  newData.value = ''
}

async function checkValidity() {
  const isValid = await blockchain.isValid()
  alert(isValid ? '✅ 區塊鏈有效！' : '❌ 區塊鏈被竄改！')
}
</script>

<style scoped>
pre {
  background: #f3f3f3;
  padding: 8px;
  border-radius: 4px;
}
</style>
