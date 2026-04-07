<template>
  <div class="app">

    <!-- Sidebar -->
    <aside class="sidebar" :class="{ collapsed: sidebarCollapsed, 'mobile-open': mobileOpen }">

      <!-- Brand -->
      <div class="sidebar-brand">
        <div class="brand-mark">F</div>
        <div class="brand-text" v-show="!sidebarCollapsed">
          <span class="brand-name">{{ t('nav.companyName') }}</span>
          <span class="brand-sub">{{ t('nav.subtitle') }}</span>
        </div>
      </div>

      <!-- Nav -->
      <nav class="sidebar-nav">
        <router-link
          v-for="item in navItems"
          :key="item.path"
          :to="item.path"
          class="nav-item"
          :class="{ active: $route.path === item.path }"
          :title="sidebarCollapsed ? (item.label || t(item.labelKey)) : ''"
          @click="isMobile && (mobileOpen = false)"
        >
          <!-- Icon slot — inline SVG per route -->
          <span class="nav-icon" v-html="getNavIcon(item.path)"></span>
          <span class="nav-label" v-show="!sidebarCollapsed">
            {{ item.label || t(item.labelKey) }}
          </span>
        </router-link>
      </nav>

      <!-- Bottom: language, profile, collapse -->
      <div class="sidebar-bottom">
        <div class="sidebar-utils" v-show="!sidebarCollapsed">
          <LanguageSwitcher />
        </div>
        <button
          class="sidebar-profile-btn"
          @click="showProfileDetails = true"
          :title="sidebarCollapsed ? 'Profile' : ''"
        >
          <div class="avatar">{{ t('nav.companyName').charAt(0) }}</div>
          <span class="profile-label" v-show="!sidebarCollapsed">Profile &amp; Settings</span>
        </button>
        <button class="collapse-btn" @click="sidebarCollapsed = !sidebarCollapsed">
          <svg width="16" height="16" viewBox="0 0 16 16" fill="none">
            <path d="M10 12L6 8L10 4" stroke="currentColor" stroke-width="1.75" stroke-linecap="round" stroke-linejoin="round"/>
          </svg>
          <span v-show="!sidebarCollapsed" style="font-size:11px;margin-left:6px;">Collapse</span>
        </button>
      </div>
    </aside>

    <!-- Main area -->
    <div class="content-shell" :class="{ expanded: sidebarCollapsed }">
      <!-- Top utility bar -->
      <div class="top-utility-bar">
        <button v-if="isMobile" class="hamburger-btn" @click="mobileOpen = !mobileOpen" aria-label="Toggle menu">
          <svg width="18" height="18" viewBox="0 0 18 18" fill="none">
            <path d="M2 4.5h14M2 9h14M2 13.5h14" stroke="currentColor" stroke-width="1.75" stroke-linecap="round"/>
          </svg>
        </button>
        <FilterBar />
        <button class="tasks-btn" @click="showTasks = true">
          <svg width="16" height="16" viewBox="0 0 16 16" fill="none">
            <path d="M2 4h12M2 8h8M2 12h10" stroke="currentColor" stroke-width="1.75" stroke-linecap="round"/>
          </svg>
          <span>Tasks</span>
        </button>
      </div>

      <main class="main-content">
        <router-view />
      </main>
    </div>

    <!-- Mobile overlay backdrop -->
    <Transition name="fade">
      <div
        v-if="isMobile && mobileOpen"
        class="sidebar-overlay"
        @click="mobileOpen = false"
      ></div>
    </Transition>

    <!-- Modals -->
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
import { ref, onMounted, onUnmounted, computed, watch } from 'vue'
import { useRoute } from 'vue-router'
import { useAuth } from './composables/useAuth'
import { useI18n } from './composables/useI18n'
import FilterBar from './components/FilterBar.vue'
import ProfileDetailsModal from './components/ProfileDetailsModal.vue'
import TasksModal from './components/TasksModal.vue'
import LanguageSwitcher from './components/LanguageSwitcher.vue'

