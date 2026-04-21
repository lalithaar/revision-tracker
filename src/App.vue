<template>
  <div class="app">

    <!-- HOME -->
    <template v-if="!currentSubject">
      <div class="header">
        <span class="title">revision</span>
        <button class="action-btn" @click="openAddSubject">+ subject</button>
      </div>

      <div class="subject-list">
        <div v-if="!data.subjects.length" class="empty">no subjects yet</div>

        <div
          v-for="subject in data.subjects"
          :key="subject.id"
          class="subject-row"
          @click="openSubject(subject)"
        >
          <span class="subject-name">{{ subject.name }}</span>
          <div class="strip-wrap">
            <div v-if="!subject.topics.length" class="strip-empty" />
            <div
              v-for="topic in subject.topics"
              :key="topic.id"
              class="strip-seg"
              :style="{ background: STAGES[topic.stage] }"
            />
          </div>
          <div class="row-right">
            <button class="del-btn" @click.stop="deleteSubject(subject.id)">×</button>
            <span class="chevron">›</span>
          </div>
        </div>
      </div>

      <!-- add subject modal -->
      <div v-if="showAddSubject" class="modal-overlay" @click.self="showAddSubject = false">
        <div class="modal">
          <div class="modal-label">new subject</div>
          <input
            ref="subjectInput"
            v-model="newSubjectName"
            class="modal-input"
            placeholder="e.g. biochemistry"
            @keydown.enter="confirmAddSubject"
            @keydown.esc="showAddSubject = false"
          />
          <div class="modal-actions">
            <button @click="showAddSubject = false">cancel</button>
            <button class="primary" @click="confirmAddSubject">add</button>
          </div>
        </div>
      </div>
    </template>

    <!-- SUBJECT VIEW -->
    <template v-else>
      <div class="header">
        <button class="back-btn" @click="currentSubject = null">‹</button>
        <input
          v-model="currentSubject.name"
          class="subj-name-input"
          placeholder="subject name"
          @blur="save"
        />
        <button class="action-btn" @click="openAddTopic">+ topic</button>
      </div>

      <div class="topic-list">
        <div v-if="!currentSubject.topics.length" class="empty">no topics yet</div>

        <div
          v-for="topic in currentSubject.topics"
          :key="topic.id"
          class="topic-row"
        >
          <span class="topic-name">{{ topic.name }}</span>
          <div class="stage-dots">
            <div
              v-for="(color, i) in STAGES"
              :key="i"
              class="stage-dot"
              :class="{ filled: i <= topic.stage }"
              :style="i <= topic.stage ? { background: color, borderColor: color } : {}"
              :title="STAGE_LABELS[i]"
              @click="setStage(topic, i)"
            />
          </div>
          <button class="del-btn" @click="deleteTopic(topic.id)">×</button>
        </div>
      </div>

      <div class="legend">
        <template v-for="(color, i) in STAGES" :key="i">
          <div class="legend-swatch" :style="{ background: color }" />
          <span class="legend-text" :style="i < STAGES.length - 1 ? { marginRight: '10px' } : {}">
            {{ STAGE_LABELS[i] }}
          </span>
        </template>
      </div>

      <!-- add topic modal -->
      <div v-if="showAddTopic" class="modal-overlay" @click.self="showAddTopic = false">
        <div class="modal">
          <div class="modal-label">new topic</div>
          <input
            ref="topicInput"
            v-model="newTopicName"
            class="modal-input"
            placeholder="e.g. glycolysis"
            @keydown.enter="confirmAddTopic"
            @keydown.esc="showAddTopic = false"
          />
          <div class="modal-actions">
            <button @click="showAddTopic = false">cancel</button>
            <button class="primary" @click="confirmAddTopic">add</button>
          </div>
        </div>
      </div>
    </template>

  </div>
</template>

<script setup>
import { ref, nextTick, onMounted } from 'vue'

const STAGES = ['#E24B4A', '#EF9F27', '#FAC775', '#97C459', '#639922', '#27500A']
const STAGE_LABELS = ['not started', 'seen it', 'getting it', 'mostly know', 'confident', 'done']

