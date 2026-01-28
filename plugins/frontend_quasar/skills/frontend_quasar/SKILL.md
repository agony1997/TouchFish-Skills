---
name: frontend_quasar
description: "Quasar Framework 開發專家,精通 Vue 3 多平台應用。關鍵字: quasar, vue3, SPA, PWA, 前端框架"
---

# Quasar Framework 專家技能

## 角色定位
你是 Quasar Framework 開發專家,精通使用 Quasar 建構高品質 Vue 3 應用程式。專注於響應式設計、Material Design 元件、以及多平台部署 (SPA, PWA, SSR, Mobile, Desktop)。

## 核心能力

### 1. Quasar 核心概念

#### 專案結構
```
src/
├── assets/           # 靜態資源
├── components/       # 可重用元件
├── layouts/          # 布局元件
├── pages/            # 頁面元件
├── router/           # Vue Router 路由配置
├── stores/           # Pinia 狀態管理
├── boot/             # 啟動檔案 (plugins, axios 配置等)
├── css/              # 全域樣式
└── App.vue           # 根元件
```

#### Quasar CLI
```bash
# 建立專案
npm init quasar

# 開發模式
quasar dev

# 建構生產版本
quasar build

# 加入模式 (PWA, Electron, Capacitor)
quasar mode add pwa
quasar mode add electron

# Lint
quasar inspect
```

### 2. Layout 系統

#### QLayout 結構
```vue
<template>
  <q-layout view="lHh Lpr lFf">
    <!-- Header -->
    <q-header elevated>
      <q-toolbar>
        <q-btn flat dense round icon="menu" aria-label="Menu" @click="toggleLeftDrawer" />
        <q-toolbar-title>App Title</q-toolbar-title>
        <q-btn flat round dense icon="notifications" />
      </q-toolbar>
    </q-header>

    <!-- Left Drawer (側邊欄) -->
    <q-drawer v-model="leftDrawerOpen" show-if-above bordered>
      <q-list>
        <q-item-label header>Navigation</q-item-label>
        <q-item clickable v-ripple to="/">
          <q-item-section avatar>
            <q-icon name="home" />
          </q-item-section>
          <q-item-section>
            <q-item-label>Home</q-item-label>
          </q-item-section>
        </q-item>
      </q-list>
    </q-drawer>

    <!-- Page Content -->
    <q-page-container>
      <router-view />
    </q-page-container>

    <!-- Footer -->
    <q-footer elevated>
      <q-toolbar>
        <q-toolbar-title>Footer</q-toolbar-title>
      </q-toolbar>
    </q-footer>
  </q-layout>
</template>

<script setup lang="ts">
import { ref } from 'vue'

const leftDrawerOpen = ref(false)

function toggleLeftDrawer() {
  leftDrawerOpen.value = !leftDrawerOpen.value
}
</script>
```

#### Layout View 配置
- `l/r/h/f`: Left/Right/Header/Footer (小寫 = 始終可見)
- `L/R/H/F`: Left/Right/Header/Footer (大寫 = 響應式隱藏)
- `p`: Page (內容區域)

常用配置:
- `hHh Lpr lFf`: Header 固定, 左側抽屜響應式, Footer 固定
- `lHh Lpr lFf`: Header 和 Footer 固定, 左側抽屜始終顯示

### 3. 核心元件

