<script setup lang="ts">
import { computed, nextTick, onBeforeUnmount, onMounted, ref, watch } from 'vue'
import {
  ArrowDownRight, ArrowRight, ArrowUpRight, BarChart3, Check, Compass, Edit3,
  ExternalLink, Eye, Image, LayoutDashboard, LogIn, LogOut, Mail, MapPin,
  Menu, Save, Settings, ShieldCheck, Sparkles, Trash2, Upload, Video, X,
} from 'lucide-vue-next'

type ThemeKey = 'signal' | 'coral' | 'cobalt'
type EntryCategory = 'PROJECT' | 'WORK' | 'LEARNING' | 'LIFE' | 'NOW'
type AdminSection = 'dashboard' | 'profile' | 'entries' | 'media' | 'settings'

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
  theme: ThemeKey
}

type Entry = {
  id: number
  year: string
  date: string
  category: EntryCategory
  title: string
  place: string
  summary: string
  body: string
  tags: string
  image: string
  images: string[]
  metric: string
}

type MediaAsset = {
  id: number
  name: string
  type: 'IMAGE' | 'VIDEO'
  size: string
  date: string
  url: string
  entryId?: number
}

type VisualSettings = {
  rotationSeconds: number
  backgroundOpacity: number
  coverImage: string
  selectedBackgroundImages: string[]
}

const categoryOptions: EntryCategory[] = ['PROJECT', 'WORK', 'LEARNING', 'LIFE', 'NOW']
const publicCategories = ['ALL', ...categoryOptions] as const
const themeOptions: Array<{ id: ThemeKey; label: string; note: string }> = [
  { id: 'signal', label: 'Alpine Light', note: '通透自然 · 薄荷高亮 · 适合旅行档案' },
  { id: 'coral', label: 'Coral Current', note: '暖灰底 · 珊瑚重点 · 更有温度' },
  { id: 'cobalt', label: 'Cobalt Field', note: '冷白底 · 钴蓝重点 · 更偏作品集' },
]

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
  theme: 'coral',
}

const seedVisualSettings: VisualSettings = {
  rotationSeconds: 7,
  backgroundOpacity: .1,
  coverImage: '',
  selectedBackgroundImages: [],
}

const seedEntries: Entry[] = [
  { id: 1, year: '2025', date: '2025.02 — 至今', category: 'NOW', title: '把生活搬到户外', place: '中国 · 西南线', summary: '一边工作，一边把更多时间还给山、河流和清晨。', body: '正在整理一份关于西南山地的长期影像计划。它不追求打卡，而是记录人在自然里如何改变速度、距离和判断。', tags: '长期项目, 影像计划', image: 'https://images.unsplash.com/photo-1464278533981-50106e6176b1?auto=format&fit=crop&w=1200&q=85', images: ['https://images.unsplash.com/photo-1464278533981-50106e6176b1?auto=format&fit=crop&w=1600&q=85', 'https://images.unsplash.com/photo-1500534623283-312aade485b7?auto=format&fit=crop&w=1600&q=85', 'https://images.unsplash.com/photo-1441974231531-c6227db76b6e?auto=format&fit=crop&w=1600&q=85', 'https://images.unsplash.com/photo-1500530855697-b586d89ba3ee?auto=format&fit=crop&w=1600&q=85'], metric: '01 / ongoing' },
  { id: 2, year: '2024', date: '2024.09 — 2024.11', category: 'PROJECT', title: '沿江骑行 312 公里', place: '浙江 · 钱塘江', summary: '用三天时间，从潮汐、桥和路边小店重新认识一条江。', body: '路线从上游开始，尽量避开快速路。每天只设一个目的地，剩下的时间留给天气、偶遇和拍摄。', tags: '骑行, 纪实, 312 km', image: 'https://images.unsplash.com/photo-1502744688674-c619d1586c9e?auto=format&fit=crop&w=1200&q=85', images: ['https://images.unsplash.com/photo-1502744688674-c619d1586c9e?auto=format&fit=crop&w=1600&q=85', 'https://images.unsplash.com/photo-1485965120184-e220f721d03e?auto=format&fit=crop&w=1600&q=85', 'https://images.unsplash.com/photo-1500534623283-312aade485b7?auto=format&fit=crop&w=1600&q=85'], metric: '312 km / 03 days' },
  { id: 3, year: '2023', date: '2023.12 — 2024.01', category: 'WORK', title: '雪线之上的 17 个清晨', place: '北海道 · 日本', summary: '在雪落下来之前，记录山脊上的蓝色时刻。', body: '这一组照片拍摄于清晨五点到七点之间。低温、风向和能见度共同决定了每一张照片的构图。', tags: '摄影, 雪山, 17 frames', image: 'https://images.unsplash.com/photo-1486911278844-a81c5267e227?auto=format&fit=crop&w=1200&q=85', images: ['https://images.unsplash.com/photo-1486911278844-a81c5267e227?auto=format&fit=crop&w=1600&q=85', 'https://images.unsplash.com/photo-1483347756197-71ef80e95f73?auto=format&fit=crop&w=1600&q=85', 'https://images.unsplash.com/photo-1497250681960-ef046c08a56e?auto=format&fit=crop&w=1600&q=85', 'https://images.unsplash.com/photo-1511497584788-876760111969?auto=format&fit=crop&w=1600&q=85'], metric: '17 frames / film' },
  { id: 4, year: '2022', date: '2022.05 — 2022.10', category: 'LEARNING', title: '开始认真观察世界', place: '上海 · 日常半径', summary: '从一台相机和一张空白地图开始，建立自己的观察习惯。', body: '摄影不是目的，注意力才是。那一年我开始记录每天经过的街道、光线和陌生人的动作。', tags: '摄影, 观察, 练习', image: 'https://images.unsplash.com/photo-1516035069371-29a1b244cc32?auto=format&fit=crop&w=1200&q=85', images: ['https://images.unsplash.com/photo-1516035069371-29a1b244cc32?auto=format&fit=crop&w=1600&q=85', 'https://images.unsplash.com/photo-1452780212940-6f5c0d14d848?auto=format&fit=crop&w=1600&q=85', 'https://images.unsplash.com/photo-1492562080023-ab3db95bfbce?auto=format&fit=crop&w=1600&q=85'], metric: '100+ walks' },
  { id: 5, year: '2021', date: '2021.04 — 2021.09', category: 'LIFE', title: '在城市里找到水面', place: '上海 · 苏州河', summary: '把日常散步变成一份关于城市边缘的生活档案。', body: '没有明确的项目目标，只是每周沿着河走一段，拍下桥下的光、钓鱼的人和慢慢变化的岸线。', tags: '生活, 观察, 城市', image: 'https://images.unsplash.com/photo-1493246507139-91e8fad9978e?auto=format&fit=crop&w=1200&q=85', images: ['https://images.unsplash.com/photo-1493246507139-91e8fad9978e?auto=format&fit=crop&w=1600&q=85', 'https://images.unsplash.com/photo-1470770841072-f978cf4d019e?auto=format&fit=crop&w=1600&q=85', 'https://images.unsplash.com/photo-1500534314209-a25ddb2bd429?auto=format&fit=crop&w=1600&q=85'], metric: '42 walks' },
]

