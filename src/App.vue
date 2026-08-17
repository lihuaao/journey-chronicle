<script setup lang="ts">
import { computed, onMounted, ref } from 'vue'
import {
  ArrowDownRight, ArrowRight, ArrowUpRight, Check, Compass, Edit3,
  ExternalLink, Mail, MapPin, Menu, Save, Sparkles, Trash2, X, Zap,
} from 'lucide-vue-next'

type Profile = {
  name: string
  initials: string
  role: string
  tagline: string
  bio: string
  location: string
  availability: string
  avatar: string
  email: string
  website: string
  interests: string
}

type Entry = {
  id: number
  year: string
  date: string
  category: string
  title: string
  place: string
  summary: string
  body: string
  tags: string
  image: string
  metric: string
}

const seedProfile: Profile = {
  name: 'LI HAO',
  initials: 'LH',
  role: '风光记录者 · 户外爱好者',
  tagline: '在山野、风里和光影之间，保持好奇。',
  bio: '我喜欢把自己放进真实的环境里：走一段没有路牌的山路，沿着江岸骑到天黑，或者在一场雪里等一束刚好的光。这里是我的个人索引，记录经历，也分享正在发生的事。',
  location: '上海 · 中国',
  availability: '开放合作',
  avatar: 'https://images.unsplash.com/photo-1500534623283-312aade485b7?auto=format&fit=crop&w=1200&q=85',
  email: 'hello@example.com',
  website: 'fieldnote.example',
  interests: '徒步, 骑行, 摄影, 钓鱼, 滑雪',
}

const seedEntries: Entry[] = [
  { id: 1, year: '2025', date: '2025.02 — 至今', category: 'CURRENTLY', title: '把生活搬到户外', place: '中国 · 西南线', summary: '一边工作，一边把更多时间还给山、河流和清晨。', body: '正在整理一份关于西南山地的长期影像计划。它不追求打卡，而是记录人在自然里如何改变速度、距离和判断。', tags: '长期项目, 影像计划', image: 'https://images.unsplash.com/photo-1464278533981-50106e6176b1?auto=format&fit=crop&w=1200&q=85', metric: '01 / ongoing' },
  { id: 2, year: '2024', date: '2024.09 — 2024.11', category: 'PROJECT', title: '沿江骑行 312 公里', place: '浙江 · 钱塘江', summary: '用三天时间，从潮汐、桥和路边小店重新认识一条江。', body: '路线从上游开始，尽量避开快速路。每天只设一个目的地，剩下的时间留给天气、偶遇和拍摄。', tags: '骑行, 纪实, 312 km', image: 'https://images.unsplash.com/photo-1502744688674-c619d1586c9e?auto=format&fit=crop&w=1200&q=85', metric: '312 km / 03 days' },
  { id: 3, year: '2023', date: '2023.12 — 2024.01', category: 'SERIES', title: '雪线之上的 17 个清晨', place: '北海道 · 日本', summary: '在雪落下来之前，记录山脊上的蓝色时刻。', body: '这一组照片拍摄于清晨五点到七点之间。低温、风向和能见度共同决定了每一张照片的构图。', tags: '摄影, 雪山, 17 frames', image: 'https://images.unsplash.com/photo-1486911278844-a81c5267e227?auto=format&fit=crop&w=1200&q=85', metric: '17 frames / film' },
  { id: 4, year: '2022', date: '2022.05 — 2022.10', category: 'LEARNING', title: '开始认真观察世界', place: '上海 · 日常半径', summary: '从一台相机和一张空白地图开始，建立自己的观察习惯。', body: '摄影不是目的，注意力才是。那一年我开始记录每天经过的街道、光线和陌生人的动作。', tags: '摄影, 观察, 练习', image: 'https://images.unsplash.com/photo-1516035069371-29a1b244cc32?auto=format&fit=crop&w=1200&q=85', metric: '100+ walks' },
]

const profile = ref<Profile>({ ...seedProfile })
const entries = ref<Entry[]>([])
const activeView = ref<'public' | 'admin'>('public')
const activeTab = ref<'profile' | 'timeline' | 'appearance'>('profile')
const selectedEntry = ref<Entry | null>(null)
const editingId = ref<number | null>(null)
const mobileOpen = ref(false)
const profileForm = ref<Profile>({ ...seedProfile })
const entryForm = ref<Entry>({ ...seedEntries[0], id: 0, year: '', date: '', title: '', place: '', summary: '', body: '', tags: '', image: '', metric: '', category: 'PROJECT' })

