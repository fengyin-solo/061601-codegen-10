<script setup lang="ts">
import { ref, computed } from 'vue'
import { useGameStore } from '../stores/gameStore'
import gameConfig from '../config/gameConfig'
import { getTimeLabel } from '../utils/gameUtils'

const emit = defineEmits<{
  (e: 'close'): void
}>()

const gameStore = useGameStore()
const activeTab = ref<'preview' | 'review'>('preview')

gameStore.clearNewClueCount()

const previewClues = computed(() => gameStore.activeClues)

const reviewClues = computed(() => gameStore.dismissedClues)

const allClues = computed(() => {
  if (activeTab.value === 'preview') return previewClues.value
  return reviewClues.value
})

function getCharacterName(characterId?: string): string {
  if (!characterId) return ''
  const char = gameConfig.characters.find(c => c.id === characterId)
  return char?.name || ''
}

function getCharacterAvatar(characterId?: string): string {
  if (!characterId) return ''
  const char = gameConfig.characters.find(c => c.id === characterId)
  return char?.avatar || ''
}

function getProgressPercent(clueId: string): number {
  const progress = gameStore.getClueProgress(clueId)
  return Math.round(progress.current / progress.target * 100)
}

function getProgressLabel(clueId: string): string {
  const progress = gameStore.getClueProgress(clueId)
  return progress.label
}

function handleDismiss(clueId: string) {
  gameStore.dismissClue(clueId)
}

function handleRestore(clueId: string) {
  const clue = gameStore.collectedClues.find(c => c.id === clueId)
  if (clue) {
    clue.dismissed = false
  }
}
</script>

<template>
  <Teleport to="body">
    <div class="modal-overlay" @click.self="emit('close')">
      <div class="modal-content clue-panel">
        <div class="panel-header">
          <h2 class="panel-title">
            <span class="title-icon">🔮</span>
            事件预告
          </h2>
          <button class="close-btn" @click="emit('close')">✕</button>
        </div>

        <div class="tab-bar">
          <button
            class="tab-btn"
            :class="{ active: activeTab === 'preview' }"
            @click="activeTab = 'preview'"
          >
            🔔 预告
            <span v-if="previewClues.length" class="tab-badge">{{ previewClues.length }}</span>
          </button>
          <button
            class="tab-btn"
            :class="{ active: activeTab === 'review' }"
            @click="activeTab = 'review'"
          >
            📜 线索回看
            <span v-if="reviewClues.length" class="tab-badge secondary">{{ reviewClues.length }}</span>
          </button>
        </div>

        <div class="clue-list scrollbar-thin">
          <div
            v-for="clue in allClues"
            :key="clue.id"
            class="clue-card"
            :class="{ 'clue-active': !clue.dismissed, 'clue-dismissed': clue.dismissed }"
          >
            <div class="clue-top">
              <div class="clue-character" v-if="clue.characterId">
                <span class="char-emoji">{{ getCharacterAvatar(clue.characterId) }}</span>
                <span class="char-label">{{ getCharacterName(clue.characterId) }}</span>
              </div>
              <span class="clue-event-tag">🔮 {{ clue.eventTitle }}</span>
            </div>

            <h3 class="clue-title">{{ clue.title }}</h3>
            <p class="clue-hint">{{ clue.hint }}</p>

            <div v-if="!clue.dismissed" class="clue-progress">
              <div class="progress-bar">
                <div
                  class="progress-fill clue-progress-fill"
                  :style="{ width: getProgressPercent(clue.id) + '%' }"
                ></div>
              </div>
              <span class="progress-label">{{ getProgressLabel(clue.id) }}</span>
            </div>

            <div class="clue-footer">
              <span class="clue-time">第{{ clue.revealedDay }}天 {{ getTimeLabel(clue.revealedTime) }}</span>
              <button
                v-if="!clue.dismissed"
                class="action-btn dismiss-btn"
                @click="handleDismiss(clue.id)"
              >
                已知晓
              </button>
              <button
                v-else
                class="action-btn restore-btn"
                @click="handleRestore(clue.id)"
              >
                恢复预告
              </button>
            </div>
          </div>

          <div v-if="allClues.length === 0" class="empty-state">
            <span class="empty-icon">{{ activeTab === 'preview' ? '🔕' : '📭' }}</span>
            <p>{{ activeTab === 'preview' ? '暂无预告线索，继续探索吧' : '暂无已知晓的线索' }}</p>
          </div>
        </div>
      </div>
    </div>
  </Teleport>
