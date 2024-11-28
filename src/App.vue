// src/App.vue
<template>
  <div class="min-h-screen transition-colors duration-300" :class="[
    isDarkMode
      ? 'bg-gradient-to-br from-gray-900 to-gray-800 text-white'
      : 'bg-gradient-to-br from-blue-50 to-indigo-50 text-gray-900'
  ]">
    <!-- Toast Notification -->
    <div v-if="showToast"
      class="fixed top-1 left-1/2 transform -translate-x-1/2 bg-gradient-to-r from-green-600 to-blue-600 text-white px-6 py-3 rounded-lg shadow-lg transition-all duration-300 z-50"
      :class="{ 'opacity-0 translate-y-2': isToastFading }">
      {{ toastMessage }}
    </div>

    <!-- 顶部状态栏 -->
    <div :class="[
      'fixed top-0 left-0 right-0 z-40 px-4 py-2',
      isDarkMode ? 'bg-gray-900/80' : 'bg-white/80',
      'backdrop-blur-md border-b',
      isDarkMode ? 'border-gray-800' : 'border-gray-200'
    ]">
      <div class="max-w-6xl mx-auto flex justify-between items-center">
        <!-- 左侧播放信息 - 添加溢出保护 -->
        <div class="text-xs flex items-center gap-2 min-w-0 flex-shrink">
          <span class="truncate max-w-[200px]" :class="{ 'animate-pulse': isPlaying }">
            {{ isPlaying ? `🎵 正在播放-${musicName}` : 'Ready' }}
          </span>
        </div>

        <!-- 右侧控制区 - 防止收缩 -->
        <div class="flex items-center gap-4 flex-shrink-0">
          <!-- 音质选择器 -->
          <div class="relative flex-shrink-0">
            <label for="quality-select" class="sr-only">选择音质</label>
            <div class="relative">
              <select v-model="selectedQuality" id="quality-select" :class="[
                'appearance-none pl-8 pr-10 py-2 rounded-lg transition-all duration-200',
                'focus:ring-2 focus:ring-blue-500 focus:outline-none cursor-pointer',
                'text-center',
                isDarkMode
                  ? 'bg-gray-800/50 hover:bg-gray-700/50 text-gray-200'
                  : 'bg-white/50 hover:bg-white/80 text-gray-700',
                'backdrop-blur'
              ]">
                <option v-for="quality in audioQualities" :key="quality.value" :value="quality.value">
                  {{ quality.label }}
                </option>
              </select>
              <span class="absolute left-2.5 top-1/2 -translate-y-1/2 text-blue-500">◉</span>
              <span class="absolute right-2 top-1/2 -translate-y-1/2 pointer-events-none"
                :class="isDarkMode ? 'text-gray-400' : 'text-gray-600'">
                ▼
              </span>
            </div>
          </div>

          <!-- 播放列表按钮 -->
          <button @click="togglePlaylistView" :class="[
            'flex items-center gap-2 px-2 py-2 rounded-lg transition flex-shrink-0',
            isDarkMode ? 'bg-blue-600/20 hover:bg-blue-600/30' : 'bg-blue-100 hover:bg-blue-200'
          ]">
            <span>🔖</span>
            <span :class="[
              'text-xs px-2 py-0.5 rounded-full',
              isDarkMode ? 'bg-blue-500' : 'bg-blue-400'
            ]">{{ playlist.length }}</span>
          </button>

          <!-- 主题切换按钮 -->
          <button @click="toggleTheme" class="p-2 hover:text-blue-400 transition flex-shrink-0">
            {{ isDarkMode ? '🌙' : '☀️' }}
          </button>
        </div>
      </div>
    </div>

    <div class="max-w-6xl mx-auto pt-16 pb-24 px-4">
      <!-- Logo和标题区域 -->
      <div class="flex flex-col items-center text-center space-y-2 mb-8">
        <div class="text-4xl">🎵</div>
        <h1 class="text-3xl font-bold bg-gradient-to-r from-blue-500 to-purple-500 bg-clip-text text-transparent">
          Music Player
        </h1>
        <p :class="[
          'text-sm',
          isDarkMode ? 'text-gray-400' : 'text-gray-600'
        ]">
          今日无事，勾栏听曲
        </p>
      </div>

      <div class="mb-8 transform transition-all" :class="{ 'scale-105': isSearchFocused }">
        <!-- 将 flex gap-2 改为响应式布局 -->
        <div class="relative group flex flex-col md:flex-row gap-2">
          <!-- 搜索框部分 -->
          <div class="relative flex-1">
            <div class="absolute inset-y-0 left-4 flex items-center pointer-events-none">
              <font-awesome-icon icon="search" />
            </div>
            <input v-model="searchQuery" @focus="isSearchFocused = true" @blur="isSearchFocused = false"
              @keyup.enter="searchMusic" :class="[
                'w-full pl-12 pr-4 py-3 rounded-xl transition-all duration-300',
                'focus:ring-2 focus:ring-blue-500 focus:outline-none',
                isDarkMode ? 'bg-gray-800/50 focus:bg-gray-800/80' : 'bg-white/50 focus:bg-white/80',
                'backdrop-blur'
              ]" placeholder="搜索音乐..." />
            <button @click="searchMusic"
              class="absolute right-2 top-1/2 -translate-y-1/2 px-6 py-2 bg-blue-600 rounded-lg hover:bg-blue-700 transition flex items-center gap-2 text-white">
              <span class="hidden md:inline">搜索</span>
              <span><font-awesome-icon icon="search" /></span>
            </button>
          </div>

          <!-- 音源选择器 -->
          <div class="relative w-full md:w-auto"> <!-- 移动端下占满宽度 -->
            <label for="source-select" class="sr-only">选择音源</label>
            <div class="relative">
              <select v-model="selectedSource" id="source-select" :class="[
                'w-full md:w-auto', // 移动端下占满宽度
                'appearance-none pl-8 pr-10 py-2 rounded-lg transition-all duration-200',
                'focus:ring-2 focus:ring-blue-500 focus:outline-none cursor-pointer',
                'text-center',
                isDarkMode
                  ? 'bg-gray-800/50 hover:bg-gray-700/50 text-gray-200'
                  : 'bg-white/50 hover:bg-white/80 text-gray-700',
                'backdrop-blur'
              ]">
                <option v-for="source in musicSources" :key="source.value" :value="source.value">
                  {{ source.label }}
                </option>
              </select>
              <span class="absolute left-2.5 top-1/2 -translate-y-1/2 text-blue-500">♫</span>
              <span class="absolute right-2 top-1/2 -translate-y-1/2 pointer-events-none"
                :class="isDarkMode ? 'text-gray-400' : 'text-gray-600'">
                ▼
              </span>
            </div>
          </div>
        </div>
        <!-- 新增：排行榜导航 -->
        <div v-if="QQList" class="mt-4 flex justify-center space-x-2 md:space-x-4">
          <button v-for="rank in rankOptions" :key="rank.value" @click="handleRankSelect(rank.value)" :class="[
            'px-4 py-2 rounded-lg transition-all duration-300 text-sm md:text-base',
            selectedRank === rank.value
              ? 'bg-blue-600 text-white'
              : isDarkMode
                ? 'bg-gray-700 text-gray-300 hover:bg-gray-600'
                : 'bg-gray-200 text-gray-700 hover:bg-gray-300'
          ]">
            {{ rank.label }}
          </button>
        </div>
      </div>

      <!-- 搜索结果 -->
      <div class="space-y-2">
        <div v-for="(song, index) in formattedSearchResults" :key="song.id" :id="`${index + 1}`" :class="[
          'group relative flex items-center gap-4 p-4 rounded-xl transition-all duration-300',
          isDarkMode
            ? 'bg-gray-800/30 hover:bg-gray-700/50'
            : 'bg-white/30 hover:bg-white/50',
          'backdrop-blur cursor-pointer hover:scale-[1.02]'
        ]">
          <div class="relative">
            <!-- 将点击事件绑定到 img 元素 -->
            <img :src="song.cover" class="w-16 h-16 rounded-lg shadow-lg object-cover cursor-pointer" :alt="song.song" />
            <div @click="playSong(song)"
              class="absolute inset-0 bg-black/40 group-hover:bg-black/20 rounded-lg flex items-center justify-center opacity-0 group-hover:opacity-100 transition-all">
              <span class="text-2xl"><font-awesome-icon icon="play" /></span>
            </div>
          </div>
          <div class="flex-1 min-w-0">
            <h3 class="font-semibold truncate">{{ song.song }}</h3>
            <p :class="[
              'truncate',
              isDarkMode ? 'text-gray-400' : 'text-gray-600'
            ]">{{ song.singer }}</p>
          </div>
          <div class="flex gap-2 opacity-0 group-hover:opacity-100 transition-all">
            <button @click.stop="addToPlaylist(song)"
              class="px-4 py-2 bg-blue-600/20 rounded hover:bg-blue-600/40 transition">
              添加到列表
            </button>
            <button @click.stop="downloadSong(song)"
              class="px-4 py-2 bg-purple-600/20 rounded hover:bg-purple-600/40 transition">
              下载
            </button>
          </div>

          <!-- 正在播放标识 -->
          <div v-if="currentSong?.id === song.id"
            class="absolute right-2 top-2 text-xs px-2 py-1 rounded-full bg-blue-500 text-white">
            正在播放
          </div>
        </div>
      </div>


      <!-- 侧边播放列表 -->
      <div class="fixed top-0 right-0 bottom-0 w-full md:w-96 transform transition-all duration-300 ease-in-out z-50"
        :class="[
          showPlaylist ? 'translate-x-0' : 'translate-x-full',
          isDarkMode
            ? 'bg-gray-800/95 backdrop-blur border-l border-gray-700'
            : 'bg-white/95 backdrop-blur border-l border-gray-200'
        ]">
        <div class="h-full flex flex-col p-4">
          <div class="flex justify-between items-center mb-4">
            <h2 class="text-xl font-bold flex items-center gap-2">
              播放列表
              <span :class="[
                'text-xs px-2 py-0.5 rounded-full',
                isDarkMode ? 'bg-blue-500' : 'bg-blue-400'
              ]">{{ playlist.length }}</span>
            </h2>
            <div class="flex gap-2">
              <button @click="downloadPlaylist" v-if="playlist.length > 0"
                class="text-sm px-3 py-1 bg-purple-600/20 rounded hover:bg-purple-600/40 transition">
                下载全部
              </button>
              <button @click="clearPlaylist" v-if="playlist.length > 0"
                class="text-sm px-3 py-1 bg-red-600/20 rounded hover:bg-red-600/40 transition">
                清空
              </button>
              <button @click="togglePlaylistView" class="text-2xl">&times;</button>
            </div>
          </div>
          <!-- 可视化区域 -->
          <div v-if="currentSong"
            class="mb-4 aspect-video rounded-lg overflow-hidden bg-gradient-to-br from-blue-500 to-purple-500"
            ref="visualizer"></div>
          <div class="flex-1 overflow-y-auto">
            <div class="space-y-2">
              <div v-for="(song, index) in playlist" :key="song.id" :class="[
                'flex items-center gap-2 p-2 rounded-lg transition-all hover:scale-[1.02]',
                currentSong?.id === song.id
                  ? isDarkMode
                    ? 'bg-blue-600/20'
                    : 'bg-blue-500/20'
                  : isDarkMode
                    ? 'hover:bg-gray-700/50'
                    : 'hover:bg-white/50'
              ]">
                <span class="w-6 text-center text-sm text-gray-400">{{ index + 1 }}</span>
                <img :src="song.cover" class="w-10 h-10 rounded-lg object-cover" :alt="song.song" />
                <div class="flex-1 min-w-0">
                  <h3 class="font-semibold truncate">{{ song.song }}</h3>
                  <p class="text-gray-400 text-sm truncate">{{ song.singer }}</p>
                  <p class="text-gray-400 text-sm truncate">来源：{{ song.uname }}</p>
                </div>
                <div class="flex gap-1">
                  <button @click="playSong(song)" class="p-2 hover:text-blue-400 transition"><font-awesome-icon
                      icon="play" /></button>
                  <button @click="downloadSong(song)" class="p-2 hover:text-purple-400 transition"><font-awesome-icon
                      icon="download" /></button>
                  <button @click="removeFromPlaylist(index)" class="p-2 hover:text-red-400 transition"><font-awesome-icon
                      icon="trash" /></button>
                </div>
              </div>
            </div>
            <div v-if="playlist.length === 0" class="flex flex-col items-center justify-center h-full text-gray-500">
              <span class="text-4xl mb-4">🎵</span>
              <p>播放列表为空</p>
              <p class="text-sm mt-2">搜索并添加歌曲到播放列表</p>
            </div>
          </div>
        </div>
      </div>


      <!-- 播放器控制栏 -->
      <div class="fixed bottom-0 left-0 right-0  w-full transition-all duration-300 z-30">
        <!-- 歌曲播放页 -->
        <div style="height: 100vh;"
          class="flex flex-col items-center justify-center absolute bottom-0 left-0 right-0 w-full transition-all duration-300 z-30 px-4"
          :class="[
            { 'translate-y-full': !isDetail },
            isDarkMode
              ? 'bg-gray-900/95 backdrop-blur border-t border-gray-800'
              : 'bg-white/95 backdrop-blur border-t border-gray-200'
          ]">

          <!-- 旋转的歌曲封面 - 修复了z-index和定位问题 -->
          <div class="relative w-56 h-56 mb-8 z-10">
            <img v-if="currentSong?.cover" :src="currentSong.cover" :class="[
              'absolute inset-0 w-full h-full rounded-full object-cover bg-center shadow-xl',
              isPlaying ? 'animate-spin-slow' : ''
            ]" :alt="currentSong?.song" style="object-fit: cover;" />
            <div class="absolute inset-0 rounded-full -z-10" :class="isDarkMode ? 'bg-gray-800' : 'bg-gray-100'"></div>
          </div>

          <!-- 歌曲信息 -->
          <div class="text-center mb-8 w-full max-w-md px-4">
            <h2 class="text-2xl font-bold mb-2 break-words">{{ currentSong?.song }}</h2>
            <p class="text-gray-400">{{ currentSong?.singer }}</p>
          </div>

          <!-- 歌词显示区域 - 修复了横向滚动问题 -->
          <div class="w-full max-w-md h-48 overflow-hidden relative"
            :class="isDarkMode ? 'text-gray-300' : 'text-gray-700'">
            <div class="absolute inset-0 flex items-center pointer-events-none z-20">
              <div class="w-full h-8 bg-gradient-to-b from-current to-transparent opacity-10"></div>
            </div>
            <div class="h-full overflow-y-auto hide-scrollbar px-4" ref="lyricsContainer">
              <div class="space-y-4 text-center py-20">
                <p v-for="(line, index) in currentSong?.lyrics" :key="index" @click="handleLyricClick(line)" :class="[
                  'transition-all duration-300 whitespace-normal break-words px-4 cursor-pointer hover:opacity-80',
                  currentLyricIndex === index
                    ? 'text-blue-500 font-medium'
                    : 'text-gray-400'
                ]" style="transform: scale(var(--scale)); transform-origin: center center;" :style="{
  '--scale': currentLyricIndex === index ? '1.1' : '1'
}">
                  {{ line.name }}
                </p>
              </div>
            </div>
            <div class="absolute inset-0 flex items-center pointer-events-none z-20">
              <div class="w-full h-8 bg-gradient-to-t from-current to-transparent opacity-10"></div>
            </div>
          </div>
        </div>
        <div class="absolute bottom-0 left-0 right-0 w-full transition-all duration-300 z-40" :class="[
          { 'translate-y-full': !currentSong },

          isDarkMode
            ? 'bg-gray-900/95 backdrop-blur border-t border-gray-800'
            : 'bg-white/95 backdrop-blur border-t border-gray-200'
        ]">
          <div class="absolute top-0 left-0 right-0 -translate-y-full">
            <div class="relative h-1 bg-gray-700">
              <div class="absolute h-full bg-blue-600" :style="{ width: `${progress}%` }"></div>
              <input type="range" v-model="currentTime" min="0" :max="duration" step="1"
                class="absolute w-full h-full opacity-0 cursor-pointer" @input="seekTo" />
            </div>
          </div>

          <div class="max-w-8xl mx-auto px-2 py-3">
            <div class="flex items-center justify-center gap-2 ">
              <!-- Same as main player controls -->
              <!-- <div class="flex items-center gap-4 flex-1 min-w-0">
                <div class="relative group">
                  <img v-if="currentSong" :src="currentSong.cover" class="w-12 h-12 rounded-lg shadow-lg object-cover"
                    :alt="currentSong.song" />
                </div>
              </div> -->

              <div class="flex items-center gap-1 text-sm text-gray-400">
                <div @click="toggleDetail" style="cursor: pointer; display: inline-flex; align-items: center;">
                  <font-awesome-icon :icon="isDetail ? 'angle-down' : 'angle-up'" />
                  <!-- <span style="margin-left: 8px;">{{ isDetail ? '折叠' : '展开' }}</span> -->
                </div>
              </div>
              <div class="flex items-center gap-1 text-sm text-gray-400">
                <span>{{ formatTime(currentTime) }}</span>
                <span>/</span>
                <span>{{ formatTime(duration) }}</span>
              </div>

              <div class="flex items-center gap-4">
                <button @click="toggleShuffle" :class="{ 'text-blue-500': isShuffleMode }"
                  class="hover:text-blue-400 transition">
                  <font-awesome-icon icon="random" />
                </button>
                <button @click="previousSong" class="hover:text-blue-400 transition">
                  <font-awesome-icon icon="step-backward" />
                </button>
                <button @click="togglePlay"
                  class="w-10 h-10 flex items-center justify-center rounded-full bg-blue-600 hover:bg-blue-700 transition">
                  <font-awesome-icon v-if="isPlaying" icon="pause" />
                  <font-awesome-icon v-else icon="play" />
                </button>
                <button @click="nextSong" class="hover:text-blue-400 transition">
                  <font-awesome-icon icon="step-forward" />
                </button>
                <button @click="toggleRepeat" :class="{ 'text-blue-500': isRepeatMode }"
                  class="hover:text-blue-400 transition">
                  <font-awesome-icon icon="redo" />
                </button>
              </div>

              <div class="flex items-center gap-2">
                <button class="hover:text-blue-400 transition">
                  <font-awesome-icon v-if="volume == 0" icon="volume-mute" class="text-xl" />
                  <font-awesome-icon v-else icon="volume-up" class="text-xl" />
                </button>
                <input type="range" v-model="volume" min="0" max="1" step="0.1" class="w-16 accent-blue-600" />
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- Download Progress Modal -->
    <div v-if="showDownloadProgress"
      class="fixed inset-0 bg-black/80 backdrop-blur-sm flex items-center justify-center z-50">
      <div class="bg-gray-800 p-6 rounded-lg max-w-md w-full mx-4 border border-gray-700">
        <h3 class="text-lg font-semibold mb-4">
          {{ isDownloadingPlaylist ? '批量下载中' : '下载中' }}
        </h3>

        <!-- 当前歌曲进度 -->
        <div class="mb-4">
          <div class="text-sm mb-2 truncate">
            {{ currentDownloadSong?.song || '' }}
          </div>
          <div class="h-2 bg-gray-700 rounded-full overflow-hidden">
            <div class="h-full bg-purple-600 transition-all duration-300" :style="{ width: `${downloadProgress}%` }">
            </div>
          </div>
          <div class="text-right text-xs text-gray-400 mt-1">
            {{ downloadProgress.toFixed(1) }}%
          </div>
        </div>

        <!-- 播放列表总进度 -->
        <div v-if="isDownloadingPlaylist" class="mb-2">
          <div class="text-sm mb-2 flex justify-between">
            <span>总体进度</span>
            <span class="text-gray-400">{{ downloadedCount }}/{{ totalDownloads }} 首歌曲</span>
          </div>
          <div class="h-2 bg-gray-700 rounded-full overflow-hidden">
            <div class="h-full bg-green-600 transition-all duration-300" :style="{ width: `${playlistProgress}%` }">
            </div>
          </div>
          <div class="text-right text-xs text-gray-400 mt-1">
            {{ playlistProgress.toFixed(1) }}%
          </div>
        </div>
      </div>
    </div>


  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted, watch } from 'vue';

