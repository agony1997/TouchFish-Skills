---
name: composite_frontend
description: "複合技能：Vue 3 + Quasar Framework + TypeScript 前端全端開發。Composition API、Composables、Vue Router、Pinia 狀態管理、Quasar Layout、核心元件 (QTable, QForm, QDialog)、Quasar Plugins (Notify, Loading, Dialog)、Axios 整合、響應式設計、TypeScript 型別系統、泛型、進階型別、工具型別、Vue+TS 整合。關鍵字: vue, vue3, composition api, composable, pinia, vue router, quasar, spa, pwa, typescript, ts, type, interface, generic, 組件, 前端, 型別, 泛型, 介面, utility types, 狀態管理, 響應式, component, reactivity, state management"
---

# 前端全端技術棧 (Composite Frontend)

你是 Vue 3 + Quasar + TypeScript 前端開發專家,涵蓋 Vue 3 核心、Pinia 狀態管理、Quasar Framework 元件與佈局、TypeScript 型別系統。

---

## Part 1: Vue 3 核心

### 1. Composition API

```vue
<script setup>
import { ref, computed, watch, onMounted } from 'vue'

const count = ref(0)
const doubleCount = computed(() => count.value * 2)

function increment() { count.value++ }

watch(count, (newVal, oldVal) => {
  console.log(`Count: ${oldVal} → ${newVal}`)
})

onMounted(() => console.log('Component mounted'))
</script>

<template>
  <div>
    <p>Count: {{ count }} / Double: {{ doubleCount }}</p>
    <button @click="increment">+1</button>
  </div>
</template>
```

### 2. Composables (可重用邏輯)

```typescript
// composables/useCounter.ts
import { ref, computed } from 'vue'

export function useCounter(initial = 0) {
  const count = ref(initial)
  const double = computed(() => count.value * 2)
  const increment = () => count.value++
  const decrement = () => count.value--
  const reset = () => { count.value = initial }
  return { count, double, increment, decrement, reset }
}
```

```typescript
// composables/useApi.ts
import { ref } from 'vue'

export function useApi<T>(fetcher: () => Promise<T>) {
  const data = ref<T | null>(null)
  const loading = ref(false)
  const error = ref<string | null>(null)

  async function execute() {
    loading.value = true
    error.value = null
    try {
      data.value = await fetcher()
    } catch (err: any) {
      error.value = err.message
    } finally {
      loading.value = false
    }
  }

  return { data, loading, error, execute }
}
```

### 3. Vue Router

```typescript
import { createRouter, createWebHistory } from 'vue-router'

const routes = [
  { path: '/', name: 'Home', component: () => import('@/views/Home.vue') },
  { path: '/users/:id', name: 'UserDetail', component: () => import('@/views/UserDetail.vue'), props: true },
  {
    path: '/admin',
    component: () => import('@/layouts/AdminLayout.vue'),
    meta: { requiresAuth: true },
    children: [
      { path: 'dashboard', component: () => import('@/views/admin/Dashboard.vue') }
    ]
  }
]

const router = createRouter({ history: createWebHistory(), routes })

router.beforeEach((to, from, next) => {
  if (to.meta.requiresAuth && !isAuthenticated()) next('/login')
  else next()
})

export default router
```

### 4. 組件通訊

```vue
<!-- Parent -->
<script setup>
import { ref } from 'vue'
import Child from './Child.vue'
const msg = ref('Hello')
</script>
<template>
  <Child :message="msg" @update="msg = $event" />
</template>

<!-- Child -->
<script setup>
const props = defineProps<{ message: string }>()
const emit = defineEmits<{ update: [value: string] }>()
</script>
<template>
  <p>{{ message }}</p>
  <button @click="emit('update', 'New message')">Update</button>
</template>
```

### 5. 表單處理

```vue
<script setup>
import { reactive, ref } from 'vue'

const form = reactive({ name: '', email: '', password: '' })
const errors = ref<Record<string, string>>({})

async function handleSubmit() {
  errors.value = {}
  if (!form.name) errors.value.name = '請輸入姓名'
  if (!form.email) errors.value.email = '請輸入電子郵件'
  if (Object.keys(errors.value).length > 0) return
  await api.submit(form)
}
</script>

<template>
  <form @submit.prevent="handleSubmit">
    <input v-model="form.name" type="text">
    <span v-if="errors.name">{{ errors.name }}</span>
    <input v-model="form.email" type="email">
    <button type="submit">提交</button>
  </form>
</template>
```

### 6. 效能優化