onMounted(() => {
  profile.value = JSON.parse(localStorage.getItem('fieldnote-profile') || JSON.stringify(seedProfile))
  entries.value = JSON.parse(localStorage.getItem('fieldnote-entries') || JSON.stringify(seedEntries))
  profileForm.value = { ...profile.value }
})

const interestList = computed(() => profile.value.interests.split(',').map(item => item.trim()).filter(Boolean))
const sortedEntries = computed(() => [...entries.value].sort((a, b) => b.year.localeCompare(a.year)))
const featuredEntries = computed(() => sortedEntries.value.slice(0, 3))

function saveProfile() {
  profile.value = { ...profileForm.value }
  localStorage.setItem('fieldnote-profile', JSON.stringify(profile.value))
}
function resetEntry() {
  editingId.value = null
  entryForm.value = { ...seedEntries[0], id: 0, year: '', date: '', title: '', place: '', summary: '', body: '', tags: '', image: '', metric: '', category: 'PROJECT' }
}
function editEntry(item: Entry) { editingId.value = item.id; entryForm.value = { ...item }; activeTab.value = 'timeline'; window.scrollTo({ top: 0, behavior: 'smooth' }) }
function saveEntry() {
  if (!entryForm.value.title || !entryForm.value.year || !entryForm.value.summary) return
  const next = { ...entryForm.value, id: editingId.value || Date.now(), image: entryForm.value.image || seedEntries[0].image }
  entries.value = editingId.value ? entries.value.map(item => item.id === editingId.value ? next : item) : [next, ...entries.value]
  localStorage.setItem('fieldnote-entries', JSON.stringify(entries.value)); resetEntry()
}
function deleteEntry(id: number) {
  entries.value = entries.value.filter(item => item.id !== id)
  localStorage.setItem('fieldnote-entries', JSON.stringify(entries.value))
}
function openPublic() { activeView.value = 'public'; mobileOpen.value = false }
function scrollTo(id: string) { activeView.value = 'public'; mobileOpen.value = false; requestAnimationFrame(() => document.getElementById(id)?.scrollIntoView({ behavior: 'smooth' })) }
</script>