const seedMedia: MediaAsset[] = [
  { id: 1, name: 'southwest-dawn.jpg', type: 'IMAGE', size: '2.4 MB', date: '2025.02.18', url: 'https://images.unsplash.com/photo-1464278533981-50106e6176b1?auto=format&fit=crop&w=1200&q=85', entryId: 1 },
  { id: 2, name: 'qiantang-ride.jpg', type: 'IMAGE', size: '1.8 MB', date: '2024.11.03', url: 'https://images.unsplash.com/photo-1502744688674-c619d1586c9e?auto=format&fit=crop&w=1200&q=85', entryId: 2 },
  { id: 3, name: 'wildflower-motion.mp4', type: 'VIDEO', size: '1.1 MB', date: '2024.01.08', url: 'https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.mp4', entryId: 3 },
  { id: 4, name: 'city-waterline.jpg', type: 'IMAGE', size: '3.1 MB', date: '2021.09.20', url: 'https://images.unsplash.com/photo-1493246507139-91e8fad9978e?auto=format&fit=crop&w=1200&q=85', entryId: 5 },
  { id: 5, name: 'trail-sequence.mp4', type: 'VIDEO', size: '5.2 MB', date: '2024.10.19', url: 'https://media.w3.org/2010/05/sintel/trailer.mp4', entryId: 2 },
]

const profile = ref<Profile>({ ...seedProfile })
const entries = ref<Entry[]>([])
const activeView = ref<'public' | 'admin'>('public')
const adminAuthenticated = ref(false)
const adminSection = ref<AdminSection>('dashboard')
const adminNotice = ref('')
const mediaAssets = ref<MediaAsset[]>([])
const activeCategory = ref<'ALL' | EntryCategory>('ALL')
const selectedEntry = ref<Entry | null>(null)
const editingId = ref<number | null>(null)
const mobileOpen = ref(false)
const heroTilt = ref({ x: 0, y: 0 })
const headerScrolled = ref(false)
const activePublicSection = ref('home')
const timelineElement = ref<HTMLElement | null>(null)
const activeTimelineIndex = ref(0)
const activeTimelineImageIndex = ref(0)
const timelineInView = ref(false)
const visibleTimelineEntries = ref(new Set<number>())
const activeDetailImageIndex = ref(0)
const ambientImageIndex = ref(0)
const profileForm = ref<Profile>({ ...seedProfile })
const visualSettings = ref<VisualSettings>({ ...seedVisualSettings })
const entryForm = ref<Entry>({ ...seedEntries[0], images: [], id: 0, year: '', date: '', title: '', place: '', summary: '', body: '', tags: '', image: '', metric: '', category: 'PROJECT' })
let timelineObserver: IntersectionObserver | undefined
let timelineRotationTimer: number | undefined
let publicNavObserver: IntersectionObserver | undefined
let timelineEntryObserver: IntersectionObserver | undefined
let detailRotationTimer: number | undefined
let ambientRotationTimer: number | undefined

onMounted(() => {
  const storedProfile = JSON.parse(localStorage.getItem('fieldnote-profile') || 'null') as Partial<Profile> | null
  const storedEntries = JSON.parse(localStorage.getItem('fieldnote-entries') || 'null') as Array<Partial<Entry>> | null
  const storedVisualSettings = JSON.parse(localStorage.getItem('fieldnote-visual-settings') || 'null') as Partial<VisualSettings> | null
  profile.value = { ...seedProfile, ...storedProfile, theme: storedProfile?.theme ?? seedProfile.theme }
  visualSettings.value = { ...seedVisualSettings, ...storedVisualSettings }
  entries.value = hydrateEntries(storedEntries)
  localStorage.setItem('fieldnote-entries', JSON.stringify(entries.value))
  const storedMedia = JSON.parse(localStorage.getItem('fieldnote-media') || 'null') as MediaAsset[] | null
  mediaAssets.value = hydrateMediaAssets(storedMedia)
  localStorage.setItem('fieldnote-media', JSON.stringify(mediaAssets.value))
  profileForm.value = { ...profile.value }
  if (window.location.hash === '#/admin') activeView.value = 'admin'
  updateHeaderState()
  window.addEventListener('scroll', updateHeaderState, { passive: true })
  window.addEventListener('keydown', handleGlobalKeydown)
  nextTick(() => {
    if ('IntersectionObserver' in window) {
      publicNavObserver = new IntersectionObserver((entries) => {
        const visibleSection = entries.find(entry => entry.isIntersecting)
        if (visibleSection) activePublicSection.value = visibleSection.target.id
      }, { rootMargin: '-35% 0px -55% 0px', threshold: 0 })
      document.querySelectorAll<HTMLElement>('[data-nav-section]').forEach(section => publicNavObserver?.observe(section))
    }
    if (!timelineElement.value || !('IntersectionObserver' in window)) {
      timelineInView.value = true
      visibleTimelineEntries.value = new Set(entries.value.map(entry => entry.id))
      syncTimelineRotation()
      return
    }
    timelineObserver = new IntersectionObserver(([entry]) => {
      timelineInView.value = entry.isIntersecting
      syncTimelineRotation()
    }, { threshold: .18 })
    timelineObserver.observe(timelineElement.value)
    timelineEntryObserver = new IntersectionObserver((observedEntries) => {
      observedEntries.forEach((entry) => {
        if (!entry.isIntersecting) return
        const target = entry.target as HTMLElement
        const id = Number(target.dataset.traceId)
        const index = Number(target.dataset.traceIndex)
        visibleTimelineEntries.value = new Set([...visibleTimelineEntries.value, id])
        setActiveTimelineEntry(index)
        timelineEntryObserver?.unobserve(target)
      })
    }, { threshold: .28, rootMargin: '0px 0px -10% 0px' })
    document.querySelectorAll<HTMLElement>('[data-trace-id]').forEach(entry => timelineEntryObserver?.observe(entry))
  })
  window.addEventListener('visibilitychange', syncAllRotations)
  syncAmbientRotation()
})