const QQList = ref(false)

QQList.value = import.meta.env.VITE_APP_QQList === 'true' || false;

const title = import.meta.env.VITE_APP_TITLE || '音乐播放';
const music_api = import.meta.env.VITE_APP_MUSIC_API || '';
const music_lyric_api = import.meta.env.VITE_APP_MUSIC_LYRIC_API || '';
const music_list_api = import.meta.env.VITE_APP_MUSIC_LIST_API || '';
const icon = import.meta.env.VITE_APP_MUSIC_ICON_LINK || '/vite.svg';


const showPlaylist = ref(false);
const isSearchFocused = ref(false);
const isDarkMode = ref(true);


// State
const searchQuery = ref('');
const searchResults = ref([]);
const playlist = ref([]);
const currentSong = ref(null);
const isPlaying = ref(false);

const musicName = ref('');
const audio = ref(null);
const volume = ref(1);
const currentTime = ref(0);
const duration = ref(0);
const isShuffleMode = ref(false);
const isRepeatMode = ref(false);
const showToast = ref(false);
const isToastFading = ref(false);
const toastMessage = ref('');

// 控制详情页显示
const isDetail = ref(false)

// 当前播放的歌词索引
const currentLyricIndex = ref(0)
const lyricsContainer = ref(null)

// 下载相关的状态
const showDownloadProgress = ref(false);
const downloadProgress = ref(0);
const isDownloadingPlaylist = ref(false);
const currentDownloadSong = ref(null);
const downloadedCount = ref(0);
const totalDownloads = ref(0);
const playlistProgress = ref(0);
const audioQualities = [
  { label: '试听', value: 1 },
  { label: '有损', value: 3 },
  { label: '标准', value: 7 },
  { label: 'HQ高音质', value: 9 },
  { label: 'SQ无损音质', value: 13 },
  { label: 'Hi-Res音质', value: 14 }
];