</template>

<style scoped>
.clue-panel {
  padding: 24px;
  max-width: 560px;
  max-height: 80vh;
  display: flex;
  flex-direction: column;
}

.panel-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 16px;
}

.panel-title {
  font-size: 20px;
  font-weight: 700;
  display: flex;
  align-items: center;
  gap: 8px;
}

.title-icon {
  font-size: 24px;
}

.close-btn {
  width: 32px;
  height: 32px;
  border-radius: 50%;
  background: var(--bg-tertiary);
  font-size: 14px;
  display: flex;
  align-items: center;
  justify-content: center;
  color: var(--text-secondary);
}

.close-btn:hover {
  background: var(--accent-light);
  color: var(--accent-primary);
}

.tab-bar {
  display: flex;
  gap: 8px;
  margin-bottom: 16px;
  background: var(--bg-tertiary);
  border-radius: var(--radius-md);
  padding: 4px;
}

.tab-btn {
  flex: 1;
  padding: 10px;
  border-radius: var(--radius-sm);
  font-size: 14px;
  font-weight: 500;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 6px;
  color: var(--text-secondary);
}

.tab-btn.active {
  background: var(--bg-secondary);
  color: var(--text-primary);
  box-shadow: 0 1px 4px var(--shadow-color);
}

.tab-badge {
  font-size: 11px;
  padding: 1px 6px;
  border-radius: 9999px;
  background: var(--accent-primary);
  color: white;
  min-width: 18px;
  text-align: center;
}

.tab-badge.secondary {
  background: var(--text-muted);
}

.clue-list {
  flex: 1;
  overflow-y: auto;
  display: flex;
  flex-direction: column;
  gap: 12px;
  padding-right: 4px;
}

.clue-card {
  padding: 16px;
  border-radius: var(--radius-md);
  border: 2px solid transparent;
  transition: all 0.2s;
}

.clue-active {
  background: #faf5ff;
  border-color: #d8b4fe;
}

[data-theme='dark'] .clue-active {
  background: #3b0764;
  border-color: #7c3aed;
}

.clue-dismissed {
  background: var(--bg-tertiary);
  opacity: 0.7;
}

.clue-top {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 10px;
}

.clue-character {
  display: flex;
  align-items: center;
  gap: 6px;
}

.char-emoji {
  font-size: 18px;
}

.char-label {
  font-size: 13px;
  font-weight: 500;
  color: var(--accent-primary);
}

.clue-event-tag {
  font-size: 11px;
  padding: 3px 8px;
  background: var(--accent-light);
  color: var(--accent-primary);
  border-radius: 9999px;
}

.clue-title {
  font-size: 15px;
  font-weight: 600;
  margin-bottom: 6px;
  color: var(--text-primary);
}

.clue-hint {
  font-size: 13px;
  line-height: 1.6;
  color: var(--text-secondary);
  margin-bottom: 12px;
}

.clue-progress {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-bottom: 12px;
}

.clue-progress .progress-bar {
  flex: 1;
  height: 6px;
  background: var(--bg-secondary);
  border-radius: 3px;
  overflow: hidden;
}

.clue-progress-fill {
  height: 100%;
  border-radius: 3px;
  background: linear-gradient(90deg, #a855f7, #ec4899);
  transition: width 0.3s;
}

.progress-label {
  font-size: 11px;
  color: var(--text-muted);
  white-space: nowrap;
}

.clue-footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.clue-time {
  font-size: 11px;
  color: var(--text-muted);
}

.action-btn {
  font-size: 12px;
  padding: 4px 12px;
  border-radius: var(--radius-sm);
  font-weight: 500;
}

.dismiss-btn {
  background: var(--bg-secondary);
  color: var(--text-secondary);
}

.dismiss-btn:hover {
  background: var(--accent-light);
  color: var(--accent-primary);
}

.restore-btn {
  background: var(--accent-light);
  color: var(--accent-primary);
}

.restore-btn:hover {
  background: var(--accent-primary);
  color: white;
}

.empty-state {
  text-align: center;
  padding: 40px 0;
  color: var(--text-muted);
}

.empty-icon {
  font-size: 40px;
  display: block;
  margin-bottom: 12px;
}

.empty-state p {
  font-size: 14px;
}
</style>