#### QTable (資料表格)
```vue
<template>
  <q-table
    title="Users"
    :rows="rows"
    :columns="columns"
    row-key="id"
    v-model:pagination="pagination"
    :loading="loading"
    :filter="filter"
    @request="onRequest"
    selection="multiple"
    v-model:selected="selected"
  >
    <!-- 頂部插槽 -->
    <template v-slot:top-right>
      <q-input borderless dense debounce="300" v-model="filter" placeholder="Search">
        <template v-slot:append>
          <q-icon name="search" />
        </template>
      </q-input>
    </template>

    <!-- 自訂欄位 -->
    <template v-slot:body-cell-actions="props">
      <q-td :props="props">
        <q-btn flat round dense icon="edit" @click="editRow(props.row)" />
        <q-btn flat round dense icon="delete" @click="deleteRow(props.row)" />
      </q-td>
    </template>
  </q-table>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'
import type { QTableProps } from 'quasar'

interface User {
  id: number
  name: string
  email: string
  role: string
}

const columns: QTableProps['columns'] = [
  { name: 'id', label: 'ID', field: 'id', align: 'left', sortable: true },
  { name: 'name', label: 'Name', field: 'name', align: 'left', sortable: true },
  { name: 'email', label: 'Email', field: 'email', align: 'left', sortable: true },
  { name: 'role', label: 'Role', field: 'role', align: 'left', sortable: true },
  { name: 'actions', label: 'Actions', field: 'actions', align: 'center' }
]

const rows = ref<User[]>([])
const loading = ref(false)
const filter = ref('')
const selected = ref<User[]>([])

const pagination = ref({
  sortBy: 'id',
  descending: false,
  page: 1,
  rowsPerPage: 10,
  rowsNumber: 0
})

async function onRequest(props: any) {
  const { page, rowsPerPage, sortBy, descending } = props.pagination
  loading.value = true

  try {
    // API 呼叫
    const response = await fetchUsers({
      page,
      pageSize: rowsPerPage,
      sortBy,
      descending,
      filter: filter.value
    })

    rows.value = response.data
    pagination.value.rowsNumber = response.total
    pagination.value.page = page
    pagination.value.rowsPerPage = rowsPerPage
    pagination.value.sortBy = sortBy
    pagination.value.descending = descending
  } finally {
    loading.value = false
  }
}

function editRow(row: User) {
  // 編輯邏輯
}

function deleteRow(row: User) {
  // 刪除邏輯
}

onMounted(() => {
  onRequest({ pagination: pagination.value })
})
</script>
```

#### QForm (表單)
```vue
<template>
  <q-form @submit="onSubmit" @reset="onReset" class="q-gutter-md">
    <q-input
      filled
      v-model="form.name"
      label="Name *"
      hint="Enter your full name"
      lazy-rules
      :rules="[
        val => val && val.length > 0 || 'Please enter your name',
        val => val.length >= 3 || 'Name must be at least 3 characters'
      ]"
    />

    <q-input
      filled
      type="email"
      v-model="form.email"
      label="Email *"
      lazy-rules
      :rules="[
        val => val && val.length > 0 || 'Please enter your email',
        val => /.+@.+\..+/.test(val) || 'Invalid email format'
      ]"
    />

    <q-select
      filled
      v-model="form.role"
      :options="roleOptions"
      label="Role *"
      :rules="[val => val !== null || 'Please select a role']"
    />

    <q-toggle v-model="form.active" label="Active" />

    <q-input
      filled
      v-model="form.bio"
      label="Bio"
      type="textarea"
      rows="4"
    />

    <div>
      <q-btn label="Submit" type="submit" color="primary" />
      <q-btn label="Reset" type="reset" color="secondary" flat class="q-ml-sm" />
    </div>
  </q-form>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import { useQuasar } from 'quasar'

const $q = useQuasar()

const form = ref({
  name: '',
  email: '',
  role: null,
  active: true,
  bio: ''
})

const roleOptions = ['Admin', 'User', 'Guest']

function onSubmit() {
  $q.notify({
    color: 'green-4',
    textColor: 'white',
    icon: 'cloud_done',
    message: 'Form submitted successfully'
  })

  // 提交邏輯
  console.log('Form data:', form.value)
}

function onReset() {
  form.value = {
    name: '',
    email: '',
    role: null,
    active: true,
    bio: ''
  }
}
</script>
```

#### QDialog (對話框)
```vue
<template>
  <q-btn label="Open Dialog" color="primary" @click="dialog = true" />

  <q-dialog v-model="dialog" persistent>
    <q-card style="min-width: 350px">
      <q-card-section>
        <div class="text-h6">Confirm Action</div>
      </q-card-section>

      <q-card-section class="q-pt-none">
        Are you sure you want to delete this item?
      </q-card-section>

      <q-card-actions align="right" class="text-primary">
        <q-btn flat label="Cancel" v-close-popup />
        <q-btn flat label="Delete" color="negative" @click="confirmDelete" />
      </q-card-actions>
    </q-card>
  </q-dialog>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import { useQuasar } from 'quasar'

const $q = useQuasar()
const dialog = ref(false)

function confirmDelete() {
  dialog.value = false
  $q.notify({
    color: 'negative',
    message: 'Item deleted',
    icon: 'delete'
  })
}
</script>
```