// 用来存储选择的音质
const selectedQuality = ref(audioQualities[5].value);  // 默认选择 Hi-Res 音质

// 音源选项
const musicSources = [
  { label: 'QQ音乐', value: 'qqdg/?word' },
  { label: '网易云音乐', value: 'wydg/?msg' }
];

// 默认选择 QQ音乐
const selectedSource = ref(musicSources[0].value);

watch(selectedSource, (newSource) => {

  searchResults.value = []; // 注意这里使用 .value
  if(QQList.value){
    handleRankSelect(selectedRank.value)
  }
  else{
    searchResults.value =[]
  }
  

  showToastMessage(`已切换到${newSource.includes('qqdg') ? 'QQ音乐' : '网易云音乐'}`);
});

// 排行榜选项
const rankOptions = [
  { value: 'soaring', label: '飙升榜' },
  { value: 'hot', label: '热歌榜' },
  { value: 'new', label: '新歌榜' },
  { value: 'popular', label: '流行榜' }
]

// 默认选中飙升榜
const selectedRank = ref('soaring')

// 定义各榜单的处理函数
const soaring = async () => {
  try {
    const response = await fetch(`${music_list_api}/fetch_music_soaring`);
    const data = await response.json();
    searchResults.value = data.data || [];

  } catch (error) {
    console.error('加载每日热曲失败:', error);
    // showToastMessage('搜索音乐失败');
  }
}