onBeforeUnmount(() => {
  window.removeEventListener('scroll', updateHeaderState)
  window.removeEventListener('keydown', handleGlobalKeydown)
  publicNavObserver?.disconnect()
  window.removeEventListener('visibilitychange', syncAllRotations)
  timelineObserver?.disconnect()
  timelineEntryObserver?.disconnect()
  stopTimelineRotation()
  stopDetailRotation()
  stopAmbientRotation()
})

const interestList = computed(() => profile.value.interests.split(',').map(item => item.trim()).filter(Boolean))
const sortedEntries = computed(() => [...entries.value].sort((a, b) => b.year.localeCompare(a.year)))
const featuredEntries = computed(() => {
  const source = activeCategory.value === 'ALL' ? sortedEntries.value : sortedEntries.value.filter(item => item.category === activeCategory.value)
  return source.slice(0, 4)
})
const backgroundCandidates = computed(() => [...new Set(sortedEntries.value.flatMap(entry => entry.images))])
const selectedBackgroundImages = computed(() => {
  const selected = visualSettings.value.selectedBackgroundImages.filter(image => backgroundCandidates.value.includes(image))
  return selected.length ? selected : backgroundCandidates.value
})
const ambientImage = computed(() => selectedBackgroundImages.value[ambientImageIndex.value % Math.max(selectedBackgroundImages.value.length, 1)] || profile.value.avatar)
const heroCoverImage = computed(() => visualSettings.value.coverImage || profile.value.avatar)
const activeTimelineEntry = computed(() => sortedEntries.value[activeTimelineIndex.value % Math.max(sortedEntries.value.length, 1)] ?? seedEntries[0])
const timelineBackground = computed(() => activeTimelineEntry.value.images[activeTimelineImageIndex.value % activeTimelineEntry.value.images.length] ?? activeTimelineEntry.value.image)
const detailImage = computed(() => {
  const entry = selectedEntry.value
  return entry?.images[activeDetailImageIndex.value % entry.images.length] ?? entry?.image ?? ''
})

function normalizeCategory(value: string | undefined): EntryCategory {
  if (value === 'CURRENTLY') return 'NOW'
  if (value === 'SERIES') return 'WORK'
  return categoryOptions.includes(value as EntryCategory) ? value as EntryCategory : 'PROJECT'
}
function normalizeEntry(item: Partial<Entry>): Entry {
  const image = item.image || seedEntries[0].image
  const suppliedImages = item.images?.filter(Boolean)
  const images = suppliedImages?.length ? suppliedImages : [image]
  return { ...seedEntries[0], ...item, image, images, id: Number(item.id || Date.now()), category: normalizeCategory(item.category) }
}
function hydrateEntries(stored: Array<Partial<Entry>> | null): Entry[] {
  if (!stored) return seedEntries
  const fixtures = new Map(seedEntries.map(entry => [entry.id, entry]))
  return stored.map((entry) => {
    const fixture = fixtures.get(Number(entry.id))
    return normalizeEntry({ ...fixture, ...entry, images: entry.images?.length ? entry.images : fixture?.images })
  })
}
function hydrateMediaAssets(stored: MediaAsset[] | null): MediaAsset[] {
  if (!stored) return seedMedia
  const fixtures = new Map(seedMedia.map(asset => [asset.id, asset]))
  const hydrated = stored.map(asset => {
    const fixture = fixtures.get(asset.id)
    return fixture?.type === 'VIDEO' ? { ...asset, ...fixture } : asset
  })
  const existingIds = new Set(hydrated.map(asset => asset.id))
  return [...hydrated, ...seedMedia.filter(asset => asset.type === 'VIDEO' && !existingIds.has(asset.id))]
}