#### QBtn (按鈕)
```vue
<template>
  <!-- 基礎按鈕 -->
  <q-btn label="Primary" color="primary" />
  <q-btn label="Secondary" color="secondary" />
  <q-btn label="Negative" color="negative" />

  <!-- 樣式變化 -->
  <q-btn label="Flat" flat color="primary" />
  <q-btn label="Outline" outline color="primary" />
  <q-btn label="Unelevated" unelevated color="primary" />

  <!-- Icon 按鈕 -->
  <q-btn icon="add" color="primary" round />
  <q-btn icon="edit" label="Edit" color="primary" />
  <q-btn label="Delete" icon-right="delete" color="negative" />

  <!-- Loading 狀態 -->
  <q-btn label="Submit" color="primary" :loading="loading" @click="handleSubmit" />

  <!-- Disabled -->
  <q-btn label="Disabled" color="primary" disable />
</template>

<script setup lang="ts">
import { ref } from 'vue'

const loading = ref(false)

async function handleSubmit() {
  loading.value = true
  await new Promise(resolve => setTimeout(resolve, 2000))
  loading.value = false
}
</script>
```

### 4. Quasar Plugins

#### Notify (通知)
```typescript
import { Notify } from 'quasar'

// 成功通知
Notify.create({
  type: 'positive',
  message: 'Operation successful',
  position: 'top'
})

// 錯誤通知
Notify.create({
  type: 'negative',
  message: 'An error occurred',
  position: 'top-right',
  timeout: 3000
})

// 自訂通知
Notify.create({
  color: 'purple',
  textColor: 'white',
  icon: 'info',
  message: 'Custom notification',
  actions: [
    { label: 'Dismiss', color: 'white', handler: () => {} }
  ]
})
```

#### Loading (載入指示)
```typescript
import { Loading } from 'quasar'

// 顯示 Loading
Loading.show({
  message: 'Please wait...',
  spinnerColor: 'white'
})

// 隱藏 Loading
Loading.hide()

// 使用範例
async function fetchData() {
  Loading.show()
  try {
    const data = await api.getData()
    return data
  } finally {
    Loading.hide()
  }
}
```

#### Dialog (程式化對話框)
```typescript
import { Dialog } from 'quasar'

// 確認對話框
Dialog.create({
  title: 'Confirm',
  message: 'Are you sure you want to delete this?',
  cancel: true,
  persistent: true
}).onOk(() => {
  // 確認動作
}).onCancel(() => {
  // 取消動作
})

// Prompt 對話框
Dialog.create({
  title: 'Enter Name',
  message: 'Please enter your name',
  prompt: {
    model: '',
    type: 'text'
  },
  cancel: true
}).onOk(data => {
  console.log('User entered:', data)
})
```

### 5. 整合技術棧

#### Vue 3 Composition API
```vue
<script setup lang="ts">
import { ref, computed, watch, onMounted } from 'vue'
import { useRouter, useRoute } from 'vue-router'
import { useUserStore } from 'stores/user'

const router = useRouter()
const route = useRoute()
const userStore = useUserStore()

const count = ref(0)
const doubleCount = computed(() => count.value * 2)

watch(count, (newVal, oldVal) => {
  console.log(`Count changed from ${oldVal} to ${newVal}`)
})

onMounted(() => {
  console.log('Component mounted')
})
</script>
```

#### Pinia 狀態管理
```typescript
// stores/user.ts
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'
import type { User } from 'src/types'

export const useUserStore = defineStore('user', () => {
  const user = ref<User | null>(null)
  const isAuthenticated = computed(() => user.value !== null)

  function setUser(newUser: User) {
    user.value = newUser
  }

  function logout() {
    user.value = null
  }

  return {
    user,
    isAuthenticated,
    setUser,
    logout
  }
})
```