// 每日热曲
const hot = async () => {
  try {
    const response = await fetch(`${music_list_api}/fetch_music_hot`);
    const data = await response.json();
    searchResults.value = data.data || [];

  } catch (error) {
    console.error('加载每日热曲失败:', error);
    // showToastMessage('搜索音乐失败');
  }
};

const newSongs = async () => {
  try {
    const response = await fetch(`${music_list_api}/fetch_music_newSongs`);
    const data = await response.json();
    searchResults.value = data.data || [];

  } catch (error) {
    console.error('加载每日热曲失败:', error);
    // showToastMessage('搜索音乐失败');
  }
}

const popular = async () => {
  try {
    const response = await fetch(`${music_list_api}/fetch_music_popular`);
    const data = await response.json();
    searchResults.value = data.data || [];

  } catch (error) {
    console.error('加载每日热曲失败:', error);
    // showToastMessage('搜索音乐失败');
  }
}

// 创建一个映射函数，根据选中的榜单调用对应的函数
const handleRankSelect = (rank) => {
  selectedRank.value = rank

  switch (rank) {
    case 'soaring':
      soaring()
      break
    case 'hot':
      hot()
      break
    case 'new':
      newSongs()
      break
    case 'popular':
      popular()
      break
  }
}





const ensureHttps = (url) => {
  if (!url) return url;
  return url.replace(/^http:/, 'https:');
};

