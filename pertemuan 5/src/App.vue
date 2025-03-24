<script setup>
import { ref, computed } from 'vue';

// Data Barang (List Rendering)
const items = ref([
  { id: 1, name: 'Laptop', price: 15000000, quantity: 1 },
  { id: 2, name: 'Mouse', price: 300000, quantity: 2 },
  { id: 3, name: 'Keyboard', price: 500000, quantity: 1 }
]);

const newItemName = ref('');
const newItemPrice = ref('');
const newItemQuantity = ref('');
const discount = ref(0);
const taxRate = ref(10); // Pajak 10%
const transactions = ref([]);
const paymentMethod = ref('Cash');

// Computed Property: Hitung total harga belanjaan setelah diskon dan pajak
const totalPrice = computed(() => {
  const total = items.value.reduce((sum, item) => sum + item.price * item.quantity, 0);
  const discountedTotal = total - (total * discount.value) / 100;
  const finalTotal = discountedTotal + (discountedTotal * taxRate.value) / 100;
  return finalTotal;
});

// Fungsi Menambahkan Barang
const addItem = () => {
  if (!newItemName.value || !newItemPrice.value || !newItemQuantity.value) return;
  const price = parseInt(newItemPrice.value);
  const quantity = parseInt(newItemQuantity.value);
  if (price <= 0 || quantity <= 0) return;

  items.value.push({
    id: Date.now(),
    name: newItemName.value,
    price: price,
    quantity: quantity
  });

  newItemName.value = '';
  newItemPrice.value = '';
  newItemQuantity.value = '';
};

// Fungsi Menghapus Barang
const removeItem = (id) => {
  items.value = items.value.filter(item => item.id !== id);
};

// Fungsi Menambah & Mengurangi Jumlah Barang
const increaseQuantity = (id) => {
  const item = items.value.find(item => item.id === id);
  if (item) item.quantity++;
};

const decreaseQuantity = (id) => {
  const item = items.value.find(item => item.id === id);
  if (item && item.quantity > 1) {
    item.quantity--;
  }
};

// Fungsi Checkout
const checkout = () => {
  transactions.value.push({
    date: new Date().toLocaleString(),
    items: [...items.value],
    total: totalPrice.value,
    paymentMethod: paymentMethod.value
  });
  alert(`Pembayaran sebesar Rp${totalPrice.value.toLocaleString()} berhasil dengan ${paymentMethod.value}!`);
  items.value = [];
};

// Fungsi Menghapus Riwayat Transaksi
const clearTransactionHistory = () => {
  transactions.value = [];
};
</script>

<template>
  <div class="container mx-auto max-w-lg text-center p-6 bg-white text-black shadow-lg rounded-lg">
    <h1 class="text-3xl font-bold mb-4">Daftar Belanja</h1>

    <!-- Tambah Barang -->
    <div class="grid grid-cols-3 gap-2 mb-4">
      <input v-model="newItemName" placeholder="Nama barang" class="border p-2 rounded text-black" />
      <input v-model="newItemPrice" type="number" placeholder="Harga (Rp)" class="border p-2 rounded text-black" />
      <input v-model="newItemQuantity" type="number" placeholder="Jumlah" class="border p-2 rounded text-black" />
    </div>
    <button @click="addItem" class="bg-blue-700 text-white px-4 py-2 rounded w-full">Tambah Barang</button>

    <!-- Diskon dan Pajak -->
    <div class="mt-4">
      <label class="block">Diskon (%)</label>
      <input v-model.number="discount" type="number" class="border p-2 w-full rounded text-black" placeholder="Masukkan diskon" />
    </div>
    <div class="mt-2">
      <label class="block">Pajak (PPN %)</label>
      <input v-model.number="taxRate" type="number" class="border p-2 w-full rounded text-black" placeholder="Masukkan pajak" />
    </div>

    <!-- List Rendering dengan Animasi -->
    <transition-group name="fade" tag="ul" class="space-y-2 mt-4">
      <li v-for="item in items" :key="item.id" class="flex justify-between items-center bg-gray-200 text-black p-3 rounded-lg shadow-md">
        <div class="flex flex-col text-left">
          <span class="font-semibold">{{ item.name }}</span>
          <span class="text-gray-600">Rp{{ item.price.toLocaleString() }}</span>
        </div>
        <div class="flex items-center gap-2">
          <button @click="decreaseQuantity(item.id)" class="bg-red-500 text-white px-2 py-1 rounded">-</button>
          <span class="font-bold">{{ item.quantity }}</span>
          <button @click="increaseQuantity(item.id)" class="bg-green-500 text-white px-2 py-1 rounded">+</button>
        </div>
        <button @click="removeItem(item.id)" class="text-red-500">Hapus</button>
      </li>
    </transition-group>

    <!-- Metode Pembayaran -->
    <div class="mt-4">
      <label class="block">Metode Pembayaran</label>
      <select v-model="paymentMethod" class="border p-2 w-full rounded text-black">
        <option>Cash</option>
        <option>Kartu Kredit</option>
        <option>E-Wallet</option>
      </select>
    </div>

    <!-- Total Harga -->
    <h2 class="mt-4 bg-yellow-400 text-black py-2 rounded-lg font-bold text-lg">Total: Rp{{ totalPrice.toLocaleString() }}</h2>

    <!-- Tombol Checkout -->
    <button @click="checkout" class="mt-4 bg-green-600 text-white px-4 py-2 rounded w-full">Checkout</button>

    <!-- Riwayat Transaksi -->
    <div v-if="transactions.length" class="mt-6 bg-gray-800 p-4 rounded-lg">
      <h2 class="text-xl font-bold text-white">Riwayat Transaksi</h2>
      <ul class="mt-2 text-white">
        <li v-for="(transaction, index) in transactions" :key="index" class="text-sm border-b py-2">
          <strong>{{ transaction.date }}</strong> - Rp{{ transaction.total.toLocaleString() }} ({{ transaction.paymentMethod }})
        </li>
      </ul>
      <button @click="clearTransactionHistory" class="mt-4 bg-red-500 text-white px-4 py-2 rounded w-full">Hapus Riwayat</button>
    </div>
  </div>
</template>
