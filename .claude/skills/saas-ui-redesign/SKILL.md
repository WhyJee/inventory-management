---
name: saas-ui-redesign
description: >
  Redesigns this Vue 3 app's UI into a modern SaaS-style layout: replaces the
  horizontal top nav bar with a dark vertical sidebar on the left, moves
  LanguageSwitcher and ProfileMenu into the sidebar footer, and updates the
  FilterBar sticky offset. Use when the user asks to redesign the UI, switch
  to a sidebar layout, modernize the navigation, or apply a SaaS-style look.
allowed-tools: [Read, Edit, Write, Agent, Bash]
---

# SaaS UI Redesign Skill

Converts this app's horizontal top-nav layout into a modern SaaS-style interface with a dark vertical sidebar.

## Target Layout

```
┌──────────────────┬──────────────────────────────────────┐
│                  │                                        │
│  SIDEBAR (240px) │  FilterBar (sticky top: 0)            │
│  [Logo/Subtitle] │──────────────────────────────────────│
│  ─────────────── │                                        │
│  ● Overview      │  <router-view />                      │
│    Inventory     │                                        │
│    Orders        │                                        │
│    Finance       │                                        │
│    Demand        │                                        │
│    Reports       │                                        │
│  ─────────────── │                                        │
│  [Lang] [User]   │                                        │
└──────────────────┴──────────────────────────────────────┘
Sidebar: #0f172a bg — Active nav item: blue left border + #1e293b bg
```

---

## Step 1 — Read Current Structure

Before making any changes, read these three files:

1. `client/src/App.vue` — current template, script, and global CSS
2. `client/src/components/FilterBar.vue` — contains `top: 70px` sticky offset to fix
3. `client/src/main.js` — router routes and nav label i18n keys

Verify the nav link i18n keys (`t('nav.overview')`, `t('nav.inventory')`, etc.) and exact `$route.path` values used for active state before writing the new template.

---

## Step 2 — Delegate .vue Changes to vue-expert

Per project CLAUDE.md rules, **any significant modification to a `.vue` file must be delegated to vue-expert.**

Delegate **both** file changes in a single vue-expert call. Provide the agent with:
- The full current content of `App.vue` and `FilterBar.vue`
- The exact new template, script, and CSS for `App.vue` from Step 3 below
- The exact single-line CSS fix for `FilterBar.vue` from Step 4 below

---

## Step 3 — App.vue Changes

### Template

Replace the entire `<header class="top-nav">...</header>` block and the `<FilterBar />` and `<main>` structure with this new layout. Keep the modal components (`<ProfileDetailsModal>`, `<TasksModal>`) at the root level, unchanged.

```html
<template>
  <div class="app">
    <aside class="sidebar">
      <div class="sidebar-brand">
        <div class="sidebar-logo">{{ t('nav.companyName') }}</div>
        <div class="sidebar-subtitle">{{ t('nav.subtitle') }}</div>
      </div>
      <nav class="sidebar-nav">
        <router-link to="/" :class="{ active: $route.path === '/' }">
          <span class="nav-icon">⊞</span>{{ t('nav.overview') }}
        </router-link>
        <router-link to="/inventory" :class="{ active: $route.path === '/inventory' }">
          <span class="nav-icon">☰</span>{{ t('nav.inventory') }}
        </router-link>
        <router-link to="/orders" :class="{ active: $route.path === '/orders' }">
          <span class="nav-icon">◈</span>{{ t('nav.orders') }}
        </router-link>
        <router-link to="/spending" :class="{ active: $route.path === '/spending' }">
          <span class="nav-icon">$</span>{{ t('nav.finance') }}
        </router-link>
        <router-link to="/demand" :class="{ active: $route.path === '/demand' }">
          <span class="nav-icon">↗</span>{{ t('nav.demandForecast') }}
        </router-link>
        <router-link to="/reports" :class="{ active: $route.path === '/reports' }">
          <span class="nav-icon">▤</span>Reports
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
```

### Script

**No changes to the `<script>` block.** All imports, `setup()` logic, and `return` stay exactly as they are.

### Global CSS

