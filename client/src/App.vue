<template>
  <div class="app">
    <aside class="sidebar">
      <div class="sidebar-brand">
        <div class="sidebar-logo">{{ t('nav.companyName') }}</div>
        <div class="sidebar-subtitle">{{ t('nav.subtitle') }}</div>
      </div>

      <nav class="sidebar-nav">
        <router-link to="/" :class="{ active: $route.path === '/' }">
          <svg class="nav-icon" viewBox="0 0 16 16" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round">
            <rect x="1" y="1" width="6" height="6" rx="1"/><rect x="9" y="1" width="6" height="6" rx="1"/>
            <rect x="1" y="9" width="6" height="6" rx="1"/><rect x="9" y="9" width="6" height="6" rx="1"/>
          </svg>
          {{ t('nav.overview') }}
        </router-link>

        <router-link to="/inventory" :class="{ active: $route.path === '/inventory' }">
          <svg class="nav-icon" viewBox="0 0 16 16" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round">
            <path d="M2 5l6-3 6 3v7l-6 3-6-3V5z"/><path d="M8 2v13"/><path d="M2 5l6 3 6-3"/>
          </svg>
          {{ t('nav.inventory') }}
        </router-link>

        <router-link to="/orders" :class="{ active: $route.path === '/orders' }">
          <svg class="nav-icon" viewBox="0 0 16 16" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round">
            <rect x="2" y="2" width="12" height="12" rx="1.5"/>
            <path d="M5 6h6M5 8.5h6M5 11h4"/>
          </svg>
          {{ t('nav.orders') }}
        </router-link>

        <router-link to="/spending" :class="{ active: $route.path === '/spending' }">
          <svg class="nav-icon" viewBox="0 0 16 16" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round">
            <circle cx="8" cy="8" r="6"/>
            <path d="M8 4.5v7M6 6.5c0-1 .9-1.5 2-1.5s2 .6 2 1.5-1 1.5-2 1.5-2 .5-2 1.5.9 1.5 2 1.5 2-.5 2-1.5"/>
          </svg>
          {{ t('nav.finance') }}
        </router-link>

        <router-link to="/demand" :class="{ active: $route.path === '/demand' }">
          <svg class="nav-icon" viewBox="0 0 16 16" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round">
            <polyline points="2,12 6,8 9,10 14,4"/>
            <polyline points="11,4 14,4 14,7"/>
          </svg>
          {{ t('nav.demandForecast') }}
        </router-link>

        <router-link to="/reports" :class="{ active: $route.path === '/reports' }">
          <svg class="nav-icon" viewBox="0 0 16 16" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round">
            <rect x="2" y="9" width="3" height="5" rx="0.5"/>
            <rect x="6.5" y="5" width="3" height="9" rx="0.5"/>
            <rect x="11" y="2" width="3" height="12" rx="0.5"/>
          </svg>
          Reports
        </router-link>
      </nav>

      <div class="sidebar-footer">
        <LanguageSwitcher />
        <ProfileMenu
          @show-profile-details="showProfileDetails = true"
          @show-tasks="showTasks = true"
        />
      </div>
    </aside>

    <div class="content-wrapper">
      <FilterBar />
      <main class="main-content">
        <router-view />
      </main>
    </div>

    <ProfileDetailsModal
      :is-open="showProfileDetails"
      @close="showProfileDetails = false"
    />

    <TasksModal
      :is-open="showTasks"
      :tasks="tasks"
      @close="showTasks = false"
      @add-task="addTask"
      @delete-task="deleteTask"
      @toggle-task="toggleTask"
    />
  </div>
</template>

<script>
import { ref, onMounted, computed } from 'vue'
import { api } from './api'
import { useAuth } from './composables/useAuth'
import { useI18n } from './composables/useI18n'
import FilterBar from './components/FilterBar.vue'
import ProfileMenu from './components/ProfileMenu.vue'
import ProfileDetailsModal from './components/ProfileDetailsModal.vue'
import TasksModal from './components/TasksModal.vue'
import LanguageSwitcher from './components/LanguageSwitcher.vue'