- `v-if` vs `v-show`: v-if 移除 DOM,v-show 切換 display (頻繁切換用 v-show)
- 懶載入: `component: () => import('@/views/About.vue')`
- `computed` 有快取,比 methods 效率高
- `shallowRef`/`shallowReactive` 處理大型物件
- `markRaw` 標記不需要響應式的物件

---

## Part 2: Pinia 狀態管理

```typescript
// stores/user.ts
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'

export const useUserStore = defineStore('user', () => {
  // State
  const user = ref<User | null>(null)
  const token = ref('')

  // Getters
  const isAuthenticated = computed(() => !!token.value)
  const userName = computed(() => user.value?.name || 'Guest')

  // Actions
  async function login(credentials: LoginRequest) {
    const response = await api.post('/login', credentials)
    token.value = response.data.token
    user.value = response.data.user
    localStorage.setItem('token', token.value)
  }

  function logout() {
    token.value = ''
    user.value = null
    localStorage.removeItem('token')
  }

  return { user, token, isAuthenticated, userName, login, logout }
})
```

**使用原則:**
- 僅存放真正需要共享的狀態
- 組件內部狀態保持在組件內
- 按領域劃分 Store (userStore, orderStore, productStore)

---

## Part 3: Quasar Framework

### 1. 專案結構

```
src/
├── assets/           # 靜態資源
├── components/       # 可重用元件
├── layouts/          # 布局元件
├── pages/            # 頁面元件
├── router/           # Vue Router
├── stores/           # Pinia
├── boot/             # 啟動檔案 (axios, plugins)
├── css/              # 全域樣式
└── App.vue
```

### 2. QLayout 系統

```vue
<template>
  <q-layout view="lHh Lpr lFf">
    <q-header elevated>
      <q-toolbar>
        <q-btn flat dense round icon="menu" @click="toggleDrawer" />
        <q-toolbar-title>App Title</q-toolbar-title>
        <q-btn flat round dense icon="notifications" />
      </q-toolbar>
    </q-header>

    <q-drawer v-model="drawerOpen" show-if-above bordered>
      <q-list>
        <q-item clickable v-ripple to="/">
          <q-item-section avatar><q-icon name="home" /></q-item-section>
          <q-item-section>Home</q-item-section>
        </q-item>
      </q-list>
    </q-drawer>

    <q-page-container>
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script setup lang="ts">
import { ref } from 'vue'
const drawerOpen = ref(false)
const toggleDrawer = () => { drawerOpen.value = !drawerOpen.value }
</script>
```

**Layout View 配置:** `lHh Lpr lFf` — 小寫=始終可見, 大寫=響應式隱藏, p=Page

### 3. QTable (資料表格)

```vue
<template>
  <q-table
    title="Users" :rows="rows" :columns="columns" row-key="id"
    v-model:pagination="pagination" :loading="loading" :filter="filter"
    @request="onRequest"
  >
    <template v-slot:top-right>
      <q-input borderless dense debounce="300" v-model="filter" placeholder="Search">
        <template v-slot:append><q-icon name="search" /></template>
      </q-input>
    </template>
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

const columns: QTableProps['columns'] = [
  { name: 'id', label: 'ID', field: 'id', sortable: true, align: 'left' },
  { name: 'name', label: 'Name', field: 'name', sortable: true, align: 'left' },
  { name: 'actions', label: 'Actions', field: 'actions', align: 'center' }
]

const rows = ref([])
const loading = ref(false)
const filter = ref('')
const pagination = ref({ sortBy: 'id', descending: false, page: 1, rowsPerPage: 10, rowsNumber: 0 })

async function onRequest(props: any) {
  const { page, rowsPerPage, sortBy, descending } = props.pagination
  loading.value = true
  try {
    const res = await fetchData({ page, pageSize: rowsPerPage, sortBy, descending, filter: filter.value })
    rows.value = res.data
    pagination.value = { ...pagination.value, page, rowsPerPage, sortBy, descending, rowsNumber: res.total }
  } finally { loading.value = false }
}

onMounted(() => onRequest({ pagination: pagination.value }))
</script>
```

### 4. QForm (表單)

```vue
<template>
  <q-form @submit="onSubmit" @reset="onReset" class="q-gutter-md">
    <q-input filled v-model="form.name" label="Name *" lazy-rules
      :rules="[val => !!val || 'Required', val => val.length >= 3 || 'Min 3 chars']" />
    <q-input filled type="email" v-model="form.email" label="Email *"
      :rules="[val => !!val || 'Required', val => /.+@.+\..+/.test(val) || 'Invalid email']" />
    <q-select filled v-model="form.role" :options="['Admin','User','Guest']" label="Role *"
      :rules="[val => val !== null || 'Required']" />
    <q-toggle v-model="form.active" label="Active" />
    <div>
      <q-btn label="Submit" type="submit" color="primary" />
      <q-btn label="Reset" type="reset" color="secondary" flat class="q-ml-sm" />
    </div>
  </q-form>
</template>
```