<template>
  <div class="app-shell">
    <header class="site-header">
      <button class="wordmark" @click="openPublic"><span class="wordmark-dot"></span><span>FIELDNOTE</span><small>/ PERSONAL INDEX</small></button>
      <nav class="desktop-nav"><button @click="scrollTo('about')">关于我</button><button @click="scrollTo('timeline')">履历时间线</button><button @click="scrollTo('work')">作品切片</button><button class="edit-link" @click="activeView = 'admin'"><Edit3 :size="14" /> 编辑主页</button></nav>
      <button class="icon-button mobile-menu" title="打开菜单" @click="mobileOpen = !mobileOpen"><Menu :size="18" /></button>
      <div v-if="mobileOpen" class="mobile-nav"><button @click="scrollTo('about')">关于我</button><button @click="scrollTo('timeline')">履历时间线</button><button @click="scrollTo('work')">作品切片</button><button @click="activeView = 'admin'; mobileOpen = false">编辑主页</button></div>
    </header>

    <main v-if="activeView === 'public'" class="public-page">
      <section class="hero-section page-width">
        <div class="hero-copy reveal"><p class="overline"><span class="live-dot"></span> {{ profile.availability }} · {{ profile.location }}</p><h1>{{ profile.name }}<span class="stroke-mark">.</span></h1><p class="hero-role">{{ profile.role }}</p><p class="hero-tagline">{{ profile.tagline }}</p><div class="hero-actions"><button class="primary-btn" @click="scrollTo('timeline')">从时间线认识我 <ArrowDownRight :size="17" /></button><a class="text-link" :href="`mailto:${profile.email}`">联系我 <ArrowRight :size="15" /></a></div></div>
        <div class="hero-visual reveal delay-1"><div class="hero-image-wrap"><img :src="profile.avatar" :alt="profile.name" /><div class="image-caption"><span>01 / FIELD NOTES</span><span>LOOKING CLOSER</span></div></div><div class="hero-stamp"><Compass :size="22" /><span>STAY<br />CURIOUS</span></div></div>
      </section>

      <div class="ticker"><div class="ticker-track"><span v-for="(interest, index) in [...interestList, ...interestList]" :key="`${interest}-${index}`">{{ interest }} <i>✦</i></span></div></div>

      <section id="about" class="about-section page-width section-grid"><div class="section-index">01 <span>ABOUT</span></div><div class="about-content"><h2>一个正在持续更新的人。</h2><p class="about-lede">{{ profile.bio }}</p><div class="about-facts"><div><small>BASE</small><strong>{{ profile.location }}</strong></div><div><small>FOCUS</small><strong>{{ interestList.slice(0, 3).join(' · ') }}</strong></div><div><small>OPEN TO</small><strong>{{ profile.availability }}</strong></div></div><div class="interest-list"><span v-for="(interest, index) in interestList" :key="interest" class="interest-chip" :class="`tone-${index % 4}`">{{ interest }}</span></div></div></section>

      <section id="timeline" class="timeline-section page-width"><div class="section-grid section-heading"><div class="section-index">02 <span>THE TRACE</span></div><div><h2>经历不是履历表，<br /><em>是你走过的路径。</em></h2><p>每一个节点都可以展开，看到当时的背景、选择和留下的作品。</p></div></div><div class="trace-list"><article v-for="item in sortedEntries" :key="item.id" class="trace-item" @click="selectedEntry = item"><div class="trace-year">{{ item.year }}</div><div class="trace-marker"><span></span></div><div class="trace-main"><div class="trace-meta"><span>{{ item.category }}</span><time>{{ item.date }}</time></div><h3>{{ item.title }}</h3><p class="trace-place"><MapPin :size="13" /> {{ item.place }}</p><p class="trace-summary">{{ item.summary }}</p><div class="trace-bottom"><div class="mini-tags"><span v-for="tag in item.tags.split(',').slice(0, 3)" :key="tag">{{ tag.trim() }}</span></div><button class="round-arrow" title="查看详情"><ArrowUpRight :size="17" /></button></div></div><img class="trace-image" :src="item.image" :alt="item.title" /></article></div></section>

      <section id="work" class="work-section page-width"><div class="section-grid section-heading"><div class="section-index">03 <span>SELECTED WORK</span></div><div><h2>留下一些可以<br /><em>被看见的东西。</em></h2><p>路上的照片、项目和一些仍然在生长的想法。</p></div></div><div class="work-grid"><article v-for="(item, index) in featuredEntries" :key="item.id" class="work-card" :class="{ featured: index === 0 }" @click="selectedEntry = item"><img :src="item.image" :alt="item.title" /><div class="work-overlay"><span>{{ item.category }}</span><h3>{{ item.title }}</h3><p>{{ item.metric }}</p></div><div class="work-corner"><ExternalLink :size="15" /></div></article></div></section>

      <section class="contact-section page-width"><div><p class="overline">LET'S MAKE SOMETHING REAL</p><h2>有一个故事，<br />正在等着被记录。</h2></div><a class="contact-button" :href="`mailto:${profile.email}`"><Mail :size="18" /> {{ profile.email }} <ArrowUpRight :size="18" /></a></section>
    </main>

    <main v-else class="admin-page page-width"><div class="admin-top"><div><p class="overline">FIELDNOTE / EDITOR</p><h1>编辑你的个人索引</h1><p>这里的每一处修改都会同步到公开主页。</p></div><button class="text-link" @click="openPublic">返回公开主页 <ExternalLink :size="15" /></button></div><div class="admin-tabs"><button :class="{ active: activeTab === 'profile' }" @click="activeTab = 'profile'">个人资料</button><button :class="{ active: activeTab === 'timeline' }" @click="activeTab = 'timeline'">履历节点 <span>{{ entries.length }}</span></button><button :class="{ active: activeTab === 'appearance' }" @click="activeTab = 'appearance'">外观设置</button></div>
      <section v-if="activeTab === 'profile'" class="editor-panel"><div class="panel-title"><div><span class="panel-kicker">PROFILE / 01</span><h2>先让别人认识你</h2></div><button class="primary-btn" @click="saveProfile"><Save :size="16" /> 保存资料</button></div><div class="editor-grid"><label>你的名字<input v-model="profileForm.name" /></label><label>头像字母<input v-model="profileForm.initials" maxlength="3" /></label><label>身份 / 角色<input v-model="profileForm.role" /></label><label>所在位置<input v-model="profileForm.location" /></label><label class="wide">一句话介绍<textarea v-model="profileForm.tagline" rows="2" /></label><label class="wide">个人简介<textarea v-model="profileForm.bio" rows="5" /></label><label>对外状态<input v-model="profileForm.availability" /></label><label>联系邮箱<input v-model="profileForm.email" type="email" /></label><label class="wide">头像图片地址<input v-model="profileForm.avatar" /></label><label class="wide">兴趣标签（用逗号分隔）<input v-model="profileForm.interests" placeholder="徒步, 摄影, 滑雪" /></label></div><div class="editor-preview"><span class="preview-label">LIVE PREVIEW</span><strong>{{ profileForm.name || '你的名字' }}</strong><span>{{ profileForm.role || '你的身份 / 角色' }}</span></div></section>
      <section v-else-if="activeTab === 'timeline'" class="editor-panel"><div class="panel-title"><div><span class="panel-kicker">TRACE / 02</span><h2>{{ editingId ? '编辑这个节点' : '添加一个新节点' }}</h2></div><button v-if="editingId" class="icon-button" title="取消编辑" @click="resetEntry"><X :size="17" /></button></div><div class="editor-grid"><label>年份<input v-model="entryForm.year" placeholder="2025" /></label><label>节点日期<input v-model="entryForm.date" placeholder="2025.02 — 至今" /></label><label>节点类型<select v-model="entryForm.category"><option>PROJECT</option><option>CURRENTLY</option><option>SERIES</option><option>LEARNING</option><option>LIFE</option></select></label><label>地点<input v-model="entryForm.place" placeholder="中国 · 西南线" /></label><label class="wide">标题<input v-model="entryForm.title" placeholder="这个节点发生了什么？" /></label><label class="wide">一句话摘要<textarea v-model="entryForm.summary" rows="2" /></label><label class="wide">详细介绍<textarea v-model="entryForm.body" rows="5" /></label><label class="wide">标签（用逗号分隔）<input v-model="entryForm.tags" placeholder="摄影, 纪实, 长期项目" /></label><label class="wide">图片地址<input v-model="entryForm.image" placeholder="https://..." /></label><label>数据标记<input v-model="entryForm.metric" placeholder="312 km / 03 days" /></label></div><div class="editor-actions"><button class="ghost-btn" @click="resetEntry">清空</button><button class="primary-btn" :disabled="!entryForm.title || !entryForm.year || !entryForm.summary" @click="saveEntry"><Check :size="16" /> {{ editingId ? '保存修改' : '添加节点' }}</button></div><div class="entry-manager"><div class="manager-head"><span>已有节点</span><small>点击编辑或删除</small></div><div v-for="item in sortedEntries" :key="item.id" class="manager-row"><div><strong>{{ item.year }} / {{ item.title }}</strong><span>{{ item.category }} · {{ item.place }}</span></div><div><button class="icon-button" title="编辑节点" @click="editEntry(item)"><Edit3 :size="15" /></button><button class="icon-button danger" title="删除节点" @click="deleteEntry(item.id)"><Trash2 :size="15" /></button></div></div></div></section>
      <section v-else class="editor-panel"><div class="panel-title"><div><span class="panel-kicker">MOOD / 03</span><h2>选择你的现场</h2></div></div><div class="appearance-options"><button class="appearance-choice selected"><span class="swatch ink"></span><strong>Ink & Paper</strong><small>当前主题 · 深墨黑 / 米白 / 荧光绿</small></button><button class="appearance-choice"><span class="swatch coral"></span><strong>Coral Signal</strong><small>即将支持自定义色彩</small></button><button class="appearance-choice"><span class="swatch cobalt"></span><strong>Cobalt Night</strong><small>即将支持深色模式</small></button></div><div class="motion-note"><Zap :size="17" /><span>页面已启用轻量动效：滚动出现、图片悬停和时间线交互。动效不会影响阅读。</span></div></section>
    </main>

    <footer class="site-footer page-width"><span>© {{ new Date().getFullYear() }} {{ profile.name }} / FIELDNOTE</span><span>MADE TO BE SHARED <Sparkles :size="14" /></span></footer>

    <div v-if="selectedEntry" class="detail-backdrop" @click.self="selectedEntry = null"><article class="detail-modal"><button class="modal-close icon-button" title="关闭详情" @click="selectedEntry = null"><X :size="17" /></button><img :src="selectedEntry.image" :alt="selectedEntry.title" /><div class="modal-content"><div class="trace-meta"><span>{{ selectedEntry.category }}</span><time>{{ selectedEntry.date }}</time></div><h2>{{ selectedEntry.title }}</h2><p class="trace-place"><MapPin :size="13" /> {{ selectedEntry.place }}</p><p class="modal-body">{{ selectedEntry.body }}</p><div class="mini-tags"><span v-for="tag in selectedEntry.tags.split(',')" :key="tag">{{ tag.trim() }}</span></div><div class="modal-metric">{{ selectedEntry.metric }}</div></div></article></div>
  </div>
</template>