export default {
  name: 'App',
  components: {
    FilterBar,
    ProfileMenu,
    ProfileDetailsModal,
    TasksModal,
    LanguageSwitcher
  },
  setup() {
    const { currentUser } = useAuth()
    const { t } = useI18n()
    const showProfileDetails = ref(false)
    const showTasks = ref(false)
    const apiTasks = ref([])

    // Merge mock tasks from currentUser with API tasks
    const tasks = computed(() => {
      return [...currentUser.value.tasks, ...apiTasks.value]
    })

    const loadTasks = async () => {
      try {
        apiTasks.value = await api.getTasks()
      } catch (err) {
        console.error('Failed to load tasks:', err)
      }
    }

    const addTask = async (taskData) => {
      try {
        const newTask = await api.createTask(taskData)
        // Add new task to the beginning of the array
        apiTasks.value.unshift(newTask)
      } catch (err) {
        console.error('Failed to add task:', err)
      }
    }

    const deleteTask = async (taskId) => {
      try {
        // Check if it's a mock task (from currentUser)
        const isMockTask = currentUser.value.tasks.some(t => t.id === taskId)

        if (isMockTask) {
          // Remove from mock tasks
          const index = currentUser.value.tasks.findIndex(t => t.id === taskId)
          if (index !== -1) {
            currentUser.value.tasks.splice(index, 1)
          }
        } else {
          // Remove from API tasks
          await api.deleteTask(taskId)
          apiTasks.value = apiTasks.value.filter(t => t.id !== taskId)
        }
      } catch (err) {
        console.error('Failed to delete task:', err)
      }
    }

    const toggleTask = async (taskId) => {
      try {
        // Check if it's a mock task (from currentUser)
        const mockTask = currentUser.value.tasks.find(t => t.id === taskId)

        if (mockTask) {
          // Toggle mock task status
          mockTask.status = mockTask.status === 'pending' ? 'completed' : 'pending'
        } else {
          // Toggle API task
          const updatedTask = await api.toggleTask(taskId)
          const index = apiTasks.value.findIndex(t => t.id === taskId)
          if (index !== -1) {
            apiTasks.value[index] = updatedTask
          }
        }
      } catch (err) {
        console.error('Failed to toggle task:', err)
      }
    }

    onMounted(loadTasks)

    return {
      t,
      showProfileDetails,
      showTasks,
      tasks,
      addTask,
      deleteTask,
      toggleTask
    }
  }
}
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=DM+Sans:opsz,wght@9..40,400;9..40,500;9..40,600;9..40,700&family=JetBrains+Mono:wght@400;500&display=swap');

:root {
  --bg:         #0d1117;
  --bg-surface: #161b22;
  --bg-raised:  #1c2128;
  --border:     #30363d;
  --border-dim: #21262d;
  --text:       #e6edf3;
  --text-muted: #8b949e;
  --text-dim:   #6e7681;
  --accent:     #f97316;
  --accent-dim: rgba(249, 115, 22, 0.12);
  --success:    #3fb950;
  --success-bg: rgba(63, 185, 80, 0.12);
  --warning:    #d29922;
  --warning-bg: rgba(210, 153, 34, 0.12);
  --danger:     #f85149;
  --danger-bg:  rgba(248, 81, 73, 0.12);
  --info:       #58a6ff;
  --info-bg:    rgba(88, 166, 255, 0.12);
  --mono:       'JetBrains Mono', 'Fira Code', monospace;
}

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'DM Sans', -apple-system, BlinkMacSystemFont, sans-serif;
  background: var(--bg);
  color: var(--text);
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

/* ── App layout ── */
.app {
  display: flex;
  flex-direction: row;
  min-height: 100vh;
}

/* ── Sidebar ── */
.sidebar {
  width: 232px;
  min-width: 232px;
  background: var(--bg-surface);
  border-right: 1px solid var(--border);
  display: flex;
  flex-direction: column;
  position: sticky;
  top: 0;
  height: 100vh;
  overflow-y: auto;
  z-index: 100;
}

.sidebar-brand {
  padding: 1.375rem 1.25rem 1.125rem;
  border-bottom: 1px solid var(--border-dim);
}

.sidebar-logo {
  font-size: 0.9375rem;
  font-weight: 700;
  color: var(--text);
  letter-spacing: -0.02em;
  line-height: 1.2;
}

.sidebar-subtitle {
  font-size: 0.6875rem;
  color: var(--text-dim);
  margin-top: 0.25rem;
  letter-spacing: 0.02em;
  text-transform: uppercase;
}

