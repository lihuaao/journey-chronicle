<script setup lang="ts">
import { computed, onMounted, ref } from 'vue'
import {
  ArrowRight, BookOpen, CalendarDays, Check, ChevronRight, Compass,
  FileText, Image as ImageIcon, LayoutDashboard, MapPin, Pencil, Plus,
  Sparkles, Trash2, Upload, Video, Wallet, X,
} from 'lucide-vue-next'

type Memory = {
  id: number
  date: string
  location: string
  title: string
  category: string
  summary: string
  content: string
  expense: number
  cover: string
}

const seedMemories: Memory[] = [
  { id: 1, date: '2024-10-03', location: '京都 · 日本', title: '在秋色里慢下来', category: '灵感', summary: '沿着鸭川散步，找到一家只卖三种甜点的小店。', content: '清晨从民宿出发，沿着鸭川一路向北。风里有桂花和一点木头的味道，下午在哲学之道看完最后一片红叶。', expense: 860, cover: 'https://images.unsplash.com/photo-1493976040374-85c8e12f0c0e?auto=format&fit=crop&w=900&q=85' },
  { id: 2, date: '2024-10-05', location: '奈良 · 日本', title: '把时间交给一座公园', category: '漫游', summary: '没有打卡清单的一天，只有鹿、树影和一碗热汤。', content: '把原本的购物计划取消，给自己留了一整天。公园里的小路很安静，落日的时候坐在池边写下这段旅程。', expense: 420, cover: 'https://images.unsplash.com/photo-1528360983277-13d401cdc186?auto=format&fit=crop&w=900&q=85' },
  { id: 3, date: '2024-10-07', location: '大阪 · 日本', title: '夜色与最后一班车', category: '收尾', summary: '用一顿热气腾腾的章鱼烧，为旅程画上句号。', content: '最后一晚去了通天阁附近，街灯刚刚亮起。记录下这次旅途的花费和最想再来的三个地方。', expense: 680, cover: 'https://images.unsplash.com/photo-1590559899731-a382839e5549?auto=format&fit=crop&w=900&q=85' },
]

const memories = ref<Memory[]>([])
const mode = ref<'public' | 'admin'>('public')
const detail = ref<Memory | null>(null)
const editingId = ref<number | null>(null)
const form = ref({ date: '', location: '', title: '', category: '灵感', summary: '', content: '', expense: 0, cover: '' })
const uploadedNames = ref<string[]>([])
const activeAdminTab = ref('entries')

onMounted(() => {
  const saved = localStorage.getItem('journey-chronicle-memories')
  memories.value = saved ? JSON.parse(saved) : seedMemories
})

const totalExpense = computed(() => memories.value.reduce((sum, item) => sum + Number(item.expense || 0), 0))
const sortedMemories = computed(() => [...memories.value].sort((a, b) => a.date.localeCompare(b.date)))

function persist() { localStorage.setItem('journey-chronicle-memories', JSON.stringify(memories.value)) }
function resetForm() { form.value = { date: '', location: '', title: '', category: '灵感', summary: '', content: '', expense: 0, cover: '' }; uploadedNames.value = []; editingId.value = null }
function editMemory(item: Memory) { editingId.value = item.id; form.value = { ...item }; activeAdminTab.value = 'entries'; window.scrollTo({ top: 0, behavior: 'smooth' }) }
function deleteMemory(id: number) { memories.value = memories.value.filter(item => item.id !== id); persist() }
function saveMemory() {
  if (!form.value.title || !form.value.date || !form.value.location) return
  const item: Memory = { ...form.value, id: editingId.value ?? Date.now(), expense: Number(form.value.expense || 0), cover: form.value.cover || seedMemories[0].cover }
  if (editingId.value) memories.value = memories.value.map(existing => existing.id === editingId.value ? item : existing)
  else memories.value = [item, ...memories.value]
  persist(); resetForm()
}
function handleFiles(event: Event) {
  const files = Array.from((event.target as HTMLInputElement).files ?? [])
  uploadedNames.value = files.map(file => file.name)
  const firstImage = files.find(file => file.type.startsWith('image/'))
  if (firstImage) form.value.cover = URL.createObjectURL(firstImage)
}
function scrollToTimeline() { document.getElementById('timeline')?.scrollIntoView({ behavior: 'smooth' }) }
</script>

