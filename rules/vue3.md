# Vue 3 开发规范

> 适用于 Vue 3 + TypeScript + Tailwind CSS + Vite 项目，在 AI 编辑器中使用时可将本文件内容配置为规则。

## 技术栈

- **框架**: Vue 3 (Composition API)
- **语言**: TypeScript（严格模式）
- **构建工具**: Vite
- **样式**: Tailwind CSS
- **状态管理**: Pinia
- **路由**: Vue Router 4
- **UI 组件**: shadcn/ui（Vue 版本）或其他基于 Tailwind 的组件库
- **HTTP 客户端**: Axios
- **代码规范**: ESLint + Prettier

## 组件规范

### 文件命名

- 组件文件使用 PascalCase：`UserProfile.vue`、`AppHeader.vue`
- 页面组件放在 `views/` 目录，通用组件放在 `components/` 目录
- 组件名与文件名保持一致

### 组件结构顺序

```vue
<script setup lang="ts">
// 1. 导入
// 2. Props / Emits 定义
// 3. 响应式状态
// 4. 计算属性
// 5. 方法
// 6. 生命周期钩子
// 7. Watch
</script>

<template>
  <!-- 模板内容 -->
</template>

<style scoped>
/* 样式 */
</style>
```

### Props 定义

```vue
<script setup lang="ts">
interface Props {
  userId: string
  userName?: string
  isActive?: boolean
  items: UserItem[]
}

const props = withDefaults(defineProps<Props>(), {
  userName: '',
  isActive: false,
  items: () => [],
})
</script>
```

### Emits 定义

```vue
<script setup lang="ts">
const emit = defineEmits<{
  change: [value: string]
  submit: [data: FormData]
  'update:modelValue': [value: number]
}>()
</script>
```

### 响应式状态

- 对象/数组使用 `reactive()`，基本类型使用 `ref()`
- 优先使用 `ref()` 保持一致性（Vue 官方推荐）
- 避免解构响应式对象（会失去响应性），使用 `toRefs()`

```vue
<script setup lang="ts">
import { ref, reactive, computed, toRefs } from 'vue'

const count = ref(0)
const userForm = reactive({
  name: '',
  email: '',
  age: 0,
})

const isFormValid = computed(() => {
  return userForm.name.length > 0 && userForm.email.includes('@')
})
</script>
```

## Composition API 最佳实践

### Composable 封装

```typescript
// composables/useUserList.ts
import { ref, onMounted } from 'vue'
import type { User } from '@/types'

export function useUserList() {
  const users = ref<User[]>([])
  const loading = ref(false)
  const error = ref<string | null>(null)

  async function fetchUsers() {
    loading.value = true
    error.value = null
    try {
      users.value = await userApi.getList()
    } catch (err) {
      error.value = err instanceof Error ? err.message : '获取用户列表失败'
    } finally {
      loading.value = false
    }
  }

  onMounted(fetchUsers)

  return { users, loading, error, fetchUsers }
}
```

### 生命周期钩子

```vue
<script setup lang="ts">
import { onMounted, onUnmounted } from 'vue'

let timer: ReturnType<typeof setInterval>

onMounted(() => {
  timer = setInterval(refresh, 5000)
})

onUnmounted(() => {
  clearInterval(timer)
})
</script>
```

## Pinia 状态管理

```typescript
// stores/useUserStore.ts
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'
import type { User } from '@/types'

export const useUserStore = defineStore('user', () => {
  const currentUser = ref<User | null>(null)
  const isLoggedIn = computed(() => currentUser.value !== null)

  async function login(credentials: LoginCredentials) {
    const user = await authApi.login(credentials)
    currentUser.value = user
  }

  function logout() {
    currentUser.value = null
  }

  return { currentUser, isLoggedIn, login, logout }
})
```

## Vue Router 规范

```typescript
// router/index.ts
const routes: RouteRecordRaw[] = [
  {
    path: '/',
    component: () => import('@/layouts/DefaultLayout.vue'),
    children: [
      {
        path: '',
        name: 'Home',
        component: () => import('@/views/HomePage.vue'),
        meta: { title: '首页', requiresAuth: false },
      },
      {
        path: 'user-profile',
        name: 'UserProfile',
        component: () => import('@/views/UserProfilePage.vue'),
        meta: { title: '用户资料', requiresAuth: true },
      },
    ],
  },
]

router.beforeEach((to) => {
  const userStore = useUserStore()
  if (to.meta.requiresAuth && !userStore.isLoggedIn) {
    return { name: 'Login', query: { redirect: to.fullPath } }
  }
})
```

## TypeScript 规范

```typescript
// types/user.ts
export interface User {
  id: string
  name: string
  email: string
  role: 'admin' | 'user' | 'guest'
  createdAt: string
}

export type CreateUserInput = Omit<User, 'id' | 'createdAt'>
export type UpdateUserInput = Partial<CreateUserInput>
```

## 模板规范

- 使用 `v-for` 时必须提供 `:key`，优先使用唯一 ID 而非 index
- `v-if` 和 `v-for` 不在同一元素上使用
- 事件处理函数在模板中不加括号（除非需要传参）

```vue
<template>
  <ul>
    <li v-for="user in users" :key="user.id">
      {{ user.name }}
    </li>
  </ul>

  <template v-if="isLoaded">
    <UserHeader :user="currentUser" />
    <UserContent :user="currentUser" />
  </template>
</template>
```

## 样式规范

- **默认使用 Tailwind CSS** 工具类，避免编写自定义 CSS
- 响应式设计采用移动优先（`sm:` `md:` `lg:` `xl:`）
- 使用 `@apply` 将常用工具类组合为组件级样式
- 组件样式使用 `scoped` 避免全局污染
- 深色模式使用 `dark:` 变体

```vue
<template>
  <div class="flex items-center gap-4 rounded-lg bg-white p-4 shadow-sm dark:bg-gray-800">
    <img class="h-12 w-12 rounded-full object-cover" :src="user.avatar" :alt="user.name" />
    <div class="flex-1">
      <h3 class="text-base font-semibold text-gray-900 dark:text-white">{{ user.name }}</h3>
      <p class="text-sm text-gray-500 dark:text-gray-400">{{ user.email }}</p>
    </div>
    <button
      class="rounded-md bg-blue-500 px-3 py-1.5 text-sm font-medium text-white hover:bg-blue-600 focus:outline-none focus:ring-2 focus:ring-blue-500"
      @click="handleAction"
    >
      操作
    </button>
  </div>
</template>

<style scoped>
/* 仅在 Tailwind 无法满足需求时才编写自定义 CSS */
.user-card {
  @apply flex items-center gap-4 rounded-lg bg-white p-4 shadow-sm;
}
</style>
```

## 项目目录结构

```
src/
├── api/          # API 请求封装
├── assets/       # 静态资源
│   ├── images/
│   └── styles/
├── components/   # 通用组件
├── composables/  # 组合式函数
├── layouts/      # 布局组件
├── router/       # 路由配置
├── stores/       # Pinia 状态管理
├── types/        # TypeScript 类型定义
├── utils/        # 工具函数
└── views/        # 页面组件
```

## 推荐工具

| 工具 | 用途 |
|------|------|
| `vite` | 构建工具 |
| `vue-tsc` | TypeScript 类型检查 |
| `eslint` + `@antfu/eslint-config` | 代码规范 |
| `pinia` | 状态管理 |
| `vue-router` | 路由管理 |
| `vueuse` | 常用 Composables 集合 |
| `vitest` | 单元测试 |
| `playwright` | E2E 测试 |