const data = ref({ subjects: [] })
const currentSubject = ref(null)

const showAddSubject = ref(false)
const newSubjectName = ref('')
const subjectInput = ref(null)

const showAddTopic = ref(false)
const newTopicName = ref('')
const topicInput = ref(null)

function load() {
  const raw = localStorage.getItem('rv2')
  if (raw) data.value = JSON.parse(raw)
}

function save() {
  localStorage.setItem('rv2', JSON.stringify(data.value))
}

onMounted(load)

function uid() { return Math.random().toString(36).slice(2, 9) }

function openAddSubject() {
  newSubjectName.value = ''
  showAddSubject.value = true
  nextTick(() => subjectInput.value?.focus())
}

function confirmAddSubject() {
  const name = newSubjectName.value.trim()
  if (!name) return
  data.value.subjects.push({ id: uid(), name, topics: [] })
  save()
  showAddSubject.value = false
}

function deleteSubject(id) {
  data.value.subjects = data.value.subjects.filter(s => s.id !== id)
  save()
}

function openSubject(subject) {
  currentSubject.value = subject
}

function openAddTopic() {
  newTopicName.value = ''
  showAddTopic.value = true
  nextTick(() => topicInput.value?.focus())
}

function confirmAddTopic() {
  const name = newTopicName.value.trim()
  if (!name) return
  currentSubject.value.topics.push({ id: uid(), name, stage: 0 })
  save()
  showAddTopic.value = false
}

function deleteTopic(id) {
  currentSubject.value.topics = currentSubject.value.topics.filter(t => t.id !== id)
  save()
}

function setStage(topic, stage) {
  topic.stage = stage
  save()
}
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Geist+Mono:wght@300;400;500&family=Geist:wght@300;400;500&display=swap');

*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

:root {
  --bg: #ffffff;
  --bg2: #f5f5f5;
  --bg3: #eeeeee;
  --t1: #111111;
  --t2: #666666;
  --t3: #aaaaaa;
  --border: rgba(0,0,0,0.1);
  --border2: rgba(0,0,0,0.18);
  --sans: 'Geist', system-ui, sans-serif;
  --mono: 'Geist Mono', monospace;
  --r: 8px;
  --rl: 12px;
}

body {
  font-family: var(--sans);
  font-size: 14px;
  color: var(--t1);
  background: var(--bg);
  -webkit-font-smoothing: antialiased;
}

.app { min-height: 100vh; }

/* HEADER */
.header {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 20px 24px 16px;
  border-bottom: 0.5px solid var(--border);
}

.title {
  flex: 1;
  font-size: 12px;
  font-weight: 500;
  color: var(--t2);
  letter-spacing: 0.08em;
  text-transform: uppercase;
  font-family: var(--mono);
}

.action-btn {
  font-size: 13px;
  color: var(--t2);
  background: none;
  border: 0.5px solid var(--border2);
  border-radius: var(--r);
  padding: 5px 12px;
  cursor: pointer;
  font-family: var(--sans);
  transition: background 0.12s, color 0.12s;
}
.action-btn:hover { background: var(--bg2); color: var(--t1); }

.back-btn {
  background: none;
  border: none;
  color: var(--t2);
  font-size: 20px;
  line-height: 1;
  cursor: pointer;
  padding: 2px 6px;
  border-radius: var(--r);
  transition: color 0.12s;
}
.back-btn:hover { color: var(--t1); }

.subj-name-input {
  flex: 1;
  background: none;
  border: none;
  outline: none;
  font-size: 16px;
  font-weight: 500;
  color: var(--t1);
  font-family: var(--sans);
}
.subj-name-input::placeholder { color: var(--t3); }

/* SUBJECT LIST */
.subject-list { padding: 4px 0; }

.empty {
  padding: 48px 24px;
  text-align: center;
  color: var(--t3);
  font-size: 14px;
}

