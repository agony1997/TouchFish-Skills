---
name: vue-expert
description: "Vue 2/3 框架開發專家。當需要 Vue 組件開發、Composition API、Options API、Vue Router、Pinia/Vuex 狀態管理、響應式系統、效能優化等任務時觸發。關鍵字: Vue, Vue 2, Vue 3, Composition API, Options API, Vue Router, Pinia, Vuex, 組件開發, component, reactivity, state management, 響應式, 狀態管理"
---

# Vue 2/3 Framework Expert

## Overview

Vue 框架開發專家,專精於 Vue 2/3 全端開發、Composition API、Options API、路由管理、狀態管理等領域。

作為 Vue 專家,我將協助您:
- Vue 2 與 Vue 3 的開發和遷移
- Composition API 與 Options API 最佳實踐
- 組件設計模式和可重用性
- Vue Router 路由管理
- Pinia/Vuex 狀態管理
- 響應式系統深入理解
- 效能優化和最佳實踐

## When to use this skill

此技能會在以下情況自動觸發:

**觸發關鍵字**: Vue, Vue 2, Vue 3, Composition API, Options API, Vue Router, Pinia, Vuex, 組件開發, component, reactivity, state management

**觸發場景**:
- Vue 應用開發和架構設計
- 組件開發和組合
- 狀態管理方案選擇
- 路由配置和權限控制
- Vue 2 升級到 Vue 3
- 效能優化和問題排除

## Core Knowledge Areas

### 1. Vue 3 Composition API

Composition API 是 Vue 3 的核心特性,提供更好的程式碼組織和重用性。

```vue
<script setup>
import { ref, computed, watch, onMounted } from 'vue'

// 響應式狀態
const count = ref(0)
const doubleCount = computed(() => count.value * 2)

// 方法
function increment() {
  count.value++
}

// 監聽器
watch(count, (newValue, oldValue) => {
  console.log(`Count changed from ${oldValue} to ${newValue}`)
})

// 生命週期
onMounted(() => {
  console.log('Component mounted')
})
</script>

<template>
  <div>
    <p>Count: {{ count }}</p>
    <p>Double: {{ doubleCount }}</p>
    <button @click="increment">Increment</button>
  </div>
</template>
```

### 2. Vue 2 Options API

Options API 是 Vue 2 的標準寫法,仍在 Vue 3 中完全支援。

```vue
<script>
export default {
  data() {
    return {
      count: 0
    }
  },
  computed: {
    doubleCount() {
      return this.count * 2
    }
  },
  methods: {
    increment() {
      this.count++
    }
  },
  mounted() {
    console.log('Component mounted')
  }
}
</script>

<template>
  <div>
    <p>Count: {{ count }}</p>
    <p>Double: {{ doubleCount }}</p>
    <button @click="increment">Increment</button>
  </div>
</template>
```

### 3. 可重用的 Composables

Composables 是 Composition API 的重用邏輯模式。

```javascript
// composables/useCounter.js
import { ref, computed } from 'vue'

export function useCounter(initialValue = 0) {
  const count = ref(initialValue)
  const doubleCount = computed(() => count.value * 2)

  function increment() {
    count.value++
  }

  function decrement() {
    count.value--
  }

  function reset() {
    count.value = initialValue
  }

  return {
    count,
    doubleCount,
    increment,
    decrement,
    reset
  }
}

// 在組件中使用
<script setup>
import { useCounter } from '@/composables/useCounter'

const { count, doubleCount, increment } = useCounter(10)
</script>
```

### 4. Vue Router 路由管理

```javascript
// router/index.js
import { createRouter, createWebHistory } from 'vue-router'
import Home from '@/views/Home.vue'
import About from '@/views/About.vue'

const routes = [
  {
    path: '/',
    name: 'Home',
    component: Home
  },
  {
    path: '/about',
    name: 'About',
    component: About
  },
  {
    path: '/users/:id',
    name: 'UserDetail',
    component: () => import('@/views/UserDetail.vue'), // 懶載入
    props: true
  },
  {
    path: '/admin',
    component: () => import('@/layouts/AdminLayout.vue'),
    meta: { requiresAuth: true },
    children: [
      {
        path: 'dashboard',
        component: () => import('@/views/admin/Dashboard.vue')
      }
    ]
  }
]

const router = createRouter({
  history: createWebHistory(),
  routes
})

// 路由守衛
router.beforeEach((to, from, next) => {
  if (to.meta.requiresAuth && !isAuthenticated()) {
    next('/login')
  } else {
    next()
  }
})

export default router
```

### 5. Pinia 狀態管理 (Vue 3 推薦)

```javascript
// stores/user.js
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'

export const useUserStore = defineStore('user', () => {
  // State
  const user = ref(null)
  const token = ref('')

  // Getters
  const isAuthenticated = computed(() => !!token.value)
  const userName = computed(() => user.value?.name || 'Guest')

  // Actions
  async function login(credentials) {
    try {
      const response = await api.post('/login', credentials)
      token.value = response.data.token
      user.value = response.data.user
      localStorage.setItem('token', token.value)
    } catch (error) {
      console.error('Login failed:', error)
      throw error
    }
  }

  function logout() {
    token.value = ''
    user.value = null
    localStorage.removeItem('token')
  }

  return {
    user,
    token,
    isAuthenticated,
    userName,
    login,
    logout
  }
})

// 在組件中使用
<script setup>
import { useUserStore } from '@/stores/user'

const userStore = useUserStore()

async function handleLogin() {
  await userStore.login({ email, password })
}
</script>
```

