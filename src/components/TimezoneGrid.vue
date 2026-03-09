<template>
  <div class="timezone-controls">
    <el-switch
      v-model="showDate"
      active-text="显示日期"
      inline-prompt
      inactive-text="隐藏日期"
      style="margin-right: 20px;"
    />
    <el-switch
      v-model="showSeconds"
      inline-prompt
      active-text="显示秒"
      inactive-text="隐藏秒"
      style="margin-right: 20px;"
    />
    <el-switch
      v-model="showUtc"
      inline-prompt
      active-text="显示 UTC"
      inactive-text="隐藏 UTC"
    />
  </div>
  <div class="timezone-grid">
    <div 
      v-for="timezone in timezones" 
      :key="timezone.id"
      class="timezone-item"
    >
      <div v-if="showDate" class="timezone-date">{{ timezone.currentDate }}</div>
      <div class="timezone-time">{{ timezone.currentTime }}</div>
      <div class="timezone-country">
        <div class="country-short">{{ timezone.countryShort }}</div>
        <div class="country-full">{{ timezone.countryFull }}</div>
      </div>
      <div class="timezone-identifiers">
        <div 
          class="identifier-item"
          @click="copyToClipboard(timezone.identifier)"
        >
          {{ timezone.identifier }}
          <el-tooltip :content="copiedId === timezone.identifier ? '已复制' : '点击复制'" placement="top">
            <span class="copy-icon">📋</span>
          </el-tooltip>
        </div>

      </div>
      <div v-if="showUtc" class="timezone-identifiers">
              <div 
          class="identifier-item"
          @click="copyToClipboard(timezone.utcOffset)"
        >
          {{ timezone.utcOffset }}
          <el-tooltip :content="copiedOffset === timezone.utcOffset ? '已复制' : '点击复制'" placement="top">
            <span class="copy-icon">📋</span>
          </el-tooltip>
        </div>
    </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, watch } from 'vue'

const copiedId = ref('')
const copiedOffset = ref('')
const showDate = ref(true)
const showSeconds = ref(false)
const showUtc = ref(false)

// 原始时区数据
const rawTimezones = [
  { identifier: 'Asia/Shanghai', countryShort: 'CN', countryFull: '中国', utcOffset: 'UTC+08:00' },
  { identifier: 'Pacific/Honolulu', countryShort: 'US', countryFull: '美国-夏威夷', utcOffset: 'UTC-10:00' },
  { identifier: 'America/Adak', countryShort: 'US', countryFull: '美国-埃达克', utcOffset: 'UTC-09:00' },
  { identifier: 'America/Juneau', countryShort: 'US', countryFull: '美国-阿拉斯加', utcOffset: 'UTC-08:00' },
  { identifier: 'America/Los_Angeles', countryShort: 'US', countryFull: '美国-洛杉矶', utcOffset: 'UTC-07:00' },
  { identifier: 'America/Mexico_City', countryShort: 'MX', countryFull: '墨西哥', utcOffset: 'UTC-05:00' },
  { identifier: 'America/Chicago', countryShort: 'US', countryFull: '美国-芝加哥', utcOffset: 'UTC-06:00' },
  { identifier: 'America/Toronto', countryShort: 'CA', countryFull: '加拿大', utcOffset: 'UTC-04:00' },
  { identifier: 'America/Sao_Paulo', countryShort: 'BR', countryFull: '巴西', utcOffset: 'UTC-02:00' },
  { identifier: 'America/Nuuk', countryShort: 'DK', countryFull: '丹麦-格陵兰', utcOffset: 'UTC-02:00' },
  { identifier: 'Atlantic/Cape_Verde', countryShort: 'CV', countryFull: '佛得角', utcOffset: 'UTC-01:00' },
  { identifier: 'Europe/London', countryShort: 'GB', countryFull: '英国', utcOffset: 'UTC+00:00' },
  { identifier: 'Europe/Berlin', countryShort: 'DE', countryFull: '德国', utcOffset: 'UTC+01:00' },
  { identifier: 'Africa/Johannesburg', countryShort: 'ZA', countryFull: '南非', utcOffset: 'UTC+02:00' },
  { identifier: 'Europe/Moscow', countryShort: 'RU', countryFull: '俄罗斯', utcOffset: 'UTC+03:00' },
  { identifier: 'Asia/Dubai', countryShort: 'AE', countryFull: '阿联酋', utcOffset: 'UTC+04:00' },
  { identifier: 'Asia/Karachi', countryShort: 'PK', countryFull: '巴基斯坦', utcOffset: 'UTC+05:00' },
  { identifier: 'Asia/Dhaka', countryShort: 'BD', countryFull: '孟加拉国', utcOffset: 'UTC+06:00' },
  { identifier: 'Asia/Bangkok', countryShort: 'TH', countryFull: '泰国', utcOffset: 'UTC+07:00' },
  { identifier: 'Asia/Hong_Kong', countryShort: 'CN', countryFull: '中国香港', utcOffset: 'UTC+08:00' },
  { identifier: 'Asia/Tokyo', countryShort: 'JP', countryFull: '日本', utcOffset: 'UTC+09:00' },
  { identifier: 'Pacific/Port_Moresby', countryShort: 'PG', countryFull: '巴布亚新几内亚', utcOffset: 'UTC+10:00' },
  { identifier: 'Australia/Sydney', countryShort: 'AU', countryFull: '澳大利亚', utcOffset: 'UTC+11:00' },
  { identifier: 'Pacific/Nauru', countryShort: 'NR', countryFull: '瑙鲁', utcOffset: 'UTC+12:00' },
  { identifier: 'Pacific/Auckland', countryShort: 'NZ', countryFull: '新西兰', utcOffset: 'UTC+12:00' },
]