export default {
  name: 'App',
  components: {
    FilterBar,
    ProfileDetailsModal,
    TasksModal,
    LanguageSwitcher
  },
  setup() {
    const { currentUser } = useAuth()
    const { t } = useI18n()
    const showProfileDetails = ref(false)
    const showTasks = ref(false)
    const sidebarCollapsed = ref(false)
    const mobileOpen = ref(false)
    const isMobile = ref(false)

    const navItems = [
      { path: '/', labelKey: 'nav.overview' },
      { path: '/inventory', labelKey: 'nav.inventory' },
      { path: '/orders', labelKey: 'nav.orders' },
      { path: '/spending', labelKey: 'nav.finance' },
      { path: '/demand', labelKey: 'nav.demandForecast' },
      { path: '/reports', label: 'Reports' },
      { path: '/restocking', label: 'Restocking' },
    ]

    const getNavIcon = (path) => {
      const icons = {
        '/': `<svg width="18" height="18" viewBox="0 0 18 18" fill="none" xmlns="http://www.w3.org/2000/svg"><rect x="2" y="2" width="6" height="6" rx="1.5" stroke="currentColor" stroke-width="1.75"/><rect x="10" y="2" width="6" height="6" rx="1.5" stroke="currentColor" stroke-width="1.75"/><rect x="2" y="10" width="6" height="6" rx="1.5" stroke="currentColor" stroke-width="1.75"/><rect x="10" y="10" width="6" height="6" rx="1.5" stroke="currentColor" stroke-width="1.75"/></svg>`,
        '/inventory': `<svg width="18" height="18" viewBox="0 0 18 18" fill="none" xmlns="http://www.w3.org/2000/svg"><path d="M9 2L16 5.5V12.5L9 16L2 12.5V5.5L9 2Z" stroke="currentColor" stroke-width="1.75" stroke-linejoin="round"/><path d="M9 2V16M2 5.5L9 9L16 5.5" stroke="currentColor" stroke-width="1.75"/></svg>`,
        '/orders': `<svg width="18" height="18" viewBox="0 0 18 18" fill="none" xmlns="http://www.w3.org/2000/svg"><rect x="3" y="2" width="12" height="14" rx="2" stroke="currentColor" stroke-width="1.75"/><path d="M6 6h6M6 9h6M6 12h4" stroke="currentColor" stroke-width="1.75" stroke-linecap="round"/></svg>`,
        '/spending': `<svg width="18" height="18" viewBox="0 0 18 18" fill="none" xmlns="http://www.w3.org/2000/svg"><path d="M3 14V10M7 14V7M11 14V9M15 14V5" stroke="currentColor" stroke-width="1.75" stroke-linecap="round"/></svg>`,
        '/demand': `<svg width="18" height="18" viewBox="0 0 18 18" fill="none" xmlns="http://www.w3.org/2000/svg"><path d="M2 13L6 9L9 11L14 5" stroke="currentColor" stroke-width="1.75" stroke-linecap="round" stroke-linejoin="round"/><path d="M11 5h3v3" stroke="currentColor" stroke-width="1.75" stroke-linecap="round" stroke-linejoin="round"/></svg>`,
        '/reports': `<svg width="18" height="18" viewBox="0 0 18 18" fill="none" xmlns="http://www.w3.org/2000/svg"><rect x="3" y="2" width="12" height="14" rx="2" stroke="currentColor" stroke-width="1.75"/><path d="M6 7h6M6 10h4" stroke="currentColor" stroke-width="1.75" stroke-linecap="round"/><path d="M6 13h2" stroke="currentColor" stroke-width="1.75" stroke-linecap="round"/></svg>`,
        '/restocking': `<svg width="18" height="18" viewBox="0 0 18 18" fill="none" xmlns="http://www.w3.org/2000/svg"><path d="M15 3v4h-4" stroke="currentColor" stroke-width="1.75" stroke-linecap="round" stroke-linejoin="round"/><path d="M3 9a6 6 0 0 1 10.2-4.2L15 7M3 15v-4h4" stroke="currentColor" stroke-width="1.75" stroke-linecap="round" stroke-linejoin="round"/><path d="M15 9a6 6 0 0 1-10.2 4.2L3 11" stroke="currentColor" stroke-width="1.75" stroke-linecap="round" stroke-linejoin="round"/></svg>`,
      }
      return icons[path] || icons['/']
    }

    const tasks = computed(() => currentUser.value.tasks)

    const addTask = (taskData) => {
      const newTask = {
        id: Date.now(),
        ...taskData,
        status: 'pending'
      }
      currentUser.value.tasks.unshift(newTask)
    }

    const deleteTask = (taskId) => {
      const index = currentUser.value.tasks.findIndex(t => t.id === taskId)
      if (index !== -1) {
        currentUser.value.tasks.splice(index, 1)
      }
    }

    const toggleTask = (taskId) => {
      const task = currentUser.value.tasks.find(t => t.id === taskId)
      if (task) {
        task.status = task.status === 'pending' ? 'completed' : 'pending'
      }
    }

    const handleResize = () => {
      const w = window.innerWidth
      if (w < 768) {
        isMobile.value = true
        // On mobile, sidebar is overlay — collapsed state doesn't apply
      } else {
        isMobile.value = false
        mobileOpen.value = false
        // Auto-collapse on tablet, but don't force-expand on desktop
        if (w < 1024) {
          sidebarCollapsed.value = true
        }
      }
    }

    onMounted(() => {
      handleResize() // set initial state
      window.addEventListener('resize', handleResize)
    })

    onUnmounted(() => {
      window.removeEventListener('resize', handleResize)
    })

    const route = useRoute()
    watch(() => route.path, () => {
      if (isMobile.value) mobileOpen.value = false
    })

    return {
      t,
      showProfileDetails,
      showTasks,
      tasks,
      addTask,
      deleteTask,
      toggleTask,
      sidebarCollapsed,
      mobileOpen,
      isMobile,
      navItems,
      getNavIcon,
    }
  }
}
</script>