function saveProfile() {
  profile.value = { ...profileForm.value }
  localStorage.setItem('fieldnote-profile', JSON.stringify(profile.value))
}
function saveVisualSettings() {
  localStorage.setItem('fieldnote-visual-settings', JSON.stringify(visualSettings.value))
  adminNotice.value = '背景轮播设置已保存'
}
function setTheme(theme: ThemeKey) {
  profileForm.value.theme = theme
  profile.value.theme = theme
  localStorage.setItem('fieldnote-profile', JSON.stringify(profile.value))
}
function setActiveCategory(category: 'ALL' | EntryCategory) { activeCategory.value = category }
function resetEntry() {
  editingId.value = null
  entryForm.value = { ...seedEntries[0], images: [], id: 0, year: '', date: '', title: '', place: '', summary: '', body: '', tags: '', image: '', metric: '', category: 'PROJECT' }
}
function editEntry(item: Entry) { editingId.value = item.id; entryForm.value = { ...item }; adminSection.value = 'entries'; window.scrollTo({ top: 0, behavior: 'smooth' }) }
function saveEntry() {
  if (!entryForm.value.title || !entryForm.value.year || !entryForm.value.summary) return
  const firstGalleryImage = entryForm.value.images.find(Boolean)
  const next = normalizeEntry({ ...entryForm.value, id: editingId.value || Date.now(), image: entryForm.value.image || firstGalleryImage || seedEntries[0].image })
  entries.value = editingId.value ? entries.value.map(item => item.id === editingId.value ? next : item) : [next, ...entries.value]
  localStorage.setItem('fieldnote-entries', JSON.stringify(entries.value)); resetEntry()
}
function setEntryImages(value: string) {
  entryForm.value.images = value.split(',').map(image => image.trim()).filter(Boolean)
}
function isBackgroundImageSelected(image: string) {
  return visualSettings.value.selectedBackgroundImages.length === 0 || visualSettings.value.selectedBackgroundImages.includes(image)
}
function setBackgroundImageSelected(image: string, checked: boolean) {
  const allImages = backgroundCandidates.value
  const selected = visualSettings.value.selectedBackgroundImages.length ? [...visualSettings.value.selectedBackgroundImages] : [...allImages]
  const next = checked ? [...new Set([...selected, image])] : selected.filter(item => item !== image)
  visualSettings.value.selectedBackgroundImages = next.length === allImages.length ? [] : next
  ambientImageIndex.value = 0
  stopAmbientRotation()
  syncAmbientRotation()
}
function uploadCoverImage(event: Event) {
  const file = (event.target as HTMLInputElement).files?.[0]
  if (!file) return
  if (!file.type.startsWith('image/') || file.size > 2 * 1024 * 1024) { adminNotice.value = '请选择小于 2 MB 的图片文件'; return }
  const reader = new FileReader()
  reader.addEventListener('load', () => {
    visualSettings.value.coverImage = String(reader.result)
    saveVisualSettings()
  })
  reader.readAsDataURL(file)
}
function deleteEntry(id: number) {
  entries.value = entries.value.filter(item => item.id !== id)
  localStorage.setItem('fieldnote-entries', JSON.stringify(entries.value))
}
function openPublic() { activeView.value = 'public'; adminAuthenticated.value = false; mobileOpen.value = false; window.location.hash = '' }
function openAdmin() { activeView.value = 'admin'; mobileOpen.value = false; window.location.hash = '/admin' }
function loginAdmin() { adminAuthenticated.value = true; adminSection.value = 'dashboard'; adminNotice.value = '已进入本地模拟管理环境'; }
function logoutAdmin() { adminAuthenticated.value = false; adminNotice.value = ''; openPublic() }
function selectAdminSection(section: AdminSection) { adminSection.value = section; adminNotice.value = '' }
function addMockMedia() {
  const next: MediaAsset = { id: Date.now(), name: `field-note-${mediaAssets.value.length + 1}.jpg`, type: 'IMAGE', size: '2.1 MB', date: new Date().toISOString().slice(0, 10), url: seedEntries[mediaAssets.value.length % seedEntries.length].image }
  mediaAssets.value = [next, ...mediaAssets.value]
  localStorage.setItem('fieldnote-media', JSON.stringify(mediaAssets.value))
  adminNotice.value = '已添加一条 mock 媒体资源'
}
function scrollTo(id: string) { activeView.value = 'public'; activePublicSection.value = id; mobileOpen.value = false; requestAnimationFrame(() => document.getElementById(id)?.scrollIntoView({ behavior: 'smooth' })) }
function updateHeaderState() { headerScrolled.value = window.scrollY > 42 }
function handleGlobalKeydown(event: KeyboardEvent) {
  if (event.key === 'Escape' && selectedEntry.value) selectedEntry.value = null
}
function stopTimelineRotation() {
  if (timelineRotationTimer === undefined) return
  window.clearInterval(timelineRotationTimer)
  timelineRotationTimer = undefined
}
function syncTimelineRotation() {
  const reducedMotion = window.matchMedia('(prefers-reduced-motion: reduce)').matches
  if (!timelineInView.value || document.hidden || reducedMotion || activeTimelineEntry.value.images.length < 2) { stopTimelineRotation(); return }
  if (timelineRotationTimer !== undefined) return
  timelineRotationTimer = window.setInterval(() => {
    activeTimelineImageIndex.value = (activeTimelineImageIndex.value + 1) % activeTimelineEntry.value.images.length
  }, 5000)
}
function setActiveTimelineEntry(index: number) {
  activeTimelineIndex.value = index
  activeTimelineImageIndex.value = 0
  stopTimelineRotation()
  syncTimelineRotation()
}
function stopDetailRotation() {
  if (detailRotationTimer === undefined) return
  window.clearInterval(detailRotationTimer)
  detailRotationTimer = undefined
}
function stopAmbientRotation() {
  if (ambientRotationTimer === undefined) return
  window.clearInterval(ambientRotationTimer)
  ambientRotationTimer = undefined
}
function syncAmbientRotation() {
  const reducedMotion = window.matchMedia('(prefers-reduced-motion: reduce)').matches
  if (document.hidden || reducedMotion || selectedBackgroundImages.value.length < 2) { stopAmbientRotation(); return }
  if (ambientRotationTimer !== undefined) return
  ambientRotationTimer = window.setInterval(() => {
    ambientImageIndex.value = (ambientImageIndex.value + 1) % selectedBackgroundImages.value.length
  }, Math.max(3, visualSettings.value.rotationSeconds) * 1000)
}
function syncDetailRotation() {
  const reducedMotion = window.matchMedia('(prefers-reduced-motion: reduce)').matches
  if (!selectedEntry.value || document.hidden || reducedMotion || selectedEntry.value.images.length < 2) { stopDetailRotation(); return }
  if (detailRotationTimer !== undefined) return
  detailRotationTimer = window.setInterval(() => {
    if (!selectedEntry.value) return
    activeDetailImageIndex.value = (activeDetailImageIndex.value + 1) % selectedEntry.value.images.length
  }, 4200)
}
function syncAllRotations() { syncAmbientRotation(); syncTimelineRotation(); syncDetailRotation() }