### 5. QDialog

```vue
<q-dialog v-model="dialog" persistent>
  <q-card style="min-width: 350px">
    <q-card-section><div class="text-h6">Confirm</div></q-card-section>
    <q-card-section class="q-pt-none">Are you sure?</q-card-section>
    <q-card-actions align="right">
      <q-btn flat label="Cancel" v-close-popup />
      <q-btn flat label="Delete" color="negative" @click="onConfirm" />
    </q-card-actions>
  </q-card>
</q-dialog>
```

### 6. Quasar Plugins

```typescript
import { Notify, Loading, Dialog } from 'quasar'

// Notify
Notify.create({ type: 'positive', message: 'Success', position: 'top' })
Notify.create({ type: 'negative', message: 'Error', timeout: 3000 })

// Loading
Loading.show({ message: 'Please wait...' })
Loading.hide()

// Dialog
Dialog.create({
  title: 'Confirm', message: 'Delete this item?', cancel: true, persistent: true
}).onOk(() => { /* confirm */ }).onCancel(() => { /* cancel */ })

Dialog.create({
  title: 'Input', prompt: { model: '', type: 'text' }, cancel: true
}).onOk(data => console.log('Entered:', data))
```

### 7. Axios 整合

```typescript
// boot/axios.ts
import { boot } from 'quasar/wrappers'
import axios from 'axios'
import { Notify } from 'quasar'

const api = axios.create({
  baseURL: import.meta.env.VITE_API_BASE_URL || 'http://localhost:8080/api'
})

api.interceptors.request.use(config => {
  const token = localStorage.getItem('token')
  if (token) config.headers.Authorization = `Bearer ${token}`
  return config
})

api.interceptors.response.use(
  response => response,
  error => {
    if (error.response?.status === 401) {
      Notify.create({ type: 'negative', message: 'Unauthorized. Please login again.' })
    }
    return Promise.reject(error)
  }
)

export default boot(({ app }) => { app.config.globalProperties.$api = api })
export { api }
```

### 8. 響應式設計

```vue
<script setup lang="ts">
import { useQuasar } from 'quasar'
const $q = useQuasar()
// $q.screen.lt.sm (<600px), $q.screen.lt.md (<1024px), $q.screen.gt.md (>1024px)
</script>

<template>
  <div v-if="$q.screen.lt.sm">Mobile</div>
  <div v-else-if="$q.screen.lt.md">Tablet</div>
  <div v-else>Desktop</div>

  <!-- Grid -->
  <div class="row">
    <div class="col-12 col-md-6 col-lg-4">Column</div>
  </div>
</template>
```

### 9. Router 整合

```typescript
// router/routes.ts
import type { RouteRecordRaw } from 'vue-router'

const routes: RouteRecordRaw[] = [
  {
    path: '/',
    component: () => import('layouts/MainLayout.vue'),
    children: [
      { path: '', component: () => import('pages/IndexPage.vue') },
      { path: 'users', component: () => import('pages/UsersPage.vue') }
    ]
  },
  { path: '/login', component: () => import('pages/LoginPage.vue') },
  { path: '/:catchAll(.*)*', component: () => import('pages/ErrorNotFound.vue') }
]
export default routes
```

---

## Part 4: TypeScript

### 1. 基礎型別快速參考

```typescript
// Primitives
let name: string; let age: number; let active: boolean;

// Arrays & Tuples
let nums: number[] = [1, 2]; let tuple: [string, number] = ["a", 1];

// Enum
enum Color { Red, Green, Blue }

// Special types
let val: unknown;   // 安全的 any, 需 type guard
let nothing: void;  // 函數無回傳
let never: never;   // 永不回傳 (throw)
```

### 2. Interface & Type

```typescript
// Interface — 物件結構, 可繼承
interface User {
  id: number;
  name: string;
  email: string;
  age?: number;          // Optional
  readonly created: Date; // Readonly
}

interface Employee extends User {
  department: string;
  salary: number;
}

// Type — 聯合, 交叉, 原始型別
type Status = 'pending' | 'approved' | 'rejected';  // Union
type Point = { x: number; y: number };
type AdminEmployee = User & { role: 'admin' };       // Intersection
type AddFn = (a: number, b: number) => number;       // Function type
```

### 3. Generics

