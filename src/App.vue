<script setup>
import TimezoneGrid from './components/TimezoneGrid.vue'
</script>

<template>
  <div class="app-container">
    <header class="app-header">
      <h1>全球时区展示</h1>
      <p>实时显示全球主要时区的当前时间</p>
    </header>
    <main>
      <TimezoneGrid />
    </main>
  </div>
</template>

<style scoped>
.app-container {
  min-height: 100vh;
  background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
}

.app-header {
  text-align: center;
  padding: 40px 20px;
  background: rgba(255, 255, 255, 0.8);
  backdrop-filter: blur(10px);
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.1);
}

.app-header h1 {
  font-size: 32px;
  font-weight: 700;
  color: #303133;
  margin-bottom: 8px;
}

.app-header p {
  font-size: 16px;
  color: #606266;
  margin: 0;
}

main {
  flex: 1;
}

@media (max-width: 768px) {
  .app-header {
    padding: 30px 16px;
  }
  
  .app-header h1 {
    font-size: 24px;
  }
  
  .app-header p {
    font-size: 14px;
  }
}
</style>