Replace the layout-specific rules listed below. **Do not touch any shared utility classes** (`.badge`, `.card`, `.card-header`, `.card-title`, `.stat-card`, `.table-container`, `table`, `thead`, `th`, `td`, `.loading`, `.error`, `.page-header`, `.stats-grid`) — those must remain identical.

**Remove these old top-nav rules entirely:**
- `.top-nav`
- `.nav-container`
- `.nav-container > .nav-tabs`
- `.nav-container > .language-switcher`
- `.logo`
- `.subtitle`
- `.nav-tabs`
- `.nav-tabs a`
- `.nav-tabs a:hover`
- `.nav-tabs a.active`
- `.nav-tabs a.active::after`

**Replace `.app` and `.main-content` rules, and add new sidebar rules:**

```css
.app {
  display: flex;
  flex-direction: row;
  min-height: 100vh;
}

.sidebar {
  width: 240px;
  min-width: 240px;
  background: #0f172a;
  display: flex;
  flex-direction: column;
  position: sticky;
  top: 0;
  height: 100vh;
  overflow-y: auto;
  border-right: 1px solid #1e293b;
}

.sidebar-brand {
  padding: 1.5rem 1.25rem 1rem;
  border-bottom: 1px solid #1e293b;
}

.sidebar-logo {
  font-size: 1rem;
  font-weight: 700;
  color: #f1f5f9;
  letter-spacing: -0.025em;
}

.sidebar-subtitle {
  font-size: 0.75rem;
  color: #475569;
  margin-top: 0.25rem;
}

.sidebar-nav {
  flex: 1;
  padding: 0.75rem;
  display: flex;
  flex-direction: column;
  gap: 0.125rem;
}

.sidebar-nav a {
  display: flex;
  align-items: center;
  gap: 0.625rem;
  padding: 0.5rem 0.75rem;
  color: #94a3b8;
  text-decoration: none;
  font-size: 0.875rem;
  font-weight: 500;
  border-radius: 6px;
  transition: all 0.15s ease;
  border-left: 2px solid transparent;
}

.sidebar-nav a:hover {
  color: #e2e8f0;
  background: #1e293b;
}

.sidebar-nav a.active {
  color: #e2e8f0;
  background: #1e293b;
  border-left-color: #3b82f6;
}

.nav-icon {
  font-size: 0.875rem;
  width: 16px;
  text-align: center;
  flex-shrink: 0;
  opacity: 0.7;
}

.sidebar-footer {
  padding: 0.75rem;
  border-top: 1px solid #1e293b;
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.content-wrapper {
  flex: 1;
  display: flex;
  flex-direction: column;
  min-width: 0;
  background: #f8fafc;
}

.main-content {
  flex: 1;
  padding: 1.5rem 2rem;
}
```

---

## Step 4 — FilterBar.vue Change

In `FilterBar.vue`, in the `<style scoped>` block, find `.filters-bar` and change:

```css
/* Before */
top: 70px;

/* After */
top: 0;
```

**Why:** The old `top: 70px` offset was needed because the sticky filter bar sat below the 70px-tall top nav. The sidebar does not affect vertical scroll position, so the filter bar should now stick to the top of the viewport.

---

## Step 5 — Verify with Playwright

After the vue-expert agent completes both file changes:

1. Navigate to `http://localhost:3000` using `mcp__playwright__browser_navigate`
2. Take a screenshot to confirm the sidebar renders correctly
3. Check for:
   - Dark sidebar (240px) on the left with company name and subtitle
   - All 6 nav links visible
   - Active link has blue left border + slightly lighter background
   - FilterBar horizontal bar sits at the very top of the content area (no gap)
   - Main content fills the remaining width
   - Language switcher and profile menu visible in the sidebar footer
4. Click two or three nav links and confirm routing still works and active state updates
5. Change a filter dropdown value and confirm the content area updates — the filter system must still function

If the Vite dev server needs to be restarted after the changes (hot reload may not catch global CSS restructuring), run:
```bash
# Kill and restart the frontend dev server
pkill -f "vite" && cd client && npm run dev &
```
Then wait 2 seconds and re-navigate before taking the screenshot.
