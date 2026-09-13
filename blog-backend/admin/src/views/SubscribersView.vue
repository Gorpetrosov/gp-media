<template>
  <div>
    <header class="page-head">
      <div>
        <h1>Subscribers</h1>
        <p>Newsletter list with double opt-in status.</p>
      </div>
      <div class="filters">
        <select v-model="status" class="select" @change="load">
          <option value="">All statuses</option>
          <option value="pending">Pending</option>
          <option value="active">Active</option>
          <option value="unsubscribed">Unsubscribed</option>
        </select>
        <button class="btn secondary" type="button" @click="load">Refresh</button>
      </div>
    </header>

    <p v-if="error" class="error">{{ error }}</p>

    <div class="card">
      <table class="table" v-if="items.length">
        <thead>
          <tr>
            <th>Email</th>
            <th>Status</th>
            <th>Locale</th>
            <th>Confirmed</th>
            <th>Created</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="s in items" :key="s.id">
            <td>{{ s.email }}</td>
            <td>{{ s.status }}</td>
            <td>{{ s.locale }}</td>
            <td>{{ formatDate(s.confirmedAt) }}</td>
            <td>{{ formatDate(s.createdAt) }}</td>
          </tr>
        </tbody>
      </table>
      <div v-else class="empty">No subscribers yet.</div>
      <p v-if="pagination.total" class="muted meta">
        {{ pagination.total }} total · page {{ pagination.page }} / {{ pagination.pages || 1 }}
      </p>
      <div class="pager" v-if="pagination.pages > 1">
        <button class="btn secondary" :disabled="page <= 1" @click="prev">Previous</button>
        <button class="btn secondary" :disabled="page >= pagination.pages" @click="next">Next</button>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { onMounted, ref } from 'vue';
import { api } from '../api/client';

type Subscriber = {
  id: string;
  email: string;
  status: string;
  locale: string;
  confirmedAt: string | null;
  unsubscribedAt: string | null;
  createdAt: string;
};

type Pagination = { page: number; limit: number; total: number; pages: number };

const items = ref<Subscriber[]>([]);
const pagination = ref<Pagination>({ page: 1, limit: 20, total: 0, pages: 0 });
const page = ref(1);
const status = ref('');
const error = ref('');

function formatDate(value: string | null) {
  if (!value) return '—';
  return new Date(value).toLocaleString();
}

async function load() {
  error.value = '';
  try {
    const qs = new URLSearchParams({
      page: String(page.value),
      limit: '20',
      ...(status.value ? { status: status.value } : {}),
    });
    const data = await api<{ items: Subscriber[]; pagination: Pagination }>(
      `/api/admin/subscribers?${qs}`
    );
    items.value = data.items;
    pagination.value = data.pagination;
  } catch (e) {
    error.value = e instanceof Error ? e.message : 'Failed to load subscribers';
  }
}

function prev() {
  page.value -= 1;
  void load();
}

function next() {
  page.value += 1;
  void load();
}

onMounted(load);
</script>

<style scoped>
.page-head {
  display: flex;
  justify-content: space-between;
  gap: 1rem;
  align-items: start;
  margin-bottom: 1rem;
}

.filters {
  display: flex;
  gap: 0.5rem;
  align-items: center;
}

.meta {
  margin: 0.75rem 0 0;
  font-size: 0.9rem;
}

.pager {
  display: flex;
  gap: 0.5rem;
  margin-top: 0.75rem;
}

@media (max-width: 700px) {
  .page-head {
    flex-direction: column;
  }
}
</style>