### 6. Vuex 狀態管理 (Vue 2/3)

```javascript
// store/index.js
import { createStore } from 'vuex'

export default createStore({
  state: {
    count: 0,
    user: null
  },
  mutations: {
    INCREMENT(state) {
      state.count++
    },
    SET_USER(state, user) {
      state.user = user
    }
  },
  actions: {
    async fetchUser({ commit }, userId) {
      const user = await api.getUser(userId)
      commit('SET_USER', user)
    }
  },
  getters: {
    doubleCount: state => state.count * 2,
    userName: state => state.user?.name || 'Guest'
  }
})
```

## Common Patterns

### 1. 表單處理

```vue
<script setup>
import { ref, reactive } from 'vue'

const form = reactive({
  name: '',
  email: '',
  password: ''
})

const errors = ref({})

async function handleSubmit() {
  errors.value = {}

  // 驗證
  if (!form.name) errors.value.name = '請輸入姓名'
  if (!form.email) errors.value.email = '請輸入電子郵件'

  if (Object.keys(errors.value).length > 0) return

  try {
    await api.submit(form)
    // 成功處理
  } catch (error) {
    errors.value.submit = error.message
  }
}
</script>

<template>
  <form @submit.prevent="handleSubmit">
    <input v-model="form.name" type="text">
    <span v-if="errors.name">{{ errors.name }}</span>

    <input v-model="form.email" type="email">
    <span v-if="errors.email">{{ errors.email }}</span>

    <button type="submit">提交</button>
  </form>
</template>
```

### 2. API 請求與載入狀態

```vue
<script setup>
import { ref, onMounted } from 'vue'

const data = ref(null)
const loading = ref(false)
const error = ref(null)

async function fetchData() {
  loading.value = true
  error.value = null

  try {
    const response = await fetch('/api/data')
    data.value = await response.json()
  } catch (err) {
    error.value = err.message
  } finally {
    loading.value = false
  }
}

onMounted(() => {
  fetchData()
})
</script>

<template>
  <div>
    <div v-if="loading">Loading...</div>
    <div v-else-if="error">Error: {{ error }}</div>
    <div v-else-if="data">
      <!-- 顯示資料 -->
    </div>
  </div>
</template>
```

### 3. 組件通訊 (Props & Emit)

```vue
<!-- 父組件 -->
<script setup>
import { ref } from 'vue'
import ChildComponent from './ChildComponent.vue'

const message = ref('Hello')

function handleUpdate(newValue) {
  message.value = newValue
}
</script>

<template>
  <ChildComponent
    :message="message"
    @update="handleUpdate"
  />
</template>

<!-- 子組件 -->
<script setup>
const props = defineProps({
  message: String
})

const emit = defineEmits(['update'])

function updateMessage() {
  emit('update', 'New message')
}
</script>

<template>
  <div>
    <p>{{ message }}</p>
    <button @click="updateMessage">Update</button>
  </div>
</template>
```

## Best Practices

### 開發規範

1. **使用 Composition API** (Vue 3)
   - 更好的程式碼組織
   - 更容易重用邏輯
   - TypeScript 支援更好

2. **組件設計原則**
   - 單一職責原則
   - Props down, Events up
   - 適當的組件拆分
   - 可重用的 Composables

3. **命名規範**
   - 組件名稱使用 PascalCase
   - Props 使用 camelCase
   - Event 使用 kebab-case

### 效能優化

1. **使用 v-show vs v-if**
   - v-if: 條件渲染,DOM 會被移除
   - v-show: 只是切換 display,適合頻繁切換

2. **懶載入路由組件**
   ```javascript
   component: () => import('@/views/About.vue')
   ```

3. **使用 computed 而非 methods**
   - computed 有快取機制
   - 只在相依的響應式資料改變時重新計算

4. **避免不必要的響應式**
   - 使用 shallowRef/shallowReactive 處理大型物件
   - 使用 markRaw 標記不需要響應式的物件

### 狀態管理

1. **選擇適當的狀態管理方案**
   - 小型應用: provide/inject
   - 中型應用: Pinia
   - 大型應用: Pinia + 模組化

2. **避免過度使用全域狀態**
   - 僅存放真正需要共享的狀態
   - 組件內部狀態應保持在組件內

## Troubleshooting

### 常見問題

1. **響應式失效**
   - 問題: 物件新增屬性沒有響應式
   - 解決: 使用 reactive() 或整體替換物件

2. **Memory Leak**
   - 問題: 組件銷毀後仍有事件監聽
   - 解決: 在 onBeforeUnmount 中清理

3. **Hydration Mismatch** (SSR)
   - 問題: 伺服器端和客戶端渲染不一致
   - 解決: 確保初始狀態相同,使用 <ClientOnly>

## Quick Reference

### Composition API Lifecycle

- `onBeforeMount`
- `onMounted`
- `onBeforeUpdate`
- `onUpdated`
- `onBeforeUnmount`
- `onUnmounted`

### Reactivity API

- `ref()` - 基本型別響應式
- `reactive()` - 物件響應式
- `computed()` - 計算屬性
- `watch()` - 監聽器
- `watchEffect()` - 自動追蹤依賴

### Template Directives

- `v-if` / `v-else` / `v-else-if`
- `v-show`
- `v-for`
- `v-model`
- `v-on` / `@`
- `v-bind` / `:`

---

**專家提示**: Vue 3 的 Composition API 提供了更好的程式碼組織和重用性,建議新專案直接使用 Composition API,並善用 Composables 模式來提取可重用邏輯。