<style>
/* ── Reset ── */
*, *::before, *::after {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

/* ── Design Tokens ── */
:root {
  --sidebar-w: 240px;
  --sidebar-w-collapsed: 64px;
  --sidebar-bg: #0f172a;
  --sidebar-border: rgba(255,255,255,0.06);
  --nav-hover-bg: rgba(255,255,255,0.07);
  --nav-active-bg: rgba(59,130,246,0.18);
  --nav-active-border: #3b82f6;
  --nav-text: rgba(255,255,255,0.6);
  --nav-text-hover: rgba(255,255,255,0.92);
  --nav-text-active: #ffffff;
  --content-bg: #f1f5f9;
  --surface: #ffffff;
  --border: #e2e8f0;
  --border-strong: #cbd5e1;
  --text-primary: #0f172a;
  --text-secondary: #475569;
  --text-muted: #94a3b8;
  --accent: #2563eb;
  --accent-light: #eff6ff;
  --accent-glow: rgba(37,99,235,0.15);
  --radius-sm: 6px;
  --radius-md: 10px;
  --radius-lg: 14px;
  --shadow-sm: 0 1px 3px rgba(0,0,0,0.07), 0 1px 2px rgba(0,0,0,0.04);
  --shadow-md: 0 4px 16px rgba(0,0,0,0.08), 0 2px 6px rgba(0,0,0,0.05);
  --shadow-lg: 0 8px 32px rgba(0,0,0,0.12), 0 4px 12px rgba(0,0,0,0.06);
  --transition: 0.2s ease;
}

/* ── Base ── */
body {
  font-family: 'Plus Jakarta Sans', -apple-system, BlinkMacSystemFont, sans-serif;
  background: var(--content-bg);
  color: var(--text-primary);
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  line-height: 1.5;
}

/* ── App Shell ── */
.app {
  display: flex;
  min-height: 100vh;
}

/* ── Sidebar ── */
.sidebar {
  width: var(--sidebar-w);
  min-height: 100vh;
  background: var(--sidebar-bg);
  display: flex;
  flex-direction: column;
  position: fixed;
  top: 0;
  left: 0;
  bottom: 0;
  z-index: 50;
  transition: width var(--transition);
  overflow: hidden;
  box-shadow: 4px 0 24px rgba(0,0,0,0.2);
}

.sidebar.collapsed {
  width: var(--sidebar-w-collapsed);
}

/* Brand */
.sidebar-brand {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 20px 14px 18px;
  border-bottom: 1px solid var(--sidebar-border);
  flex-shrink: 0;
}

.brand-mark {
  width: 34px;
  height: 34px;
  background: linear-gradient(135deg, #3b82f6, #1d4ed8);
  border-radius: 9px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-family: 'Syne', sans-serif;
  font-size: 15px;
  font-weight: 800;
  color: white;
  flex-shrink: 0;
  box-shadow: 0 2px 8px rgba(37,99,235,0.4);
}

.brand-text {
  display: flex;
  flex-direction: column;
  overflow: hidden;
  white-space: nowrap;
}

.brand-name {
  font-size: 13px;
  font-weight: 700;
  color: rgba(255,255,255,0.95);
  letter-spacing: -0.2px;
  line-height: 1.2;
}

.brand-sub {
  font-size: 10px;
  color: rgba(255,255,255,0.35);
  margin-top: 1px;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

/* Nav */
.sidebar-nav {
  flex: 1;
  padding: 10px 8px;
  display: flex;
  flex-direction: column;
  gap: 2px;
  overflow-y: auto;
  overflow-x: hidden;
}

.nav-item {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 9px 10px;
  border-radius: var(--radius-sm);
  color: var(--nav-text);
  text-decoration: none;
  font-size: 13px;
  font-weight: 500;
  white-space: nowrap;
  transition: background var(--transition), color var(--transition), box-shadow var(--transition);
  position: relative;
  overflow: hidden;
}

.nav-item:hover {
  background: var(--nav-hover-bg);
  color: var(--nav-text-hover);
}

.nav-item.active {
  background: var(--nav-active-bg);
  color: var(--nav-text-active);
  font-weight: 600;
  box-shadow: inset 3px 0 0 var(--nav-active-border);
}

.nav-icon {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 18px;
  height: 18px;
  flex-shrink: 0;
}

.nav-label {
  overflow: hidden;
  text-overflow: ellipsis;
}

/* Bottom section */
.sidebar-bottom {
  flex-shrink: 0;
  padding: 8px;
  border-top: 1px solid var(--sidebar-border);
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.sidebar-utils {
  padding: 4px 2px;
}

.sidebar-profile-btn {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 9px 10px;
  border-radius: var(--radius-sm);
  background: none;
  border: none;
  cursor: pointer;
  color: var(--nav-text);
  width: 100%;
  text-align: left;
  transition: background var(--transition), color var(--transition);
  white-space: nowrap;
}

.sidebar-profile-btn:hover {
  background: var(--nav-hover-bg);
  color: var(--nav-text-hover);
}

.avatar {
  width: 28px;
  height: 28px;
  background: linear-gradient(135deg, #6366f1, #4f46e5);
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 11px;
  font-weight: 700;
  color: white;
  flex-shrink: 0;
}

.profile-label {
  font-size: 12px;
  font-weight: 500;
}

.collapse-btn {
  display: flex;
  align-items: center;
  padding: 8px 10px;
  border: none;
  background: none;
  cursor: pointer;
  color: rgba(255,255,255,0.3);
  border-radius: var(--radius-sm);
  width: 100%;
  transition: background var(--transition), color var(--transition);
  white-space: nowrap;
}

.collapse-btn:hover {
  background: var(--nav-hover-bg);
  color: rgba(255,255,255,0.7);
}

.sidebar.collapsed .collapse-btn svg {
  transform: rotate(180deg);
}

/* ── Content shell ── */
.content-shell {
  margin-left: var(--sidebar-w);
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  flex: 1;
  transition: margin-left var(--transition);
}

.content-shell.expanded {
  margin-left: var(--sidebar-w-collapsed);
}

/* ── Top utility bar ── */
.top-utility-bar {
  background: var(--surface);
  border-bottom: 1px solid var(--border);
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 28px;
  min-height: 52px;
  position: sticky;
  top: 0;
  z-index: 40;
  box-shadow: var(--shadow-sm);
}

.tasks-btn {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 7px 14px;
  background: none;
  border: 1px solid var(--border);
  border-radius: var(--radius-sm);
  font-size: 12px;
  font-weight: 600;
  color: var(--text-secondary);
  cursor: pointer;
  font-family: inherit;
  transition: all var(--transition);
  white-space: nowrap;
  flex-shrink: 0;
}

.tasks-btn:hover {
  background: var(--accent-light);
  border-color: var(--accent);
  color: var(--accent);
}

/* ── Main content ── */
.main-content {
  flex: 1;
  padding: 28px;
  max-width: 1400px;
  width: 100%;
}

/* ── Page header ── */
.page-header {
  margin-bottom: 24px;
}

.page-header h2 {
  font-family: 'Syne', sans-serif;
  font-size: 26px;
  font-weight: 800;
  color: var(--text-primary);
  letter-spacing: -0.5px;
  line-height: 1.1;
}

.page-header p {
  color: var(--text-secondary);
  font-size: 13px;
  margin-top: 5px;
}

/* ── Stats grid ── */
.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
  gap: 16px;
  margin-bottom: 24px;
}

.stat-card {
  background: var(--surface);
  padding: 20px 22px;
  border-radius: var(--radius-md);
  border: 1px solid var(--border);
  box-shadow: var(--shadow-sm);
  transition: box-shadow var(--transition), transform var(--transition);
  position: relative;
  overflow: hidden;
}

.stat-card::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 3px;
  background: var(--border);
  border-radius: var(--radius-md) var(--radius-md) 0 0;
}

.stat-card:hover {
  box-shadow: var(--shadow-md);
  transform: translateY(-1px);
}

.stat-label {
  color: var(--text-muted);
  font-size: 11px;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.6px;
  margin-bottom: 10px;
}

.stat-value {
  font-family: 'Syne', sans-serif;
  font-size: 2.2rem;
  font-weight: 800;
  color: var(--text-primary);
  letter-spacing: -1px;
  line-height: 1;
}

/* Colored stat cards */
.stat-card.success::before { background: linear-gradient(90deg, #10b981, #059669); }
.stat-card.success .stat-value { color: #059669; }

.stat-card.warning::before { background: linear-gradient(90deg, #f59e0b, #d97706); }
.stat-card.warning .stat-value { color: #d97706; }

.stat-card.danger::before { background: linear-gradient(90deg, #ef4444, #dc2626); }
.stat-card.danger .stat-value { color: #dc2626; }

.stat-card.info::before { background: linear-gradient(90deg, #3b82f6, #2563eb); }
.stat-card.info .stat-value { color: #2563eb; }

/* ── Cards ── */
.card {
  background: var(--surface);
  border-radius: var(--radius-md);
  padding: 20px 24px;
  border: 1px solid var(--border);
  margin-bottom: 20px;
  box-shadow: var(--shadow-sm);
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 16px;
  padding-bottom: 14px;
  border-bottom: 1px solid var(--border);
}

.card-title {
  font-size: 15px;
  font-weight: 700;
  color: var(--text-primary);
  letter-spacing: -0.2px;
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
  background: #f8fafc;
  border-top: 1px solid var(--border);
  border-bottom: 2px solid var(--border);
}

th {
  text-align: left;
  padding: 8px 12px;
  font-weight: 700;
  color: var(--text-secondary);
  font-size: 11px;
  text-transform: uppercase;
  letter-spacing: 0.6px;
  white-space: nowrap;
}

td {
  padding: 10px 12px;
  border-bottom: 1px solid #f1f5f9;
  color: #334155;
  font-size: 13px;
}

tbody tr {
  transition: background-color 0.1s ease;
}

tbody tr:hover {
  background: #f8fafc;
}

tbody tr:last-child td {
  border-bottom: none;
}

/* ── Badges ── */
.badge {
  display: inline-flex;
  align-items: center;
  padding: 3px 9px;
  border-radius: 20px;
  font-size: 11px;
  font-weight: 700;
  letter-spacing: 0.2px;
  white-space: nowrap;
}

.badge.success   { background: #dcfce7; color: #15803d; }
.badge.warning   { background: #fef3c7; color: #b45309; }
.badge.danger    { background: #fee2e2; color: #b91c1c; }
.badge.info      { background: #dbeafe; color: #1d4ed8; }
.badge.increasing { background: #dcfce7; color: #15803d; }
.badge.decreasing { background: #fee2e2; color: #b91c1c; }
.badge.stable    { background: #e0e7ff; color: #4338ca; }
.badge.high      { background: #fee2e2; color: #b91c1c; }
.badge.medium    { background: #fef3c7; color: #b45309; }
.badge.low       { background: #dbeafe; color: #1d4ed8; }
.badge.processing { background: #ede9fe; color: #6d28d9; }

/* ── Utility states ── */
.loading {
  text-align: center;
  padding: 48px;
  color: var(--text-muted);
  font-size: 13px;
}

.error {
  background: #fff1f2;
  border: 1px solid #fecdd3;
  color: #be123c;
  padding: 14px 16px;
  border-radius: var(--radius-md);
  margin: 12px 0;
  font-size: 13px;
  font-weight: 500;
}

/* ── Scrollbar ── */
.sidebar-nav::-webkit-scrollbar { width: 3px; }
.sidebar-nav::-webkit-scrollbar-track { background: transparent; }
.sidebar-nav::-webkit-scrollbar-thumb { background: rgba(255,255,255,0.1); border-radius: 3px; }

/* ── Responsive: Mobile overlay backdrop ── */
.sidebar-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.45);
  z-index: 49; /* just below sidebar z-index 50 */
  backdrop-filter: blur(2px);
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.2s ease;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

/* ── Hamburger button (hidden on desktop) ── */
.hamburger-btn {
  display: none;
  align-items: center;
  justify-content: center;
  width: 36px;
  height: 36px;
  border: 1px solid var(--border);
  border-radius: var(--radius-sm);
  background: none;
  cursor: pointer;
  color: var(--text-secondary);
  flex-shrink: 0;
  transition: all var(--transition);
}
.hamburger-btn:hover {
  background: var(--accent-light);
  border-color: var(--accent);
  color: var(--accent);
}

/* ── Responsive breakpoints ── */

/* Tablet: auto-collapsed (JS handles it, CSS reinforces) */
@media (max-width: 1023px) {
  .top-utility-bar {
    padding: 0 16px;
  }
  .main-content {
    padding: 20px 16px;
  }
}

/* Mobile: sidebar becomes a slide-in overlay */
@media (max-width: 767px) {
  .hamburger-btn {
    display: flex; /* show on mobile */
  }

  .sidebar {
    /* Slide completely off-screen by default on mobile */
    transform: translateX(-100%);
    /* Include transform in the transition so slide-in is smooth */
    transition: width var(--transition), transform var(--transition);
    /* Always full width when open on mobile */
    width: var(--sidebar-w) !important;
  }

  .sidebar.mobile-open {
    transform: translateX(0);
  }

  /* Content takes full width on mobile — no sidebar offset */
  .content-shell,
  .content-shell.expanded {
    margin-left: 0 !important;
  }

  .stats-grid {
    grid-template-columns: 1fr 1fr;
  }

  .main-content {
    padding: 16px;
  }
}

@media (max-width: 480px) {
  .stats-grid {
    grid-template-columns: 1fr;
  }
}
</style>
