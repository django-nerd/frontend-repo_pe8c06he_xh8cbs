<template>
  <div class="min-h-screen bg-gray-50 text-gray-900">
    <header class="sticky top-0 z-10 bg-white border-b border-gray-200">
      <div class="mx-auto max-w-7xl px-4 py-4 flex items-center justify-between">
        <h1 class="text-xl font-semibold">Cosmetics Admin</h1>
        <div class="flex items-center gap-3">
          <span class="text-sm text-gray-600">Orders</span>
        </div>
      </div>
    </header>

    <main class="mx-auto max-w-7xl p-4">
      <section class="bg-white rounded-xl shadow-sm border border-gray-200 p-4">
        <div class="flex flex-col md:flex-row md:items-center md:justify-between gap-3">
          <div class="flex-1 flex items-center gap-3">
            <div class="relative w-full md:w-80">
              <input v-model="search" type="text" placeholder="Search orders..."
                     class="w-full rounded-lg border border-gray-300 bg-white px-3 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-rose-500" />
              <div class="absolute inset-y-0 right-0 flex items-center pr-3 text-gray-400">
                <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 21l-4.35-4.35M11 19a8 8 0 100-16 8 8 0 000 16z"/></svg>
              </div>
            </div>
            <select v-model="status" class="rounded-lg border border-gray-300 bg-white px-3 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-rose-500">
              <option value="">All statuses</option>
              <option v-for="s in statuses" :key="s" :value="s">{{ s }}</option>
            </select>
          </div>

          <div class="flex items-center gap-2">
            <button @click="fetchOrders(1)" class="inline-flex items-center gap-2 rounded-lg bg-rose-600 px-3 py-2 text-sm font-medium text-white hover:bg-rose-700">
              Refresh
            </button>
            <button @click="seedDemo" class="inline-flex items-center gap-2 rounded-lg border border-gray-300 px-3 py-2 text-sm font-medium hover:bg-gray-50">
              Seed demo data
            </button>
          </div>
        </div>

        <div class="mt-4 overflow-x-auto">
          <table class="min-w-full text-sm">
            <thead>
              <tr class="text-left text-gray-600">
                <th class="px-3 py-2">Order</th>
                <th class="px-3 py-2">Customer</th>
                <th class="px-3 py-2">Email</th>
                <th class="px-3 py-2">Status</th>
                <th class="px-3 py-2">Total</th>
                <th class="px-3 py-2">Placed</th>
                <th class="px-3 py-2"></th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="o in orders" :key="o.id" class="border-t border-gray-100">
                <td class="px-3 py-3 font-medium">{{ o.order_number }}</td>
                <td class="px-3 py-3">{{ o.customer_name }}</td>
                <td class="px-3 py-3 text-gray-600">{{ o.email }}</td>
                <td class="px-3 py-3">
                  <span :class="statusBadge(o.status)">{{ o.status }}</span>
                </td>
                <td class="px-3 py-3">${{ o.total_amount.toFixed(2) }}</td>
                <td class="px-3 py-3 text-gray-600">{{ formatDate(o.created_at) }}</td>
                <td class="px-3 py-3 text-right">
                  <button @click="openOrder(o)" class="text-rose-600 hover:underline">View</button>
                </td>
              </tr>
            </tbody>
          </table>

          <div v-if="orders.length === 0" class="py-12 text-center text-gray-500">No orders found.</div>
        </div>

        <div class="mt-4 flex items-center justify-between text-sm">
          <div class="text-gray-600">Page {{ page }} of {{ totalPages }}</div>
          <div class="flex items-center gap-2">
            <button :disabled="page === 1" @click="fetchOrders(page-1)" class="px-3 py-2 rounded border border-gray-300 disabled:opacity-50">Prev</button>
            <button :disabled="page === totalPages" @click="fetchOrders(page+1)" class="px-3 py-2 rounded border border-gray-300 disabled:opacity-50">Next</button>
          </div>
        </div>
      </section>

      <div v-if="selected" class="fixed inset-0 bg-black/40 flex items-end md:items-center justify-center p-4" @click.self="selected=null">
        <div class="w-full md:w-[700px] bg-white rounded-xl shadow-lg overflow-hidden">
          <div class="flex items-center justify-between px-4 py-3 border-b">
            <h3 class="font-semibold">Order {{ selected.order_number }}</h3>
            <button @click="selected=null" class="text-gray-500 hover:text-gray-700">✕</button>
          </div>
          <div class="p-4 space-y-4">
            <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
              <div>
                <div class="text-xs uppercase text-gray-500">Customer</div>
                <div class="font-medium">{{ selected.customer_name }}</div>
                <div class="text-gray-600 text-sm">{{ selected.email }}</div>
              </div>
              <div>
                <div class="text-xs uppercase text-gray-500">Status</div>
                <div class="mt-1">
                  <select v-model="selected.status" class="rounded-lg border border-gray-300 bg-white px-3 py-2 text-sm">
                    <option v-for="s in statuses" :key="s" :value="s">{{ s }}</option>
                  </select>
                </div>
              </div>
            </div>

            <div>
              <div class="text-xs uppercase text-gray-500 mb-2">Items</div>
              <div class="divide-y border rounded-lg">
                <div v-for="(it, idx) in selected.items" :key="idx" class="flex items-center justify-between px-3 py-2">
                  <div>
                    <div class="font-medium">{{ it.product_name }}</div>
                    <div class="text-gray-600 text-sm">Qty {{ it.quantity }}</div>
                  </div>
                  <div class="font-medium">${{ it.price.toFixed(2) }}</div>
                </div>
              </div>
            </div>

            <div class="flex items-center justify-between">
              <div class="text-gray-600">Total</div>
              <div class="text-lg font-semibold">${{ selected.total_amount.toFixed(2) }}</div>
            </div>

            <div class="flex items-center justify-end gap-2">
              <button @click="selected=null" class="px-3 py-2 rounded border border-gray-300">Cancel</button>
              <button @click="saveStatus" class="px-3 py-2 rounded bg-rose-600 text-white hover:bg-rose-700">Save</button>
            </div>
          </div>
        </div>
      </div>
    </main>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'

