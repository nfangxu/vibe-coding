# Vue 3 + TypeScript + Tailwind CSS 开发规范

> 适用于 Vue 3 + TypeScript + Tailwind CSS + Vite 项目，覆盖组件开发、类型系统与样式三大核心领域。

---

## 一、Vue 3 组件规范

### 组件结构

- 始终使用 Composition API（`<script setup lang="ts">`）
- 组件保持小而专注，单一职责
- 模板中避免复杂逻辑，使用计算属性或方法代替

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
/* 仅在 Tailwind 无法满足需求时才编写自定义 CSS */
</style>
```

### 文件命名

- 组件文件使用 PascalCase：`UserProfile.vue`、`AppHeader.vue`
- 页面组件放在 `views/` 目录，通用组件放在 `components/` 目录
- 组件名与文件名保持一致

### Props 定义

- 始终使用 TypeScript 泛型定义 Props，使用 `withDefaults` 设置默认值
- Props 使用 camelCase 命名，模板中使用 kebab-case

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

### Composable 封装

- 将可复用逻辑封装为 Composable 函数（`use` 前缀）
- 放在 `composables/` 目录下
- 返回响应式状态和方法

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

### 模板规范

- 使用 `v-for` 时必须提供 `:key`，优先使用唯一 ID 而非 index
- `v-if` 和 `v-for` 不在同一元素上使用（`v-if` 优先级更高）
- 使用 `<template>` 包裹多个元素避免不必要的 DOM 节点
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

### 性能优化

- 使用 `defineAsyncComponent` 懒加载大型组件
- 使用 `v-memo` 缓存静态列表项
- 使用 `shallowRef` / `shallowReactive` 优化大型不可变数据
- 合理使用 `computed` 缓存计算结果，避免模板中的复杂表达式
- 合理使用 `v-show` 与 `v-if`：频繁切换用 `v-show`，条件渲染用 `v-if`

---

## 二、TypeScript 规范

### 类型系统

- 优先使用 `interface` 定义对象类型，`type` 定义联合/交叉/映射类型
- 避免使用 `any`，未知类型使用 `unknown` + 类型守卫
- 开启严格模式：`"strict": true`
- 合理使用 TypeScript 内置工具类型（`Partial`、`Required`、`Omit`、`Pick` 等）
- 使用泛型实现可复用的类型模式

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

### 命名规范

- 类型名与接口名使用 PascalCase
- 变量与函数使用 camelCase
- 常量使用 UPPER_CASE
- 布尔值使用描述性辅助动词前缀：`isLoading`、`hasError`、`canSubmit`

### 函数规范

- 公共函数使用显式返回类型
- 回调与方法使用箭头函数
- 优先使用 `async/await` 而非 Promise 链

```typescript
// 显式返回类型 + async/await
async function fetchUser(id: string): Promise<User> {
  const response = await request.get<User>(`/users/${id}`)
  return response.data
}
```

### 错误处理

- 为领域特定错误创建自定义错误类型
- 使用类型化的 catch 子句
- 正确处理 Promise rejection

```typescript
interface ApiError {
  code: string
  message: string
  details?: unknown
}

function isApiError(error: unknown): error is ApiError {
  return typeof error === 'object' && error !== null && 'code' in error
}

try {
  await fetchUser(id)
} catch (err) {
  if (isApiError(err)) {
    console.error(`API 错误 [${err.code}]: ${err.message}`)
  }
}
```

### 代码组织

- 将类型定义放在靠近使用它们的地方
- 共享类型放在 `types/` 目录
- 使用 barrel exports（`index.ts`）组织导出

---

## 三、Tailwind CSS 规范

### 项目配置

```typescript
// tailwind.config.ts
import type { Config } from 'tailwindcss'

export default {
  content: ['./index.html', './src/**/*.{vue,ts,tsx}'],
  darkMode: 'class',
  theme: {
    extend: {
      colors: {
        primary: {
          50: '#eff6ff',
          500: '#3b82f6',
          600: '#2563eb',
          900: '#1e3a8a',
        },
      },
    },
  },
  plugins: [],
} satisfies Config
```

### 组件样式

- **优先使用工具类**，避免编写自定义 CSS
- 需要组合多个工具类时，使用 `@apply`
- 保持工具类的书写顺序一致（布局 → 尺寸 → 间距 → 颜色 → 字体 → 其他）

```vue
<template>
  <!-- 推荐：工具类直接内联 -->
  <div class="flex items-center gap-4 rounded-lg bg-white p-4 shadow-sm dark:bg-gray-800">
    <img class="h-12 w-12 rounded-full object-cover" :src="user.avatar" :alt="user.name" />
    <div class="flex-1 min-w-0">
      <h3 class="truncate text-base font-semibold text-gray-900 dark:text-white">
        {{ user.name }}
      </h3>
      <p class="truncate text-sm text-gray-500 dark:text-gray-400">{{ user.email }}</p>
    </div>
  </div>
</template>

<style scoped>
/* 需要复用时使用 @apply */
.btn-primary {
  @apply rounded-md bg-blue-500 px-4 py-2 text-sm font-medium text-white;
  @apply hover:bg-blue-600 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2;
  @apply disabled:cursor-not-allowed disabled:opacity-50;
}
</style>
```

### 响应式设计

- 采用移动优先策略（先写无前缀，再用 `sm:` `md:` `lg:` `xl:` 覆盖）
- 常用断点：`sm` (640px)、`md` (768px)、`lg` (1024px)、`xl` (1280px)

```vue
<template>
  <div class="grid grid-cols-1 gap-4 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4">
    <UserCard v-for="user in users" :key="user.id" :user="user" />
  </div>