// 监听音乐播放时间，更新歌词
watch(() => currentTime.value, (newTime) => {
  if (!currentSong.value?.lyrics) return

  const timeInMilliseconds = newTime * 1000
  const lyrics = currentSong.value.lyrics

  // 找到当前时间对应的歌词
  for (let i = lyrics.length - 1; i >= 0; i--) {
    const lyricTime = timeStringToMilliseconds(lyrics[i].time)
    if (timeInMilliseconds >= lyricTime) {
      if (currentLyricIndex.value !== i) {
        currentLyricIndex.value = i
        scrollToCurrentLyric()
      }
      break
    }
  }
})

// 将歌词时间字符串转换为毫秒数
const timeStringToMilliseconds = (timeString) => {
  const [minutes, seconds] = timeString.split(':')
  const [secs, ms] = seconds.split('.')
  return (parseInt(minutes) * 60 + parseInt(secs)) * 1000 + parseInt(ms)
}

// 滚动到当前歌词
const scrollToCurrentLyric = () => {
  if (!lyricsContainer.value) return

  const container = lyricsContainer.value
  const lyrics = container.getElementsByTagName('p')
  const currentLyric = lyrics[currentLyricIndex.value]

  if (!currentLyric) return

  const containerHeight = container.clientHeight
  const lyricHeight = currentLyric.clientHeight
  const scrollTop = currentLyric.offsetTop - (containerHeight / 2) + (lyricHeight / 2)

  container.scrollTo({
    top: scrollTop,
    behavior: 'smooth'
  })
}