.subject-row {
  display: flex;
  align-items: center;
  padding: 0 24px;
  height: 56px;
  border-bottom: 0.5px solid var(--border);
  cursor: pointer;
  gap: 20px;
  transition: background 0.1s;
}
.subject-row:last-child { border-bottom: none; }
.subject-row:hover { background: var(--bg2); }
.subject-row:hover .del-btn { opacity: 1; }

.subject-name {
  font-size: 15px;
  font-weight: 400;
  color: var(--t1);
  min-width: 140px;
}

.strip-wrap {
  flex: 1;
  height: 6px;
  border-radius: 3px;
  overflow: hidden;
  display: flex;
  gap: 1px;
}

.strip-empty {
  flex: 1;
  height: 100%;
  background: var(--bg3);
  border-radius: 3px;
}

.strip-seg {
  flex: 1;
  height: 100%;
}

.row-right {
  display: flex;
  align-items: center;
  gap: 12px;
}

.chevron {
  color: var(--t3);
  font-size: 13px;
}

/* DEL BUTTON (shared) */
.del-btn {
  background: none;
  border: none;
  color: var(--t3);
  cursor: pointer;
  font-size: 15px;
  padding: 4px 6px;
  border-radius: 4px;
  opacity: 0;
  transition: opacity 0.12s, color 0.12s;
  line-height: 1;
}
.del-btn:hover { color: #E24B4A; }

/* TOPIC LIST */
.topic-list { padding: 4px 0; }

.topic-row {
  display: flex;
  align-items: center;
  padding: 0 24px;
  min-height: 48px;
  border-bottom: 0.5px solid var(--border);
  gap: 12px;
  transition: background 0.1s;
}
.topic-row:last-child { border-bottom: none; }
.topic-row:hover { background: var(--bg2); }
.topic-row:hover .del-btn { opacity: 1; }

.topic-name {
  flex: 1;
  font-size: 14px;
  color: var(--t1);
}

.stage-dots {
  display: flex;
  gap: 5px;
  align-items: center;
}

.stage-dot {
  width: 14px;
  height: 14px;
  border-radius: 50%;
  border: 1.5px solid var(--border2);
  cursor: pointer;
  background: transparent;
  transition: transform 0.1s;
}
.stage-dot:hover { transform: scale(1.2); }
.stage-dot.filled { border-color: transparent; }

/* LEGEND */
.legend {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 12px 24px;
  border-top: 0.5px solid var(--border);
  flex-wrap: wrap;
}

.legend-swatch {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  flex-shrink: 0;
}

.legend-text {
  font-size: 11px;
  color: var(--t3);
  font-family: var(--mono);
}

/* MODAL */
.modal-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0,0,0,0.3);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 100;
}

.modal {
  background: var(--bg);
  border: 0.5px solid var(--border2);
  border-radius: var(--rl);
  padding: 20px 24px;
  width: 272px;
}

.modal-label {
  font-size: 12px;
  font-weight: 500;
  color: var(--t2);
  margin-bottom: 10px;
  font-family: var(--mono);
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

.modal-input {
  width: 100%;
  background: var(--bg2);
  border: 0.5px solid var(--border2);
  border-radius: var(--r);
  padding: 8px 10px;
  font-size: 14px;
  color: var(--t1);
  font-family: var(--sans);
  outline: none;
  margin-bottom: 12px;
}
.modal-input:focus { border-color: rgba(0,0,0,0.35); }

.modal-actions {
  display: flex;
  gap: 8px;
  justify-content: flex-end;
}

.modal-actions button {
  font-size: 13px;
  font-family: var(--sans);
  padding: 6px 14px;
  border-radius: var(--r);
  cursor: pointer;
  border: 0.5px solid var(--border2);
  background: none;
  color: var(--t2);
  transition: background 0.1s, color 0.1s;
}
.modal-actions button:not(.primary):hover {
  background: var(--bg2);
  color: var(--t1);
}
.modal-actions .primary {
  background: var(--t1);
  color: var(--bg);
  border-color: var(--t1);
}
.modal-actions .primary:hover { opacity: 0.82; }
</style>