watch(selectedEntry, () => {
  activeDetailImageIndex.value = 0
  stopDetailRotation()
  syncDetailRotation()
})

watch([() => visualSettings.value.rotationSeconds, selectedBackgroundImages], () => {
  ambientImageIndex.value = 0
  stopAmbientRotation()
  syncAmbientRotation()
})
function moveHero(event: PointerEvent) {
  if (event.pointerType === 'touch') return
  const bounds = (event.currentTarget as HTMLElement).getBoundingClientRect()
  heroTilt.value = {
    x: Math.max(-1, Math.min(1, ((event.clientX - bounds.left) / bounds.width - .5) * 2)),
    y: Math.max(-1, Math.min(1, ((event.clientY - bounds.top) / bounds.height - .5) * 2)),
  }
}
function resetHero() { heroTilt.value = { x: 0, y: 0 } }
</script>

<template>
  <div class="app-shell" :class="`theme-${profile.theme}`" :style="{ '--ambient-opacity': visualSettings.backgroundOpacity }">
    <Transition name="ambient-layer">
      <div :key="ambientImage" class="ambient-layer" :style="{ '--ambient-image': `url(${ambientImage})` }" aria-hidden="true"></div>
    </Transition>
    <header v-if="activeView === 'public'" class="site-header" :class="{ 'is-scrolled': headerScrolled }">
      <button class="wordmark" @click="openPublic">
        <span class="wordmark-mark"><Compass :size="15" /></span>
        <span>JOURNEY CHRONICLE</span>
      </button>
      <nav class="desktop-nav">
        <button :class="{ active: activePublicSection === 'about' }" @click="scrollTo('about')">概览</button>
        <button :class="{ active: activePublicSection === 'timeline' }" @click="scrollTo('timeline')">路线</button>
        <button :class="{ active: activePublicSection === 'work' }" @click="scrollTo('work')">图集</button>
        <button class="nav-login" @click="openAdmin"><LogIn :size="14" /> 管理</button>
      </nav>
      <button class="icon-button mobile-menu" title="打开菜单" @click="mobileOpen = !mobileOpen"><Menu :size="18" /></button>
      <div v-if="mobileOpen" class="mobile-nav">
        <button :class="{ active: activePublicSection === 'about' }" @click="scrollTo('about')">概览</button>
        <button :class="{ active: activePublicSection === 'timeline' }" @click="scrollTo('timeline')">路线</button>
        <button :class="{ active: activePublicSection === 'work' }" @click="scrollTo('work')">图集</button>
        <button @click="openAdmin">管理</button>
      </div>
    </header>

    <main v-if="activeView === 'public'" class="public-page">
      <section class="hero-section" :style="{ '--tilt-x': heroTilt.x, '--tilt-y': heroTilt.y }" @pointermove="moveHero" @pointerleave="resetHero">
        <div class="hero-backdrop" aria-hidden="true"><img :src="heroCoverImage" alt="" /></div>
        <div class="hero-shade" aria-hidden="true"></div>
        <div class="hero-content page-width">
          <div class="hero-copy reveal">
            <p class="overline"><span class="live-dot"></span> FIELD NOTES / {{ profile.location }}</p>
            <h1>把日子，<br />走成路线。</h1>
            <p class="hero-name">{{ profile.name }} <span>/</span> {{ profile.role }}</p>
            <p class="hero-tagline">{{ profile.tagline }}</p>
            <div class="hero-actions">
              <button class="primary-btn" @click="scrollTo('timeline')">浏览旅行足迹 <ArrowDownRight :size="17" /></button>
              <a class="ghost-btn" :href="`mailto:${profile.email}`">写封信 <Mail :size="16" /></a>
            </div>
          </div>

          <div class="hero-map">
            <div class="hero-base">
              <MapPin :size="16" />
              <div><span>HOME BASE</span><strong>{{ profile.location }}</strong></div>
            </div>
            <div class="route-panel">
              <div class="route-panel-head">
                <span>CURRENT EXPEDITION</span>
                <strong>{{ sortedEntries[0]?.year || 'NOW' }}</strong>
              </div>
              <div class="route-line">
                <span v-for="item in sortedEntries.slice(0, 4)" :key="item.id" class="route-stop" :title="item.title" tabindex="0">
                  <span class="route-tip"><b>{{ item.year }}</b><strong>{{ item.category }} / {{ item.title }}</strong></span>
                </span>
              </div>
              <button v-if="sortedEntries[0]" class="route-current" @click="selectedEntry = sortedEntries[0]">
                <span>{{ sortedEntries[0].category }}</span>
                <strong>{{ sortedEntries[0].title }}</strong>
                <small><MapPin :size="12" /> {{ sortedEntries[0].place }}</small>
              </button>
            </div>
          </div>
        </div>
      </section>

      <section id="about" data-nav-section class="brief-band">
        <div class="page-width brief-grid">
          <article class="brief-story">
            <span class="section-index">01 / ABOUT THE ARCHIVE</span>
            <h2>一份不断延长的个人路线图。</h2>
            <p>{{ profile.bio }}</p>
          </article>
          <div class="brief-stats">
            <article>
              <small>JOURNEYS</small>
              <strong>{{ entries.length.toString().padStart(2, '0') }}</strong>
              <span>stories logged</span>
            </article>
            <article>
              <small>MEDIA ARCHIVE</small>
              <strong>{{ mediaAssets.length.toString().padStart(2, '0') }}</strong>
              <span>photos &amp; video</span>
            </article>
            <article>
              <small>OUTDOOR FOCUS</small>
              <strong>{{ interestList.length.toString().padStart(2, '0') }}</strong>
              <span>ways of moving</span>
            </article>
          </div>
        </div>
        <div class="page-width route-index" aria-label="旅程路线目录">
          <button v-for="(item, index) in sortedEntries.slice(0, 4)" :key="item.id" class="route-index-item" @click="selectedEntry = item">
            <span>0{{ index + 1 }}</span>
            <strong>{{ item.title }}</strong>
            <small><MapPin :size="13" /> {{ item.place }}</small>
            <time>{{ item.year }}</time>
            <ArrowUpRight :size="16" />
          </button>
        </div>
      </section>

      <section id="timeline" ref="timelineElement" data-nav-section class="timeline-section page-width">
        <Transition name="timeline-backdrop">
          <div :key="timelineBackground" class="timeline-backdrop" :style="{ '--timeline-image': `url(${timelineBackground})` }" aria-hidden="true"></div>
        </Transition>
        <div class="section-heading">
          <span class="section-index">02 / ROUTE</span>
          <div>
            <h2>沿着发生的顺序，重走一遍。</h2>
            <p>每个节点保留地点、时间、照片和当时的心情。点击它，打开完整记录。</p>
          </div>
        </div>
        <div class="trace-list">
          <article v-for="(item, index) in sortedEntries" :key="item.id" class="trace-item" :class="{ 'is-active': activeTimelineIndex === index, 'is-visible': visibleTimelineEntries.has(item.id) }" :data-trace-id="item.id" :data-trace-index="index" @click="setActiveTimelineEntry(index); selectedEntry = item">
            <time>{{ item.year }}</time>
            <div class="trace-card">
              <div class="trace-copy">
                <div class="trace-meta"><span>{{ item.category }}</span><span>{{ item.date }}</span></div>
                <h3>{{ item.title }}</h3>
                <p class="trace-place"><MapPin :size="13" /> {{ item.place }}</p>
                <p>{{ item.summary }}</p>
                <div class="mini-tags">
                  <span v-for="tag in item.tags.split(',').slice(0, 3)" :key="tag">{{ tag.trim() }}</span>
                </div>
              </div>
              <div class="trace-media" aria-hidden="true">
                <img :src="item.images[0] || item.image" :alt="item.title" />
                <span>IMAGE PLAYBACK</span>
              </div>
              <button class="round-arrow" title="查看详情"><ArrowUpRight :size="17" /></button>
            </div>
          </article>
        </div>
      </section>

      <section id="work" data-nav-section class="work-section page-width">
        <div class="section-heading">
          <span class="section-index">03 / GALLERY</span>
          <div>
            <h2>路上看见的，都留在这里。</h2>
            <p>从正在发生的远行到城市里的日常观察，按主题筛选这些画面。</p>
          </div>
        </div>
        <div class="work-filters">
          <button v-for="category in publicCategories" :key="category" :class="{ active: activeCategory === category }" @click="setActiveCategory(category)">{{ category }}</button>
        </div>
        <div v-if="featuredEntries.length" class="work-grid">
          <article v-for="(item, index) in featuredEntries" :key="item.id" class="work-card" :class="{ featured: index === 0 }" @click="selectedEntry = item">
            <img :src="item.image" :alt="item.title" />
            <div class="work-overlay">
              <span>{{ item.metric }}</span>
              <h3>{{ item.title }}</h3>
              <p>{{ item.category }} / {{ item.place }}</p>
            </div>
            <div class="work-corner"><ExternalLink :size="15" /></div>
          </article>
        </div>
        <div v-else class="work-empty">这个分类还没有内容，去后台添加第一个节点。</div>
      </section>

      <section class="contact-section page-width">
        <div>
          <p class="overline">NEXT CHAPTER</p>
          <h2>下一段路，可能从一封邮件开始。</h2>
        </div>
        <a class="contact-button" :href="`mailto:${profile.email}`">{{ profile.email }} <ArrowRight :size="17" /></a>
      </section>
    </main>

    <main v-else class="admin-page">
      <section v-if="!adminAuthenticated" class="admin-login page-width">
        <div class="admin-login-mark"><ShieldCheck :size="22" /></div>
        <p class="overline">JOURNEY CHRONICLE / OWNER ACCESS</p>
        <h1>进入旅程内容后台。</h1>
        <p>这里维护公开主页的个人资料、旅程节点、媒体资源和视觉主题。当前是本地模拟登录。</p>
        <div class="admin-login-form">
          <label>管理员邮箱<input value="owner@journey.me" type="email" /></label>
          <label>访问密码<input value="mock-password" type="password" /></label>
          <button class="primary-btn" @click="loginAdmin"><LogIn :size="16" /> 进入后台</button>
        </div>
        <button class="text-link" @click="openPublic">返回公开主页 <ExternalLink :size="15" /></button>
      </section>

      <div v-else class="admin-platform">
        <aside class="admin-sidebar">
          <button class="admin-brand" @click="selectAdminSection('dashboard')">
            <span class="wordmark-mark"><Compass :size="14" /></span>
            <strong>JOURNEY</strong>
          </button>
          <div class="sidebar-group">
            <small>CONTENT</small>
            <button :class="{ active: adminSection === 'dashboard' }" @click="selectAdminSection('dashboard')"><LayoutDashboard :size="15" /> 概览</button>
            <button :class="{ active: adminSection === 'profile' }" @click="selectAdminSection('profile')"><Edit3 :size="15" /> 个人资料</button>
            <button :class="{ active: adminSection === 'entries' }" @click="selectAdminSection('entries')"><BarChart3 :size="15" /> 旅程节点 <span>{{ entries.length }}</span></button>
            <button :class="{ active: adminSection === 'media' }" @click="selectAdminSection('media')"><Image :size="15" /> 媒体 <span>{{ mediaAssets.length }}</span></button>
          </div>
          <div class="sidebar-group">
            <small>SYSTEM</small>
            <button :class="{ active: adminSection === 'settings' }" @click="selectAdminSection('settings')"><Settings :size="15" /> 设置</button>
          </div>
          <div class="sidebar-footer">
            <button @click="openPublic"><Eye :size="15" /> 公开页</button>
            <button @click="logoutAdmin"><LogOut :size="15" /> 退出</button>
          </div>
        </aside>

        <section class="admin-content">
          <header class="admin-toolbar">
            <div>
              <p class="overline">EDITOR WORKSPACE</p>
              <h1>{{ adminSection === 'dashboard' ? '内容总览' : adminSection === 'profile' ? '个人资料' : adminSection === 'entries' ? '旅程节点' : adminSection === 'media' ? '媒体资源' : '公开设置' }}</h1>
              <p>{{ adminSection === 'dashboard' ? '从这里检查公开主页内容、最近节点和媒体数量。' : '修改会写入本地浏览器存储，用于演示后台体验。' }}</p>
            </div>
            <button class="ghost-btn" @click="openPublic"><Eye :size="15" /> 预览</button>
          </header>
          <div v-if="adminNotice" class="admin-notice"><Check :size="15" /> {{ adminNotice }}</div>

          <section v-if="adminSection === 'dashboard'" class="admin-dashboard">
            <div class="admin-metric-row">
              <article><small>PUBLIC VIEWS</small><strong>2,480</strong><span>mock / 30d</span></article>
              <article><small>TRACE NODES</small><strong>{{ entries.length.toString().padStart(2, '0') }}</strong><span>published</span></article>
              <article><small>MEDIA ASSETS</small><strong>{{ mediaAssets.length.toString().padStart(3, '0') }}</strong><span>library</span></article>
            </div>
            <div class="admin-section-heading">
              <div><span class="panel-kicker">RECENT ROUTE</span><h2>最近节点</h2></div>
              <button class="ghost-btn" @click="selectAdminSection('entries')">管理全部 <ArrowRight :size="15" /></button>
            </div>
            <div class="admin-manager-list">
              <article v-for="item in sortedEntries.slice(0, 4)" :key="item.id" class="admin-manager-row">
                <span class="manager-year">{{ item.year }}</span>
                <div><strong>{{ item.title }}</strong><span>{{ item.category }} / {{ item.place }}</span></div>
                <button class="icon-button" title="编辑节点" @click="editEntry(item)"><Edit3 :size="15" /></button>
              </article>
            </div>
            <div class="admin-boundary"><ShieldCheck :size="18" /><div><strong>公开页面保持只读</strong><p>访客只能查看已发布内容；新增、删除和主题切换都留在后台。</p></div><span>OWNER ONLY</span></div>
          </section>

          <section v-else-if="adminSection === 'profile'" class="admin-panel">
            <div class="panel-title"><div><span class="panel-kicker">PROFILE</span><h2>编辑公开身份</h2></div><button class="primary-btn" @click="saveProfile"><Save :size="16" /> 保存</button></div>
            <div class="editor-grid">
              <label>你的名字<input v-model="profileForm.name" /></label>
              <label>头像字母<input v-model="profileForm.initials" maxlength="3" /></label>
              <label>身份 / 角色<input v-model="profileForm.role" /></label>
              <label>所在位置<input v-model="profileForm.location" /></label>
              <label class="wide">一句话介绍<textarea v-model="profileForm.tagline" rows="2" /></label>
              <label class="wide">个人简介<textarea v-model="profileForm.bio" rows="5" /></label>
              <label>对外状态<input v-model="profileForm.availability" /></label>
              <label>联系邮箱<input v-model="profileForm.email" type="email" /></label>
              <label class="wide">头像图片地址<input v-model="profileForm.avatar" /></label>
              <label class="wide">兴趣标签（用逗号分隔）<input v-model="profileForm.interests" placeholder="徒步, 摄影, 滑雪" /></label>
            </div>
            <div class="editor-preview"><span>LIVE PREVIEW</span><strong>{{ profileForm.name || '你的名字' }}</strong><small>{{ profileForm.role || '你的身份 / 角色' }}</small></div>
          </section>

          <section v-else-if="adminSection === 'entries'" class="admin-panel">
            <div class="panel-title"><div><span class="panel-kicker">TRACE</span><h2>{{ editingId ? '编辑节点' : '添加节点' }}</h2></div><button v-if="editingId" class="icon-button" title="取消编辑" @click="resetEntry"><X :size="17" /></button></div>
            <div class="editor-grid">
              <label>年份<input v-model="entryForm.year" placeholder="2025" /></label>
              <label>节点日期<input v-model="entryForm.date" placeholder="2025.02 - 至今" /></label>
              <label>节点类型<select v-model="entryForm.category"><option v-for="category in categoryOptions" :key="category">{{ category }}</option></select></label>
              <label>地点<input v-model="entryForm.place" placeholder="中国 / 西南线" /></label>
              <label class="wide">标题<input v-model="entryForm.title" placeholder="这个节点发生了什么？" /></label>
              <label class="wide">一句话摘要<textarea v-model="entryForm.summary" rows="2" /></label>
              <label class="wide">详细介绍<textarea v-model="entryForm.body" rows="5" /></label>
              <label class="wide">标签（用逗号分隔）<input v-model="entryForm.tags" placeholder="摄影, 纪实, 长期项目" /></label>
              <label class="wide">封面图片地址<input v-model="entryForm.image" placeholder="https://..." /></label>
              <label class="wide">图片集地址（用逗号分隔）<textarea :value="entryForm.images.join(', ')" rows="3" placeholder="https://..., https://..., https://..." @input="setEntryImages(($event.target as HTMLTextAreaElement).value)" /></label>
              <label>数据标记<input v-model="entryForm.metric" placeholder="312 km / 03 days" /></label>
            </div>
            <div class="editor-actions">
              <button class="ghost-btn" @click="resetEntry">清空</button>
              <button class="primary-btn" :disabled="!entryForm.title || !entryForm.year || !entryForm.summary" @click="saveEntry"><Check :size="16" /> {{ editingId ? '保存修改' : '发布节点' }}</button>
            </div>
            <div class="admin-section-heading"><div><span class="panel-kicker">MANAGE</span><h2>已有节点</h2></div><span class="admin-count">{{ entries.length }} nodes</span></div>
            <div class="admin-manager-list">
              <article v-for="item in sortedEntries" :key="item.id" class="admin-manager-row">
                <span class="manager-year">{{ item.year }}</span>
                <div><strong>{{ item.title }}</strong><span>{{ item.category }} / {{ item.place }}</span></div>
                <div class="row-actions"><button class="icon-button" title="编辑节点" @click="editEntry(item)"><Edit3 :size="15" /></button><button class="icon-button danger" title="删除节点" @click="deleteEntry(item.id)"><Trash2 :size="15" /></button></div>
              </article>
            </div>
          </section>

          <section v-else-if="adminSection === 'media'" class="admin-panel">
            <div class="panel-title"><div><span class="panel-kicker">MEDIA</span><h2>管理媒体库</h2></div><button class="primary-btn" @click="addMockMedia"><Upload :size="16" /> 添加 mock 媒体</button></div>
            <div class="media-dropzone"><Upload :size="22" /><strong>拖入图片或视频</strong><span>当前无服务器，先用本地 mock 数据维护上传效果。</span></div>
            <div class="media-grid">
              <article v-for="asset in mediaAssets" :key="asset.id" class="media-card">
                <div class="media-thumb">
                  <video v-if="asset.type === 'VIDEO'" :src="asset.url" muted loop autoplay playsinline preload="metadata"></video>
                  <img v-else :src="asset.url" :alt="asset.name" />
                  <span v-if="asset.type === 'VIDEO'"><Video :size="14" /> VIDEO</span>
                </div>
                <div><strong>{{ asset.name }}</strong><small>{{ asset.type }} / {{ asset.size }} / {{ asset.date }}</small></div>
              </article>
            </div>
          </section>

          <section v-else class="admin-panel">
            <div class="panel-title"><div><span class="panel-kicker">SETTINGS</span><h2>公开页面设置</h2></div></div>
            <div class="setting-block"><div><strong>公开状态</strong><span>分享链接当前可被访问，访客只能读取已发布内容。</span></div><span class="status-pill"><i></i> PUBLIC / READ ONLY</span></div>
            <div class="setting-block visual-setting-block">
              <div><strong>全站背景轮播</strong><span>默认轮播所有节点图集；取消勾选即可限定参与轮播的画面。</span></div>
              <div class="visual-control-grid">
                <label>轮播间隔（秒）<input v-model.number="visualSettings.rotationSeconds" type="number" min="3" max="30" step="1" /></label>
                <label>背景透明度 <output>{{ Math.round(visualSettings.backgroundOpacity * 100) }}%</output><input v-model.number="visualSettings.backgroundOpacity" type="range" min=".03" max=".28" step=".01" /></label>
                <label class="wide">从节点图片选择封面<select v-model="visualSettings.coverImage"><option value="">使用个人资料封面</option><option v-for="image in backgroundCandidates" :key="image" :value="image">节点图片 {{ backgroundCandidates.indexOf(image) + 1 }}</option></select></label>
                <label class="wide">封面图片地址<input v-model="visualSettings.coverImage" placeholder="https://..." /></label>
              </div>
              <label class="cover-upload"><Upload :size="16" /> 上传封面图片<input type="file" accept="image/*" @change="uploadCoverImage" /></label>
              <div class="background-picker" aria-label="选择全站背景轮播图片">
                <label v-for="(image, index) in backgroundCandidates" :key="image" class="background-choice" :class="{ selected: isBackgroundImageSelected(image) }">
                  <input type="checkbox" :checked="isBackgroundImageSelected(image)" @change="setBackgroundImageSelected(image, ($event.target as HTMLInputElement).checked)" />
                  <img :src="image" :alt="`节点图片 ${index + 1}`" />
                  <span>图 {{ index + 1 }}</span>
                </label>
              </div>
              <button class="primary-btn" @click="saveVisualSettings"><Save :size="16" /> 保存背景设置</button>
            </div>
            <div class="setting-block">
              <div><strong>选择视觉主题</strong><span>主题配置会同步到公开主页，内容与权限保持不变。</span></div>
              <div class="appearance-options">
                <button v-for="theme in themeOptions" :key="theme.id" class="appearance-choice" :class="{ selected: profile.theme === theme.id }" @click="setTheme(theme.id)">
                  <span class="swatch" :class="`swatch-${theme.id}`"></span>
                  <strong>{{ theme.label }}</strong>
                  <small>{{ theme.note }}</small>
                </button>
              </div>
            </div>
            <div class="admin-boundary"><ShieldCheck :size="18" /><div><strong>权限边界</strong><p>公开页不提供写入接口；接入正式服务器后，这里可替换为真实账号、会话和内容 API。</p></div></div>
          </section>
        </section>
      </div>
    </main>

    <footer v-if="activeView === 'public'" class="site-footer page-width">
      <span>© {{ new Date().getFullYear() }} {{ profile.name }} / JOURNEY CHRONICLE</span>
      <span>BUILT FOR LIVING ARCHIVES <Sparkles :size="14" /></span>
    </footer>

    <div v-if="selectedEntry" class="detail-backdrop" @click.self="selectedEntry = null">
      <article class="detail-modal">
        <button class="modal-close icon-button" title="关闭详情" @click="selectedEntry = null"><X :size="17" /></button>
        <Transition name="detail-image">
          <img :key="detailImage" :src="detailImage" :alt="selectedEntry.title" />
        </Transition>
        <div class="modal-content">
          <div class="trace-meta"><span>{{ selectedEntry.category }}</span><span>{{ selectedEntry.date }}</span></div>
          <h2>{{ selectedEntry.title }}</h2>
          <p class="trace-place"><MapPin :size="13" /> {{ selectedEntry.place }}</p>
          <p class="modal-body">{{ selectedEntry.body }}</p>
          <div class="mini-tags"><span v-for="tag in selectedEntry.tags.split(',')" :key="tag">{{ tag.trim() }}</span></div>
          <div class="modal-metric">{{ selectedEntry.metric }}</div>
        </div>
      </article>
    </div>
  </div>
</template>