// 添加处理歌词点击并跳转到对应时间的方法
const handleLyricClick = (lyric) => {
  if (!audio.value.currentTime) return

  // 将歌词时间转换为秒
  const timeInSeconds = timeStringToSeconds(lyric.time)

  // 在音频中跳转到特定时间
  audio.value.currentTime = timeInSeconds
}

// 实用函数：将时间字符串转换为秒
const timeStringToSeconds = (timeString) => {
  const [minutes, seconds] = timeString.split(':')
  const [secs, ms] = seconds.split('.')
  return parseInt(minutes) * 60 + parseFloat(`${secs}.${ms}`)
}

// 创建通用的下载函数
const downloadFile = async (blob, filename) => {
  const url = window.URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  a.download = filename;
  document.body.appendChild(a);
  a.click();
  document.body.removeChild(a);
  window.URL.revokeObjectURL(url);
};

// 下载单首歌曲
const downloadSong = async (song) => {
  try {
    showDownloadProgress.value = true;
    currentDownloadSong.value = song;

    downloadProgress.value = 0;
    isDownloadingPlaylist.value = false;
    const link = song.muLink;
    const selectedQualityValue = selectedQuality.value;

    // 获取下载链接
    const response = await fetch(`${music_api}/API/${link}=${encodeURIComponent(song.muName)}&n=${encodeURIComponent(song.muId)}&q=${encodeURIComponent(selectedQualityValue)}&mid=${song.mid}`);
    if (!response.ok) {
      console.error(`获取下载链接失败: ${response.status}`);
    }

    const data = await response.json();
    if (!data.data?.url && !data?.mp3) {
      throw new Error('No download URL found');
    }
    let secureUrl;
    if (data.data?.url) {
      secureUrl = ensureHttps(data.data.url);
    }
    else {
      secureUrl = ensureHttps(data.mp3);
    }

    // 下载文件内容
    const downloadResponse = await fetch(secureUrl);
    if (!downloadResponse.ok) {
      throw new Error(`Download failed: ${downloadResponse.status}`);
    }

    // 获取总大小用于进度计算
    const totalSize = parseInt(downloadResponse.headers.get('content-length') || '0');
    const reader = downloadResponse.body.getReader();
    const chunks = [];
    let receivedSize = 0;

    // 读取流
    while (true) {
      const { done, value } = await reader.read();
      if (done) break;

      chunks.push(value);
      receivedSize += value.length;

      // 更新进度
      if (totalSize) {
        downloadProgress.value = (receivedSize / totalSize) * 100;
      }
    }

    // 合并数据并创建 blob
    const blob = new Blob(chunks, { type: 'audio/mpeg' });
    const filename = `${song.song} - ${song.singer}.mp3`.replace(/[<>:"/\\|?*]/g, '');

    // 使用传统下载方式
    await downloadFile(blob, filename);

    showToastMessage('下载完成');
    return true;
  } catch (error) {
    console.error('下载失败:', error);
    showToastMessage(`下载失败: ${error.message}`);
    return false;
  } finally {
    showDownloadProgress.value = false;
    downloadProgress.value = 0;
    currentDownloadSong.value = null;
  }
};

// 下载播放列表
const downloadPlaylist = async () => {
  if (playlist.value.length === 0) return;

  // 显示确认对话框
  if (!confirm(`确认下载 ${playlist.value.length} 首歌曲？下载将按顺序进行，请耐心等待。`)) {
    return;
  }

  try {
    showDownloadProgress.value = true;
    isDownloadingPlaylist.value = true;
    downloadedCount.value = 0;
    totalDownloads.value = playlist.value.length;
    let successCount = 0;

    for (const song of playlist.value) {
      try {
        currentDownloadSong.value = song;
        downloadProgress.value = 0;

        // 添加延时避免请求过于频繁
        await new Promise(resolve => setTimeout(resolve, 1500));

        const link = song.muLink;
        const selectedQualityValue = selectedQuality.value;

        // 获取下载链接
        const response = await fetch(`${music_api}/API/${link}=${encodeURIComponent(song.muName)}&n=${encodeURIComponent(song.muId)}&q=${encodeURIComponent(selectedQualityValue)}&mid=${song.mid}`);
        if (!response.ok) {
          console.error(`获取下载链接失败: ${response.status}`);
          continue;
        }

        const data = await response.json();
        if (!data.data?.url && !data?.mp3) {
          console.error('未找到下载链接');
          continue;
        }

        let secureUrl;
        if (data.data?.url) {
          secureUrl = ensureHttps(data.data.url);
        }
        else {
          secureUrl = ensureHttps(data.mp3);
        }

        // 下载文件内容
        const downloadResponse = await fetch(secureUrl);
        if (!downloadResponse.ok) {
          console.error(`下载歌曲失败: ${downloadResponse.status}`);
          continue;
        }

        // 获取总大小用于进度计算
        const totalSize = parseInt(downloadResponse.headers.get('content-length') || '0');
        const reader = downloadResponse.body.getReader();
        const chunks = [];
        let receivedSize = 0;

        // 读取流并更新进度
        while (true) {
          const { done, value } = await reader.read();
          if (done) break;

          chunks.push(value);
          receivedSize += value.length;

          // 更新当前歌曲的下载进度
          if (totalSize) {
            downloadProgress.value = (receivedSize / totalSize) * 100;
          }
        }

        // 合并数据并创建 blob
        const blob = new Blob(chunks, { type: 'audio/mpeg' });
        const filename = `${song.song} - ${song.singer}.mp3`.replace(/[<>:"/\\|?*]/g, '');

        // 使用传统下载方式
        await downloadFile(blob, filename);

        successCount++;
      } catch (error) {
        console.error(`下载歌曲 ${song.song} 失败:`, error);
      } finally {
        downloadedCount.value++;
        // 显示总体进度
        playlistProgress.value = (downloadedCount.value / totalDownloads.value) * 100;
      }
    }

    showToastMessage(`批量下载完成，成功下载 ${successCount} 首歌曲`);
  } catch (error) {
    console.error('批量下载失败:', error);
    showToastMessage(`批量下载失败: ${error.message}`);
  } finally {
    showDownloadProgress.value = false;
    isDownloadingPlaylist.value = false;
    downloadProgress.value = 0;
    playlistProgress.value = 0;
    currentDownloadSong.value = null;
    downloadedCount.value = 0;
    totalDownloads.value = 0;
  }
};

const togglePlaylistView = () => {
  showPlaylist.value = !showPlaylist.value;
};

const toggleTheme = () => {
  isDarkMode.value = !isDarkMode.value;
  localStorage.setItem('isDarkMode', isDarkMode.value);


};

// Audio Setup
onMounted(() => {

  document.title = title;

  document.querySelector("link[rel~='icon']").href = icon;

  audio.value = new Audio();
  audio.value.volume = volume.value;


  audio.value.addEventListener('timeupdate', () => {
    currentTime.value = audio.value.currentTime;
  });

  audio.value.addEventListener('loadedmetadata', () => {
    duration.value = audio.value.duration;
  });

  audio.value.addEventListener('ended', () => {
    if (isRepeatMode.value) {
      audio.value.currentTime = 0;
      audio.value.play();
    } else {
      nextSong();
    }
  });

  soaring()

  // 从本地存储加载播放列表
  const savedPlaylist = localStorage.getItem('playlist');
  if (savedPlaylist) {
    playlist.value = JSON.parse(savedPlaylist);
  }
  if (localStorage.getItem('isDarkMode')) {
    isDarkMode.value = localStorage.getItem('isDarkMode') === 'true'
  }
});


// Toast
const showToastMessage = (message) => {
  toastMessage.value = message;
  showToast.value = true;
  isToastFading.value = false;

  setTimeout(() => {
    isToastFading.value = true;
    setTimeout(() => {
      showToast.value = false;
    }, 300);
  }, 2000);
};

// Computed
const progress = computed(() => {
  return (currentTime.value / duration.value) * 100 || 0;
});

// Methods
const searchMusic = async () => {
  if (!searchQuery.value.trim()) {
    showToastMessage('请输入音乐名');
    return;
  }
  try {
    const selectedSourceValue = selectedSource.value;

    const response = await fetch(`${music_api}/API/${selectedSourceValue}=${encodeURIComponent(searchQuery.value)}`);
    const data = await response.json();
    searchResults.value = data.data || [];

    // 为每个元素添加muName属性
    searchResults.value.forEach((song, index) => {
      song.muName = searchQuery.value;  // 给每个对象添加muName属性
      song.muId = index + 1
      song.muLink = selectedSourceValue
    });

  } catch (error) {
    console.error('搜索音乐失败:', error);
    showToastMessage('搜索音乐失败');
  }
};

// 网易云api图片修复
const fixImgUrl = (arr) => {
  if (!arr || arr.length <= 1) return arr;
  const result = JSON.parse(JSON.stringify(arr));

  // 需要从最后一个开始往前处理，否则会覆盖掉我们需要的值
  for (let i = result.length - 1; i > 0; i--) {
    result[i].img = result[i - 1].img;
  }

  return result;
};

// 格式化搜索结果
const formattedSearchResults = computed(() => {
  if (!searchResults.value) return [];
  if (searchResults.value.length === 20) {
    return searchResults.value;
  }
  // 根据不同音源格式化数据
  else if (selectedSource.value.startsWith('qqdg')) {
    // QQ音乐数据格式化
    return searchResults.value.map(item => ({
      id: item.id,
      cover: item.cover,
      song: item.song,
      singer: item.singer,
      muName: item.muName,
      muId: item.muId,
      muLink: item.muLink,
      mid: item.mid,
      uname: 'QQ音乐',
    }));
  } else {
    // 网易云音乐数据格式化
    return fixImgUrl(searchResults.value).map(item => ({
      id: item.id,
      cover: item.img,
      song: item.name,
      singer: item.singer,
      muName: item.muName,
      muId: item.muId,
      muLink: item.muLink,
      mid: item.id,
      uname: '网易云',
    }));
  }
});

const playSong = async (song) => {
  try {
    // 获取用户选择的音质
    const selectedQualityValue = selectedQuality.value;
    const response = await fetch(`${music_api}/API/${song.muLink}=${encodeURIComponent(song.muName)}&n=${encodeURIComponent(song.muId)}&q=${encodeURIComponent(selectedQualityValue)}&mid=${song.mid}`);
    const data = await response.json();
    let lyricsData = ''
    try {
      const response = await fetch(`${music_lyric_api}/get_lyrics?mid=${encodeURIComponent(song.mid)}`);
      if (response.ok) {
        lyricsData = await response.json();
      } else {
        lyricsData = {
          'lyric': [{
            "name": "请求歌词数据失败",
            "time": "00:00.000"
          }]
        }
        console.error('查询失败:', response.status);
      }
    } catch (error) {
      lyricsData = {
        'lyric': [{
          "name": "请求歌词数据失败",
          "time": "00:00.000"
        }]
      }
      console.error('请求失败:', error);
    }
    
    if (data.data?.url) {
      song.lyrics = lyricsData?.lyric || [{
        "name": "暂无歌词",
        "time": "00:00.000"
      }];
      currentSong.value = song;
      audio.value.src = data.data.url.replace(/^http:/, 'https:');


      await audio.value.play();
      isPlaying.value = true;
      musicName.value = song.song;

    }

    else if (data?.mp3) {
      song.lyrics = data?.lyric || [{
        "name": "暂无歌词",
        "time": "00:00.000"
      }];
      currentSong.value = song;


      audio.value.src = data.mp3.replace(/^http:/, 'https:');


      await audio.value.play();
      isPlaying.value = true;
      musicName.value = song.song;

    }

    else {
      showToastMessage('播放链接为空，请尝试刷新页面，或重新进行搜索');
    }

  } catch (error) {
    console.error('播放音乐失败:', error);
    showToastMessage('播放音乐失败');
  }
};

const addToPlaylist = (song) => {
  if (playlist.value.some(item => item.mid === song.mid)) {
    showToastMessage('该歌曲已在播放列表中');
    return;
  }

  playlist.value.push(song);
  savePlaylist();
  showToastMessage('已添加到播放列表');
};

const removeFromPlaylist = (index) => {
  playlist.value.splice(index, 1);
  savePlaylist();
  showToastMessage('已从播放列表中移除');
};

const clearPlaylist = () => {
  playlist.value = [];
  savePlaylist();
  showToastMessage('播放列表已清空');
};

const savePlaylist = () => {
  localStorage.setItem('playlist', JSON.stringify(playlist.value));
};

const togglePlay = () => {
  if (!currentSong.value) return;

  if (isPlaying.value) {
    audio.value.pause();
  } else {
    audio.value.play();
  }
  isPlaying.value = !isPlaying.value;
};

const toggleDetail = () => {
  isDetail.value = !isDetail.value
}

const nextSong = () => {
  if (playlist.value.length === 0) return;

  const currentIndex = playlist.value.findIndex(song => song.id === currentSong.value?.id);
  let nextIndex;

  if (isShuffleMode.value && playlist.value.length > 1) {
    let newIndex;
    do {
      newIndex = Math.floor(Math.random() * playlist.value.length);
    } while (newIndex === currentIndex);
    nextIndex = newIndex;
  } else {
    nextIndex = (currentIndex + 1) % playlist.value.length;
  }

  playSong(playlist.value[nextIndex]);
};

const previousSong = () => {
  if (playlist.value.length === 0) return;

  const currentIndex = playlist.value.findIndex(song => song.id === currentSong.value?.id);
  const previousIndex = currentIndex > 0 ? currentIndex - 1 : playlist.value.length - 1;

  playSong(playlist.value[previousIndex]);
};

const toggleShuffle = () => {
  isShuffleMode.value = !isShuffleMode.value;
  showToastMessage(isShuffleMode.value ? '已开启随机播放' : '已关闭随机播放');
};

const toggleRepeat = () => {
  isRepeatMode.value = !isRepeatMode.value;
  showToastMessage(isRepeatMode.value ? '已开启单曲循环' : '已关闭单曲循环');
};

const seekTo = () => {
  audio.value.currentTime = currentTime.value;
};

const formatTime = (time) => {
  const minutes = Math.floor(time / 60);
  const seconds = Math.floor(time % 60);
  return `${minutes}:${seconds.toString().padStart(2, '0')}`;
};

// Watch volume changes
watch(volume, (newVolume) => {
  if (audio.value) {
    audio.value.volume = newVolume;
  }
});


</script>

<style>
/* 可选：添加一些全局样式 */
::-webkit-scrollbar {
  width: 8px;
  height: 8px;
}

::-webkit-scrollbar-track {
  background: rgba(31, 41, 55, 0.5);
}

::-webkit-scrollbar-thumb {
  background: rgba(59, 130, 246, 0.5);
  border-radius: 4px;
}

::-webkit-scrollbar-thumb:hover {
  background: rgba(59, 130, 246, 0.7);
}

/* 让范围输入控件更美观 */
input[type="range"] {
  -webkit-appearance: none;
  background: transparent;
}

input[type="range"]::-webkit-slider-thumb {
  -webkit-appearance: none;
  height: 16px;
  width: 16px;
  border-radius: 50%;
  background: #3b82f6;
  cursor: pointer;
  margin-top: -4px;
}

input[type="range"]::-webkit-slider-runnable-track {
  width: 100%;
  height: 8px;
  background: rgba(31, 41, 55, 0.5);
  border-radius: 4px;
}

.animate-spin-slow {
  animation: spin 20s linear infinite;
}

@keyframes spin {
  from {
    transform: rotate(0deg);
  }

  to {
    transform: rotate(360deg);
  }
}

.hide-scrollbar {
  scrollbar-width: none;
  -ms-overflow-style: none;
}

.hide-scrollbar::-webkit-scrollbar {
  display: none;
}
</style>