const API_BASE = import.meta.env.VITE_BACKEND_URL || 'http://localhost:8000'

const search = ref('')
const status = ref('')
const statuses = ['pending','processing','shipped','delivered','cancelled']

const orders = ref([])
const page = ref(1)
const pageSize = ref(10)
const total = ref(0)
const selected = ref(null)

const totalPages = computed(() => Math.max(1, Math.ceil(total.value / pageSize.value)))

function statusBadge(s){
  const base = 'inline-flex items-center px-2 py-1 rounded-full text-xs capitalize'
  const map = {
    pending: 'bg-yellow-100 text-yellow-800',
    processing: 'bg-blue-100 text-blue-800',
    shipped: 'bg-purple-100 text-purple-800',
    delivered: 'bg-emerald-100 text-emerald-800',
    cancelled: 'bg-rose-100 text-rose-800'
  }
  return base + ' ' + (map[s] || 'bg-gray-100 text-gray-700')
}

function formatDate(iso){
  try { return new Date(iso).toLocaleString() } catch { return '-' }
}

async function fetchOrders(p=1){
  page.value = p
  const params = new URLSearchParams()
  if (search.value) params.set('q', search.value)
  if (status.value) params.set('status', status.value)
  params.set('page', String(page.value))
  params.set('page_size', String(pageSize.value))
  const res = await fetch(`${API_BASE}/api/orders?${params.toString()}`)
  const data = await res.json()
  orders.value = data.items || []
  total.value = data.total || 0
}

function openOrder(o){
  selected.value = JSON.parse(JSON.stringify(o))
}

async function saveStatus(){
  if (!selected.value) return
  const res = await fetch(`${API_BASE}/api/orders/${selected.value.id}/status`,{
    method: 'PATCH',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ status: selected.value.status })
  })
  if (res.ok){
    await fetchOrders(page.value)
    selected.value = null
  }
}

async function seedDemo(){
  await fetch(`${API_BASE}/api/orders/seed`, { method: 'POST' })
  await fetchOrders(1)
}

onMounted(() => {
  fetchOrders(1)
})
</script>

<style>
:root { font-family: Inter, system-ui, -apple-system, Segoe UI, Roboto, Ubuntu, Cantarell, Noto Sans, Helvetica, Arial, "Apple Color Emoji", "Segoe UI Emoji"; }
</style>