<template>
  <div class="shell">
    <header class="topbar">
      <div class="brand"><span class="brand-mark"><Compass :size="18" /></span><span>旅途编年册</span></div>
      <nav class="topnav" aria-label="主导航">
        <button class="nav-btn" :class="{ active: mode === 'public' }" @click="mode = 'public'"><BookOpen :size="16" /><span class="nav-label">旅程展示</span></button>
        <button class="nav-btn admin-pill" :class="{ active: mode === 'admin' }" @click="mode = 'admin'"><LayoutDashboard :size="16" /><span class="nav-label">内容后台</span></button>
      </nav>
    </header>

    <main v-if="mode === 'public'" class="main">
      <section class="hero">
        <div>
          <p class="eyebrow">A living travel journal</p>
          <h1>把走过的地方，<br />变成一段可以回看的故事。</h1>
          <p class="lede">用时间节点收集目的地、行程、攻略和花费。让每一次出发都被好好记录，也让下一次旅途更有方向。</p>
          <div class="hero-actions"><button class="btn" @click="scrollToTimeline">查看旅程 <ArrowRight :size="16" /></button><button class="btn secondary" @click="mode = 'admin'"><LayoutDashboard :size="15" /> 管理内容</button></div>
        </div>
        <div class="hero-art"><div class="hero-note"><strong>关西，7 日漫游</strong><span>京都 · 奈良 · 大阪 &nbsp; / &nbsp; 2024.10</span></div></div>
      </section>

      <section class="stats" aria-label="旅程概览">
        <div class="stat"><MapPin class="stat-icon" :size="20" /><span class="stat-label">记录地点</span><strong class="stat-value">03</strong></div>
        <div class="stat"><CalendarDays class="stat-icon" :size="20" /><span class="stat-label">时间节点</span><strong class="stat-value">{{ String(memories.length).padStart(2, '0') }}</strong></div>
        <div class="stat"><Wallet class="stat-icon" :size="20" /><span class="stat-label">旅程花费</span><strong class="stat-value">¥{{ totalExpense.toLocaleString() }}</strong></div>
        <div class="stat"><ImageIcon class="stat-icon" :size="20" /><span class="stat-label">图像记录</span><strong class="stat-value">12</strong></div>
      </section>

      <section class="progress-wrap"><div class="section-head"><div><p class="eyebrow">Trip progress</p><h2>这段旅程走到哪里了？</h2></div><strong>64%</strong></div><div class="progress-line"><span></span></div><div class="progress-meta"><span>2024.10.03 · 京都</span><span>2024.10.09 · 回到日常</span></div></section>

      <section id="timeline"><div class="section-head"><div><p class="eyebrow">The timeline</p><h2>沿着时间，重新出发</h2></div><p>{{ memories.length }} 个被保存的瞬间</p></div>
        <div class="timeline">
          <article v-for="item in sortedMemories" :key="item.id" class="timeline-item"><span class="timeline-dot"></span><time class="timeline-date">{{ item.date.replaceAll('-', '.') }}</time><div class="memory"><img class="memory-cover" :src="item.cover" :alt="item.title" /><div class="memory-body"><span class="tag"><Sparkles :size="12" /> {{ item.category }}</span><h3>{{ item.title }}</h3><p>{{ item.summary }}</p><button class="text-btn" @click="detail = item">阅读这一站 <ChevronRight :size="14" /></button></div></div></article>
        </div>
      </section>
    </main>

    <main v-else class="main">
      <div class="admin-layout">
        <aside class="admin-sidebar"><h2>内容后台</h2><button class="side-link" :class="{ active: activeAdminTab === 'entries' }" @click="activeAdminTab = 'entries'"><FileText :size="16" /> 时间节点</button><button class="side-link" :class="{ active: activeAdminTab === 'media' }" @click="activeAdminTab = 'media'"><ImageIcon :size="16" /> 媒体资料</button><button class="side-link" :class="{ active: activeAdminTab === 'settings' }" @click="activeAdminTab = 'settings'"><Compass :size="16" /> 旅程设置</button></aside>
        <section class="admin-content">
          <div class="admin-heading"><div><h1>记录你的旅程</h1><p>所有内容保存于当前浏览器，可随时编辑和整理。</p></div><button class="btn accent" @click="resetForm"><Plus :size="16" /> 新增节点</button></div>
          <template v-if="activeAdminTab === 'entries'">
            <div class="admin-card"><div class="admin-card-head"><h3>{{ editingId ? '编辑时间节点' : '新增时间节点' }}</h3><button v-if="editingId" class="icon-btn" title="取消编辑" @click="resetForm"><X :size="16" /></button></div><div class="form-grid"><div class="field"><label>发生日期</label><input v-model="form.date" type="date" /></div><div class="field"><label>目的地</label><input v-model="form.location" placeholder="例如：京都 · 日本" /></div><div class="field"><label>节点标题</label><input v-model="form.title" placeholder="给这一站一个名字" /></div><div class="field"><label>内容分类</label><select v-model="form.category"><option>灵感</option><option>漫游</option><option>攻略</option><option>收尾</option></select></div><div class="field full"><label>一句话摘要</label><input v-model="form.summary" placeholder="在列表中展示的简短描述" /></div><div class="field full"><label>详细记录</label><textarea v-model="form.content" placeholder="写下当天的行程、攻略或心情..."></textarea></div><div class="field"><label>本节点花费（元）</label><input v-model="form.expense" type="number" min="0" /></div><div class="field"><label>封面图片 URL（可选）</label><input v-model="form.cover" placeholder="https://..." /></div><div class="field full"><label>上传图片或视频</label><div class="upload-box"><Upload :size="18" /><span>{{ uploadedNames.length ? uploadedNames.join('、') : '选择本地媒体文件，生成预览' }}</span><input type="file" accept="image/*,video/*" multiple @change="handleFiles" /></div></div></div><div class="form-actions"><button class="btn secondary" @click="resetForm">清空</button><button class="btn" :disabled="!form.title || !form.date || !form.location" @click="saveMemory"><Check :size="16" /> 保存节点</button></div></div>
            <div class="admin-card"><div class="admin-card-head"><h3>已记录的节点</h3><span class="tag">{{ memories.length }} 条记录</span></div><div v-if="memories.length"><div v-for="item in memories" :key="item.id" class="record-row"><img class="record-thumb" :src="item.cover" :alt="item.title" /><div class="record-meta"><strong>{{ item.title }}</strong><span>{{ item.date.replaceAll('-', '.') }} · {{ item.location }} · ¥{{ item.expense }}</span></div><div class="row-actions"><button class="icon-btn" title="编辑" @click="editMemory(item)"><Pencil :size="15" /></button><button class="icon-btn danger" title="删除" @click="deleteMemory(item.id)"><Trash2 :size="15" /></button></div></div></div><div v-else class="empty">还没有节点，先记录第一站吧。</div></div>
          </template>
          <template v-else-if="activeAdminTab === 'media'"><div class="admin-card"><div class="admin-card-head"><h3>媒体资料</h3></div><div class="empty"><Video :size="28" /><p>媒体会跟随时间节点保存。进入“时间节点”后即可上传图片或视频。</p></div></div></template>
          <template v-else><div class="admin-card"><div class="admin-card-head"><h3>旅程设置</h3></div><div class="form-grid"><div class="field"><label>旅程名称</label><input value="关西，7 日漫游" /></div><div class="field"><label>出发日期</label><input value="2024-10-03" type="date" /></div><div class="field full"><label>公开简介</label><textarea value="把走过的地方，变成一段可以回看的故事。" /></div></div><div class="form-actions"><button class="btn"><Check :size="16" /> 保存设置</button></div></div></template>
        </section>
      </div>
    </main>

    <div v-if="detail" class="drawer-backdrop" @click.self="detail = null"><article class="drawer"><div class="drawer-top"><span class="detail-kicker">Journey note</span><button class="icon-btn" title="关闭详情" @click="detail = null"><X :size="17" /></button></div><img class="detail-cover" :src="detail.cover" :alt="detail.title" /><p class="detail-kicker">{{ detail.date.replaceAll('-', '.') }} · {{ detail.location }}</p><h2>{{ detail.title }}</h2><p class="detail-copy">{{ detail.content }}</p><div class="detail-facts"><div class="fact"><small>本节点花费</small><strong>¥{{ detail.expense.toLocaleString() }}</strong></div><div class="fact"><small>内容分类</small><strong>{{ detail.category }}</strong></div></div><button class="btn secondary" @click="detail = null">返回时间轴 <ArrowRight :size="15" /></button></article></div>
  </div>
</template>