</template>
```

### 深色模式

```vue
<template>
  <div class="bg-white text-gray-900 dark:bg-gray-900 dark:text-white">
    <p class="text-gray-600 dark:text-gray-300">内容区域</p>
  </div>
</template>
```

### 表单样式

```vue
<template>
  <form class="space-y-4" @submit.prevent="handleSubmit">
    <div>
      <label class="mb-1 block text-sm font-medium text-gray-700 dark:text-gray-300">
        用户名
      </label>
      <input
        v-model="form.name"
        type="text"
        class="w-full rounded-md border border-gray-300 px-3 py-2 text-sm shadow-sm focus:border-blue-500 focus:outline-none focus:ring-1 focus:ring-blue-500 dark:border-gray-600 dark:bg-gray-700 dark:text-white"
        placeholder="请输入用户名"
      />
    </div>
    <button
      type="submit"
      class="w-full rounded-md bg-blue-500 py-2 text-sm font-semibold text-white hover:bg-blue-600 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2 disabled:opacity-50"
      :disabled="isSubmitting"
    >
      {{ isSubmitting ? '提交中...' : '提交' }}
    </button>
  </form>
</template>
```

### 组件变体模式

```typescript
// 使用对象映射管理组件变体
const buttonVariants = {
  primary: 'bg-blue-500 text-white hover:bg-blue-600',
  secondary: 'bg-gray-100 text-gray-900 hover:bg-gray-200',
  danger: 'bg-red-500 text-white hover:bg-red-600',
  ghost: 'bg-transparent text-gray-700 hover:bg-gray-100',
} as const

type ButtonVariant = keyof typeof buttonVariants
```

---

## 四、综合示例

```vue
<!-- components/UserCard.vue -->
<script setup lang="ts">
import { computed } from 'vue'

interface User {
  id: string
  name: string
  email: string
  role: 'admin' | 'user' | 'guest'
  avatar?: string
}

interface Props {
  user: User
  showActions?: boolean
}

const props = withDefaults(defineProps<Props>(), {
  showActions: true,
})

const emit = defineEmits<{
  edit: [user: User]
  delete: [userId: string]
}>()

const roleBadgeClass = computed(() => {
  const map: Record<User['role'], string> = {
    admin: 'bg-purple-100 text-purple-800',
    user: 'bg-blue-100 text-blue-800',
    guest: 'bg-gray-100 text-gray-800',
  }
  return map[props.user.role]
})

const roleLabel = computed(() => {
  const map: Record<User['role'], string> = {
    admin: '管理员',
    user: '用户',
    guest: '访客',
  }
  return map[props.user.role]
})
</script>

<template>
  <div class="flex items-center gap-4 rounded-lg border border-gray-200 bg-white p-4 shadow-sm transition-shadow hover:shadow-md dark:border-gray-700 dark:bg-gray-800">
    <img
      class="h-12 w-12 rounded-full object-cover"
      :src="user.avatar ?? `https://ui-avatars.com/api/?name=${user.name}`"
      :alt="user.name"
    />
    <div class="flex-1 min-w-0">
      <div class="flex items-center gap-2">
        <h3 class="truncate text-sm font-semibold text-gray-900 dark:text-white">
          {{ user.name }}
        </h3>
        <span :class="['rounded-full px-2 py-0.5 text-xs font-medium', roleBadgeClass]">
          {{ roleLabel }}
        </span>
      </div>
      <p class="truncate text-sm text-gray-500 dark:text-gray-400">{{ user.email }}</p>
    </div>
    <div v-if="showActions" class="flex gap-2">
      <button
        class="rounded-md px-3 py-1 text-xs font-medium text-blue-600 hover:bg-blue-50 dark:text-blue-400 dark:hover:bg-blue-900/20"
        @click="emit('edit', user)"
      >
        编辑
      </button>
      <button
        class="rounded-md px-3 py-1 text-xs font-medium text-red-600 hover:bg-red-50 dark:text-red-400 dark:hover:bg-red-900/20"
        @click="emit('delete', user.id)"
      >
        删除
      </button>
    </div>
  </div>
</template>
```

---

## 五、项目目录结构

```
src/
├── api/          # API 请求封装
├── assets/       # 静态资源
│   ├── images/
│   └── styles/
│       └── main.css   # Tailwind 入口（@tailwind base/components/utilities）
├── components/   # 通用组件
│   └── ui/       # 基础 UI 组件（基于 Tailwind）
├── composables/  # 组合式函数
├── layouts/      # 布局组件
├── router/       # 路由配置
├── stores/       # Pinia 状态管理
├── types/        # TypeScript 类型定义
├── utils/        # 工具函数
└── views/        # 页面组件
```

---

## 六、推荐工具

| 工具 | 用途 |
|------|------|
| `vite` | 构建工具 |
| `vue-tsc` | TypeScript 类型检查 |
| `tailwindcss` | 样式框架 |
| `@tailwindcss/forms` | 表单样式重置插件 |
| `@tailwindcss/typography` | 富文本排版插件 |
| `eslint` + `@antfu/eslint-config` | 代码规范 |
| `pinia` | 状态管理 |
| `vue-router` | 路由管理 |
| `vueuse` | 常用 Composables 集合 |
| `vitest` | 单元测试 |
| `playwright` | E2E 测试 |