```typescript
// Generic function
function identity<T>(arg: T): T { return arg; }

// Generic interface
interface ApiResponse<T> {
  data: T;
  status: number;
  message: string;
}

// Generic class
class DataStore<T> {
  private data: T[] = [];
  add(item: T): void { this.data.push(item); }
  get(index: number): T | undefined { return this.data[index]; }
}

// Generic constraint
function logLength<T extends { length: number }>(arg: T): T {
  console.log(arg.length);
  return arg;
}
```

### 4. 進階型別

```typescript
// Mapped types
type ReadonlyUser = { readonly [P in keyof User]: User[P] };
type PartialUser = { [P in keyof User]?: User[P] };

// Conditional types
type IsString<T> = T extends string ? true : false;

// Extract / Exclude
type T1 = Extract<'a' | 'b' | 'c', 'a' | 'f'>;  // 'a'
type T2 = Exclude<'a' | 'b' | 'c', 'a'>;          // 'b' | 'c'

// ReturnType / Parameters
type UserReturn = ReturnType<typeof getUser>;
```

### 5. Utility Types

```typescript
Partial<T>      // 所有屬性可選
Required<T>     // 所有屬性必填
Readonly<T>     // 所有屬性唯讀
Pick<T, K>      // 選取指定屬性
Omit<T, K>      // 排除指定屬性
Record<K, V>    // 鍵值對型別
NonNullable<T>  // 排除 null/undefined
ReturnType<F>   // 函數回傳型別
Parameters<F>   // 函數參數型別
```

```typescript
// 實際應用
type UserPreview = Pick<User, 'id' | 'name'>;
type UserWithoutPwd = Omit<User, 'password'>;
type UserRoles = Record<string, string[]>;
```

### 6. Type Guards

```typescript
// typeof
function padLeft(value: string, padding: string | number) {
  if (typeof padding === 'number') return ' '.repeat(padding) + value;
  return padding + value;
}

// instanceof
function makeSound(animal: Dog | Cat) {
  if (animal instanceof Dog) animal.bark();
  else animal.meow();
}

// Custom type guard
function isBird(pet: Bird | Fish): pet is Bird {
  return (pet as Bird).fly !== undefined;
}
```

### 7. Vue + TypeScript 整合

```vue
<script setup lang="ts">
import { ref, computed } from 'vue'

// Typed refs
const count = ref<number>(0)
const user = ref<User | null>(null)

// Typed props
interface Props {
  title: string
  items: Item[]
  loading?: boolean
}
const props = withDefaults(defineProps<Props>(), { loading: false })

// Typed emits
const emit = defineEmits<{
  submit: [data: FormData]
  cancel: []
}>()

// Typed computed
const total = computed<number>(() => props.items.reduce((sum, i) => sum + i.price, 0))
</script>
```

```typescript
// Typed Pinia store
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'

interface User { id: number; name: string; email: string }

export const useUserStore = defineStore('user', () => {
  const user = ref<User | null>(null)
  const isAuthenticated = computed(() => user.value !== null)
  function setUser(newUser: User) { user.value = newUser }
  function logout() { user.value = null }
  return { user, isAuthenticated, setUser, logout }
})
```

### 8. 嚴格模式配置

```json
// tsconfig.json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true
  }
}
```

### 9. Async/Await with Types

```typescript
async function fetchUser(id: number): Promise<User> {
  const response = await fetch(`/api/users/${id}`);
  return response.json() as Promise<User>;
}

async function getUsers(): Promise<User[]> {
  try { return [await fetchUser(1)]; }
  catch { return []; }
}
```

---

## Quick Reference

### Vue 3 Lifecycle
`onBeforeMount` → `onMounted` → `onBeforeUpdate` → `onUpdated` → `onBeforeUnmount` → `onUnmounted`

### Reactivity API
`ref()` — 基本型別 | `reactive()` — 物件 | `computed()` — 計算屬性 | `watch()` / `watchEffect()` — 監聽

### Template Directives
`v-if/v-else/v-show` | `v-for` | `v-model` | `v-on/@` | `v-bind/:` | `v-slot/#`

### Quasar CLI
```bash
npm init quasar          # 建立專案
quasar dev               # 開發模式
quasar build             # 建構
quasar mode add pwa      # 加入 PWA 模式
```

### TypeScript Type Operations
```typescript
T | U         // Union
T & U         // Intersection
T extends U   // Constraint
keyof T       // Keys of T
typeof x      // Type of value
T[K]          // Index access
```

### Best Practices
1. 使用 Composition API + `<script setup>` 語法
2. Composables 提取可重用邏輯
3. TypeScript 嚴格模式
4. Pinia 按領域劃分 Store
5. Quasar Plugins 統一 UX (Notify, Loading)
6. Axios Interceptor 統一 API 錯誤處理
7. 懶載入路由元件
8. 組件命名 PascalCase, props camelCase, events kebab-case