.sidebar-nav {
  flex: 1;
  padding: 0.875rem 0.75rem;
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.sidebar-nav a {
  display: flex;
  align-items: center;
  gap: 0.625rem;
  padding: 0.5rem 0.75rem;
  color: var(--text-muted);
  text-decoration: none;
  font-size: 0.875rem;
  font-weight: 500;
  border-radius: 5px;
  transition: color 0.15s ease, background 0.15s ease;
  border-left: 2px solid transparent;
}

.sidebar-nav a:hover {
  color: var(--text);
  background: var(--bg-raised);
}

.sidebar-nav a.active {
  color: var(--accent);
  background: var(--accent-dim);
  border-left-color: var(--accent);
}

.nav-icon {
  width: 15px;
  height: 15px;
  flex-shrink: 0;
  opacity: 0.75;
  transition: opacity 0.15s ease;
}

.sidebar-nav a.active .nav-icon,
.sidebar-nav a:hover .nav-icon {
  opacity: 1;
}

.sidebar-footer {
  padding: 0.875rem 0.75rem;
  border-top: 1px solid var(--border-dim);
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

/* ── Content wrapper ── */
.content-wrapper {
  flex: 1;
  display: flex;
  flex-direction: column;
  min-width: 0;
}

.main-content {
  flex: 1;
  padding: 1.75rem 2rem;
  animation: fadeUp 0.18s ease both;
}

@keyframes fadeUp {
  from { opacity: 0; transform: translateY(6px); }
  to   { opacity: 1; transform: translateY(0); }
}

/* ── Page header ── */
.page-header {
  margin-bottom: 1.5rem;
}

.page-header h2 {
  font-size: 1.5rem;
  font-weight: 700;
  color: var(--text);
  margin-bottom: 0.25rem;
  letter-spacing: -0.03em;
}

.page-header p {
  color: var(--text-muted);
  font-size: 0.875rem;
}

/* ── Stat cards ── */
.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
  gap: 1rem;
  margin-bottom: 1.25rem;
}

.stat-card {
  background: var(--bg-surface);
  padding: 1.25rem;
  border-radius: 8px;
  border: 1px solid var(--border);
  transition: border-color 0.15s ease, box-shadow 0.15s ease;
}

.stat-card:hover {
  border-color: #484f58;
  box-shadow: 0 0 0 1px #30363d;
}

.stat-label {
  color: var(--text-dim);
  font-size: 0.6875rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.08em;
  margin-bottom: 0.625rem;
}

.stat-value {
  font-family: var(--mono);
  font-size: 2rem;
  font-weight: 500;
  color: var(--text);
  letter-spacing: -0.04em;
  line-height: 1;
}

.stat-card.warning .stat-value { color: var(--accent); }
.stat-card.success .stat-value { color: var(--success); }
.stat-card.danger  .stat-value { color: var(--danger); }
.stat-card.info    .stat-value { color: var(--info); }

/* ── Cards ── */
.card {
  background: var(--bg-surface);
  border-radius: 8px;
  padding: 1.25rem;
  border: 1px solid var(--border);
  margin-bottom: 1.25rem;
  transition: border-color 0.15s ease;
}

.card:hover {
  border-color: #484f58;
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
  padding-bottom: 0.75rem;
  border-bottom: 1px solid var(--border-dim);
}

.card-title {
  font-size: 0.9375rem;
  font-weight: 600;
  color: var(--text);
  letter-spacing: -0.02em;
}

/* ── Tables ── */
.table-container {
  overflow-x: auto;
}

table {
  width: 100%;
  border-collapse: collapse;
}

thead {
  background: var(--bg-raised);
  border-top: 1px solid var(--border-dim);
  border-bottom: 1px solid var(--border-dim);
}

th {
  text-align: left;
  padding: 0.5rem 0.875rem;
  font-weight: 600;
  color: var(--text-dim);
  font-size: 0.6875rem;
  text-transform: uppercase;
  letter-spacing: 0.08em;
}

td {
  padding: 0.5625rem 0.875rem;
  border-top: 1px solid var(--border-dim);
  color: var(--text-muted);
  font-size: 0.8125rem;
  font-family: var(--mono);
}

tbody tr {
  transition: background-color 0.12s ease;
}

tbody tr:hover {
  background: var(--bg-raised);
}

/* ── Badges ── */
.badge {
  display: inline-block;
  padding: 0.25rem 0.625rem;
  border-radius: 4px;
  font-size: 0.6875rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.06em;
  font-family: var(--mono);
  border: 1px solid transparent;
}

.badge.success    { background: var(--success-bg); color: var(--success); border-color: rgba(63, 185, 80, 0.25); }
.badge.warning    { background: var(--warning-bg); color: var(--warning); border-color: rgba(210, 153, 34, 0.25); }
.badge.danger     { background: var(--danger-bg);  color: var(--danger);  border-color: rgba(248, 81, 73, 0.25); }
.badge.info       { background: var(--info-bg);    color: var(--info);    border-color: rgba(88, 166, 255, 0.25); }
.badge.increasing { background: var(--success-bg); color: var(--success); border-color: rgba(63, 185, 80, 0.25); }
.badge.decreasing { background: var(--danger-bg);  color: var(--danger);  border-color: rgba(248, 81, 73, 0.25); }
.badge.stable     { background: var(--info-bg);    color: var(--info);    border-color: rgba(88, 166, 255, 0.25); }
.badge.high       { background: var(--danger-bg);  color: var(--danger);  border-color: rgba(248, 81, 73, 0.25); }
.badge.medium     { background: var(--warning-bg); color: var(--warning); border-color: rgba(210, 153, 34, 0.25); }
.badge.low        { background: var(--info-bg);    color: var(--info);    border-color: rgba(88, 166, 255, 0.25); }

/* ── States ── */
.loading {
  text-align: center;
  padding: 3rem;
  color: var(--text-dim);
  font-size: 0.875rem;
}

.error {
  background: var(--danger-bg);
  border: 1px solid rgba(248, 81, 73, 0.25);
  color: var(--danger);
  padding: 1rem;
  border-radius: 6px;
  margin: 1rem 0;
  font-size: 0.875rem;
}
</style>