const timezones = ref(rawTimezones.map((tz, index) => ({
  ...tz,
  id: index + 1
})))

const updateTimes = () => {
  timezones.value.forEach(timezone => {
    const date = new Date()
    
    // 日期选项
    const dateOptions = {
      year: 'numeric',
      month: '2-digit',
      day: '2-digit',
      timeZone: timezone.identifier
    }
    timezone.currentDate = date.toLocaleDateString('zh-CN', dateOptions)
    
    // 时间选项
    const timeOptions = {
      hour: '2-digit',
      minute: '2-digit',
      hour12: false,
      timeZone: timezone.identifier
    }
    
    // 根据showSeconds决定是否显示秒
    if (showSeconds.value) {
      timeOptions.second = '2-digit'
    }
    
    timezone.currentTime = date.toLocaleTimeString('zh-CN', timeOptions)
  })
}

let intervalId = null

onMounted(() => {
  updateTimes()
  intervalId = setInterval(updateTimes, 1000)
})

onUnmounted(() => {
  if (intervalId) {
    clearInterval(intervalId)
  }
})

// 监听showSeconds变化，立即更新时间显示
watch(showSeconds, () => {
  updateTimes()
})

// 监听showDate变化，立即更新时间显示
watch(showDate, () => {
  updateTimes()
})

const copyToClipboard = (text) => {
  navigator.clipboard.writeText(text).then(() => {
    if (text.includes('/')) {
      copiedId.value = text
      setTimeout(() => {
        copiedId.value = ''
      }, 1500)
    } else {
      copiedOffset.value = text
      setTimeout(() => {
        copiedOffset.value = ''
      }, 1500)
    }
  })
}
</script>

<style scoped>
.timezone-controls {
  display: flex;
  justify-content: flex-end;
  padding: 20px;
  max-width: 1400px;
  margin: 0 auto;
}

.timezone-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 20px;
  padding: 0 20px 20px;
  max-width: 1400px;
  margin: 0 auto;
}

.timezone-item {
  background: #ffffff;
  border-radius: 12px;
  padding: 24px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.08);
  transition: all 0.3s ease;
  border: 1px solid #f0f0f0;
}

.timezone-item:hover {
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.12);
  transform: translateY(-4px);
}

.timezone-date {
  font-size: 14px;
  color: #909399;
  text-align: center;
  margin-bottom: 8px;
}

.timezone-time {
  font-size: 28px;
  font-weight: 700;
  color: #303133;
  margin-bottom: 16px;
  text-align: center;
}

.timezone-country {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 12px;
  margin-bottom: 16px;
}

.country-short {
  font-size: 16px;
  font-weight: 600;
  color: #409eff;
}

.country-full {
  font-size: 14px;
  color: #606266;
}

.timezone-identifiers {
  display: flex;
  justify-content: space-between;
  gap: 12px;
  margin-top: 16px;
}

.identifier-item {
  flex: 1;
  background: #f5f7fa;
  padding: 12px;
  border-radius: 8px;
  font-size: 14px;
  color: #606266;
  display: flex;
  justify-content: space-between;
  align-items: center;
  cursor: pointer;
  transition: all 0.2s ease;
}

.identifier-item:hover {
  background: #ecf5ff;
  color: #409eff;
}

.copy-icon {
  font-size: 16px;
  margin-left: 8px;
  opacity: 0.6;
  transition: opacity 0.2s ease;
}

.identifier-item:hover .copy-icon {
  opacity: 1;
}

@media (max-width: 768px) {
  .timezone-controls {
    padding: 12px;
  }
  
  .timezone-grid {
    grid-template-columns: repeat(auto-fill, minmax(240px, 1fr));
    padding: 0 12px 12px;
  }
  
  .timezone-item {
    padding: 20px;
  }
  
  .timezone-time {
    font-size: 24px;
  }
}

@media (max-width: 480px) {
  .timezone-grid {
    grid-template-columns: 1fr;
  }
  
  .timezone-identifiers {
    flex-direction: column;
  }
  
  .identifier-item {
    width: 100%;
  }
}
</style>