#### Axios 整合
```typescript
// boot/axios.ts
import { boot } from 'quasar/wrappers'
import axios, { AxiosInstance } from 'axios'
import { Notify } from 'quasar'

declare module '@vue/runtime-core' {
  interface ComponentCustomProperties {
    $axios: AxiosInstance
    $api: AxiosInstance
  }
}

const api = axios.create({
  baseURL: import.meta.env.VITE_API_BASE_URL || 'http://localhost:8080/api'
})

// Request Interceptor
api.interceptors.request.use(
  config => {
    const token = localStorage.getItem('token')
    if (token) {
      config.headers.Authorization = `Bearer ${token}`
    }
    return config
  },
  error => Promise.reject(error)
)

// Response Interceptor
api.interceptors.response.use(
  response => response,
  error => {
    if (error.response?.status === 401) {
      Notify.create({
        type: 'negative',
        message: 'Unauthorized. Please login again.'
      })
      // 跳轉到登入頁
    }
    return Promise.reject(error)
  }
)

export default boot(({ app }) => {
  app.config.globalProperties.$axios = axios
  app.config.globalProperties.$api = api
})

export { api }
```

#### 環境變數配置
```bash
# .env
VITE_API_BASE_URL=http://localhost:8080/api
VITE_APP_TITLE=My Quasar App

# .env.production
VITE_API_BASE_URL=https://api.example.com
```

```typescript
// 使用環境變數
const apiUrl = import.meta.env.VITE_API_BASE_URL
const appTitle = import.meta.env.VITE_APP_TITLE
```

### 6. Vue Router 整合
```typescript
// router/routes.ts
import { RouteRecordRaw } from 'vue-router'

const routes: RouteRecordRaw[] = [
  {
    path: '/',
    component: () => import('layouts/MainLayout.vue'),
    children: [
      { path: '', component: () => import('pages/IndexPage.vue') },
      { path: 'users', component: () => import('pages/UsersPage.vue') }
    ]
  },
  {
    path: '/login',
    component: () => import('pages/LoginPage.vue')
  },
  {
    path: '/:catchAll(.*)*',
    component: () => import('pages/ErrorNotFound.vue')
  }
]

export default routes
```

### 7. 響應式設計

#### Quasar Screen Plugin
```vue
<script setup lang="ts">
import { useQuasar } from 'quasar'

const $q = useQuasar()

// 檢查螢幕尺寸
const isMobile = $q.screen.lt.sm  // < 600px
const isTablet = $q.screen.lt.md  // < 1024px
const isDesktop = $q.screen.gt.md // > 1024px
</script>

<template>
  <div>
    <div v-if="$q.screen.lt.sm">Mobile View</div>
    <div v-else-if="$q.screen.lt.md">Tablet View</div>
    <div v-else>Desktop View</div>
  </div>
</template>
```

#### CSS 響應式工具
```vue
<template>
  <!-- 響應式隱藏/顯示 -->
  <div class="gt-sm">顯示在 > 600px</div>
  <div class="lt-md">顯示在 < 1024px</div>

  <!-- 響應式 Grid -->
  <div class="row">
    <div class="col-12 col-md-6 col-lg-4">欄位</div>
  </div>
</template>
```

## 開發最佳實踐

1. **元件化**: 將可重用的 UI 拆分為獨立元件
2. **TypeScript**: 使用 TypeScript 提升型別安全
3. **Composition API**: 優先使用 `<script setup>` 語法
4. **狀態管理**: 全域狀態使用 Pinia
5. **API 層**: 統一管理 API 呼叫,使用 Axios Interceptor
6. **錯誤處理**: 統一處理 API 錯誤,使用 Notify 通知用戶
7. **Loading 狀態**: 適當顯示載入指示,提升用戶體驗
8. **響應式設計**: 使用 Quasar Grid 和 Screen Plugin
9. **環境配置**: 使用 .env 管理環境變數
10. **程式碼分割**: 使用 Vue Router 的 lazy loading

## 參考資源
- Quasar 官方文檔: https://quasar.dev
- Quasar Components: https://quasar.dev/vue-components
- Quasar CLI: https://quasar.dev/quasar-cli
