<template>
  <div class="habit-card">
    <div class="delete-btn-container">
      <button
        class="habit-delete-btn"
        @click.stop="(e) => showDeleteConfirm(e)"
      >
        ×
      </button>
      <div
        v-if="showConfirm"
        class="delete-confirm-popup"
        :style="{ top: popupPosition.y + 'px', left: popupPosition.x + 'px' }"
      >
        <div class="delete-confirm-text">Delete?</div>
        <div class="delete-confirm-buttons">
          <button class="confirm-yes" @click.stop="confirmDelete">yes</button>
          <button class="confirm-no" @click.stop="cancelDelete">no</button>
        </div>
      </div>
    </div>
    <div class="habit-content">
      <span class="habit-name" @click="startEditing" v-if="!isEditing">
        {{ habit.name }}
      </span>
      <input
        v-else
        ref="nameInput"
        v-model="editedName"
        class="edit-input"
        @blur="finishEditing"
        @keyup.enter="finishEditing"
        @keyup.esc="cancelEditing"
      />

      <!-- Weekly grid only for checkbox type -->
      <div v-if="habit.type === 'checkbox'" class="weekly-grid">
        <div class="weekly-tracker">
          <div
            v-for="(day, index) in weekDays"
            :key="`day-${index}`"
            class="day-container"
            @mouseenter="setHoveredDay(day)"
            @mouseleave="clearHoveredDay"
          >
            <span class="day-label">{{ day }}</span>
            <div class="day-progress">
              <div class="day-checkboxes">
                <div
                  v-for="(checked, boxIndex) in habit.dailyProgress[
                    day.toLowerCase()
                  ]"
                  :key="boxIndex"
                  :class="['progress-dot', { filled: checked }]"
                  @click="toggleDailyBox(day, boxIndex)"
                />
              </div>
              <div class="day-controls">
                <button
                  class="day-control-btn add-btn"
                  @click.stop="addDailyBox(day)"
                >
                  +
                </button>
                <button
                  class="day-control-btn remove-btn"
                  @click.stop="removeDailyBox(day)"
                >
                  -
                </button>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- Progress bar type remains unchanged -->
      <template v-else-if="habit.type === 'progress'">
        <div class="progress-container">
          <div class="progress-bar">
            <div
              class="progress-fill"
              :style="{
                width: `${
                  (habit.progress.filter((p) => p).length /
                    habit.progress.length) *
                  100
                }%`,
                backgroundColor: 'var(--theme-color, #ff6b35)',
              }"
            ></div>
          </div>
          <span class="progress-percentage">
            {{
              Math.round(
                (habit.progress.filter((p) => p).length /
                  habit.progress.length) *
                  100
              )
            }}%
          </span>
        </div>
        <div class="progress-dots-container">
          <div class="progress-dots">
            <div
              v-for="(step, index) in habit.progress"
              :key="index"
              :class="['progress-dot', { filled: step }]"
              @click="toggleProgressBox(index)"
              :style="{
                borderColor: 'var(--theme-color, #ff6b35)',
                backgroundColor: step
                  ? 'var(--theme-color, #ff6b35)'
                  : 'transparent',
              }"
            />
          </div>
          <div class="control-buttons">
            <button class="control-btn add-btn" @click.stop="addProgressBox">
              +
            </button>
            <button
              class="control-btn remove-btn"
              @click.stop="removeProgressBox"
            >
              -
            </button>
          </div>
        </div>
      </template>
    </div>
    <!-- Icon squares container moved to bottom -->
    <div class="icon-squares">
      <div class="icon-square" @click="handleIconClick('stats')">📊</div>
      <div class="icon-square" @click="handleIconClick('notes')">📝</div>
      <div class="icon-square" @click="handleIconClick('reminder')">⏰</div>
      <div class="icon-square" @click="handleIconClick('share')">🔗</div>
    </div>
  </div>
</template>

<script setup>
import { ref, nextTick, onMounted, onUnmounted } from "vue";

const props = defineProps({
  habit: Object,
  onToggle: Function,
  onDelete: Function,
  onRename: Function,
});

const isEditing = ref(false);
const editedName = ref("");
const nameInput = ref(null);
const showConfirm = ref(false);
const popupPosition = ref({ x: 0, y: 0 });

const weekDays = ["Mon", "Tue", "Wed", "Thu", "Fri", "Sat", "Sun"];

// Initialize daily progress if not exists
if (!props.habit.dailyProgress) {
  props.habit.dailyProgress = weekDays.reduce((acc, day) => {
    acc[day.toLowerCase()] = [];
    return acc;
  }, {});
}

// Track which day container is being hovered
const hoveredDay = ref(null);

function setHoveredDay(day) {
  hoveredDay.value = day;
}

function clearHoveredDay() {
  hoveredDay.value = null;
}

function addDailyBox(day) {
  props.habit.dailyProgress[day.toLowerCase()].push(false);
}

function removeDailyBox(day) {
  const dayProgress = props.habit.dailyProgress[day.toLowerCase()];
  if (dayProgress.length > 0) {
    dayProgress.pop();
  }
}

function toggleDailyBox(day, index) {
  props.habit.dailyProgress[day.toLowerCase()][index] =
    !props.habit.dailyProgress[day.toLowerCase()][index];
}

function startEditing() {
  isEditing.value = true;
  editedName.value = props.habit.name;
  nextTick(() => {
    nameInput.value.focus();
    nameInput.value.select();
  });
}

function finishEditing() {
  if (editedName.value.trim()) {
    props.onRename(editedName.value.trim());
  }
  isEditing.value = false;
}

function cancelEditing() {
  isEditing.value = false;
}

function toggleProgressBox(index) {
  props.habit.progress[index] = !props.habit.progress[index];
  if (props.onToggle) {
    props.onToggle(index);
  }
}

function addProgressBox() {
  props.habit.progress.push(false);
}

function removeProgressBox() {
  if (props.habit.progress.length > 1) {
    props.habit.progress.pop();
  }
}

function showDeleteConfirm(event) {
  const rect = event.target.getBoundingClientRect();
  popupPosition.value = {
    x: rect.right + 20,
    y: rect.top - 6,
  };
  showConfirm.value = true;
}

function confirmDelete() {
  if (props.onDelete) {
    props.onDelete();
  }
  showConfirm.value = false;
}

function cancelDelete() {
  showConfirm.value = false;
}

// Handle keyboard shortcuts
function handleKeyboard(event) {
  // Only process if it's a progress type habit
  if (props.habit.type === "progress") {
    // Check for Mac (Command) or Windows (Ctrl) key
    if (event.metaKey || event.ctrlKey) {
      if (event.key === "+" || event.key === "=") {
        // = is the same key as + without shift
        event.preventDefault();
        addProgressBox();
      } else if (event.key === "-") {
        event.preventDefault();
        removeProgressBox();
      }
    }
  }
}

// Add and remove event listeners
onMounted(() => {
  window.addEventListener("keydown", handleKeyboard);
});

onUnmounted(() => {
  window.removeEventListener("keydown", handleKeyboard);
});

function handleIconClick(type) {
  // This function will be implemented later to handle icon clicks
  console.log(`Icon clicked: ${type}`);
}
</script>

<style scoped>
.habit-card {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  background: #0d1326;
  padding: 1rem;
  border-radius: 10px;
  margin-bottom: 1rem;
  box-shadow: 0 0 10px rgba(255, 107, 53, 0.2);
  position: relative;
  border: 1px solid rgba(255, 107, 53, 0.15);
  box-shadow: 0 0 8px rgba(0, 0, 0, 0.3), 0 0 3px rgba(255, 107, 53, 0.3),
    inset 0 0 2px rgba(255, 107, 53, 0.1);
}

.habit-card::before {
  content: "";
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  border-radius: 10px;
  padding: 1px;
  background: linear-gradient(
    135deg,
    rgba(255, 107, 53, 0.1),
    rgba(255, 107, 53, 0.05)
  );
  mask: linear-gradient(#fff 0 0) content-box, linear-gradient(#fff 0 0);
  -webkit-mask: linear-gradient(#fff 0 0) content-box, linear-gradient(#fff 0 0);
  -webkit-mask-composite: xor;
  mask-composite: exclude;
  pointer-events: none;
}

.habit-name {
  color: #ff6b35;
  font-family: "Chakra Petch", sans-serif;
  font-size: 1rem;
  font-weight: 500;
  flex: 1;
  letter-spacing: 0.5px;
  cursor: pointer;
  padding: 4px;
  border-radius: 4px;
  transition: background-color 0.2s ease;
}
.habit-name:hover {
  background-color: rgba(255, 107, 53, 0.1);
}
.progress-container {
  display: flex;
  gap: 8px;
  align-items: center;
  position: relative;
  margin-left: 0;
}
.progress-dot {
  width: 16px;
  height: 16px;
  border-radius: 6px;
  border: 2px solid #ff6b35;
  background-color: transparent;
  cursor: pointer;
  transition: background-color 0.2s ease;
}
.progress-dot.filled {
  background-color: #ff6b35;
}
.habit-content {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  width: 100%;
}
.control-buttons {
  display: flex;
  flex-direction: column;
  gap: 4px;
  margin-left: 8px;
  opacity: 0;
  visibility: hidden;
  transition: opacity 0.3s ease, visibility 0.3s ease;
}
.progress-container:hover .control-buttons {
  opacity: 1;
  visibility: visible;
}
.control-btn {
  background: linear-gradient(
    145deg,
    rgba(255, 107, 53, 0.1),
    rgba(255, 107, 53, 0.05)
  );
  border: 1px solid rgba(255, 107, 53, 0.3);
  color: #ff6b35;
  font-size: 0.9rem;
  cursor: pointer;
  width: 16px;
  height: 16px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 6px;
  transition: all 0.2s;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2),
    inset 0 1px 1px rgba(255, 255, 255, 0.1);
  transform: translateY(0);
}
.control-btn:hover {
  background: linear-gradient(
    145deg,
    rgba(255, 107, 53, 0.2),
    rgba(255, 107, 53, 0.1)
  );
  transform: translateY(-1px);
  box-shadow: 0 3px 6px rgba(0, 0, 0, 0.3),
    inset 0 1px 1px rgba(255, 255, 255, 0.2);
}
.remove-btn {
  color: #ff4444;
  border-color: rgba(255, 68, 68, 0.3);
}
.remove-btn:hover {
  background: linear-gradient(
    145deg,
    rgba(255, 68, 68, 0.2),
    rgba(255, 68, 68, 0.1)
  );
}
.habit-delete-btn {
  background: linear-gradient(
    145deg,
    rgba(255, 107, 53, 0.1),
    rgba(255, 107, 53, 0.05)
  );
  border: 1px solid rgba(255, 107, 53, 0.3);
  color: #ff6b35;
  font-size: 0.8rem;
  cursor: pointer;
  width: 16px;
  height: 16px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
  transition: all 0.2s;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.3),
    inset 0 1px 1px rgba(255, 255, 255, 0.1);
  padding: 0;
  line-height: 1;
}
.habit-card:hover .habit-delete-btn {
  opacity: 1;
  visibility: visible;
}
.habit-delete-btn:hover {
  background: linear-gradient(
    145deg,
    rgba(255, 68, 68, 0.2),
    rgba(255, 68, 68, 0.1)
  );
  color: #ff4444;
  transform: scale(1.1);
  box-shadow: 0 6px 12px rgba(0, 0, 0, 0.4),
    inset 0 1px 1px rgba(255, 255, 255, 0.2);
}
.edit-input {
  background: transparent;
  border: 1px solid rgba(255, 107, 53, 0.3);
  color: #ff6b35;
  font-family: "Chakra Petch", sans-serif;
  font-size: 1rem;
  padding: 4px;
  border-radius: 4px;
  width: 100%;
  outline: none;
}
.edit-input:focus {
  border-color: #ff6b35;
  box-shadow: 0 0 5px rgba(255, 107, 53, 0.3);
}

.weekly-grid {
  position: relative;
  width: 100%;
  margin-top: 0.5rem;
  padding-top: 0.5rem;
  border-top: 1px solid rgba(255, 107, 53, 0.1);
}

.weekly-tracker {
  display: flex;
  justify-content: space-between;
  width: 100%;
  padding: 0;
  margin: 0;
}

.day-container {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  width: 14.28%;
  padding: 0 4px;
  position: relative;
  border-left: 1px solid rgba(255, 107, 53, 0.1);
  transition: background-color 0.2s ease;
}

.day-container:first-child {
  border-left: none;
}

.day-container:hover {
  background-color: rgba(255, 107, 53, 0.05);
}

.day-progress {
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 100%;
  gap: 4px;
  padding-top: 4px;
}

.day-checkboxes {
  display: flex;
  flex-wrap: wrap;
  flex-direction: row-reverse;
  gap: 4px;
  align-items: center;
  justify-content: flex-end;
  min-height: 24px;
  padding: 4px 0;
  width: 100%;
  max-width: 100%;
  max-height: none;
}

.day-controls {
  display: flex;
  gap: 4px;
  opacity: 0;
  visibility: hidden;
  transition: opacity 0.3s ease, visibility 0.3s ease;
}

.day-container:hover .day-controls {
  opacity: 1;
  visibility: visible;
}

.day-control-btn {
  background: linear-gradient(
    145deg,
    rgba(255, 107, 53, 0.1),
    rgba(255, 107, 53, 0.05)
  );
  border: 1px solid rgba(255, 107, 53, 0.3);
  color: #ff6b35;
  font-size: 0.7rem;
  cursor: pointer;
  width: 12px;
  height: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 4px;
  transition: all 0.2s;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2),
    inset 0 1px 1px rgba(255, 255, 255, 0.1);
  padding: 0;
  line-height: 1;
}

.day-control-btn:hover {
  background: linear-gradient(
    145deg,
    rgba(255, 107, 53, 0.2),
    rgba(255, 107, 53, 0.1)
  );
  transform: translateY(-1px);
  box-shadow: 0 3px 6px rgba(0, 0, 0, 0.3),
    inset 0 1px 1px rgba(255, 255, 255, 0.2);
}

.day-control-btn.remove-btn {
  color: #ff4444;
  border-color: rgba(255, 68, 68, 0.3);
}

.day-control-btn.remove-btn:hover {
  background: linear-gradient(
    145deg,
    rgba(255, 68, 68, 0.2),
    rgba(255, 68, 68, 0.1)
  );
}

.progress-dot {
  width: calc((100% - 16px) / 5); /* 5 boxes per row with 4px gap */
  min-width: 14px;
  height: 14px;
  border-radius: 4px;
  border: 2px solid #ff6b35;
  background-color: transparent;
  cursor: pointer;
  transition: all 0.2s ease;
  flex-shrink: 0;
}

.progress-dot.filled {
  background-color: #ff6b35;
}

.progress-dot:hover {
  transform: scale(1.1);
  box-shadow: 0 0 8px rgba(255, 107, 53, 0.3);
}

.day-label {
  font-size: 0.6rem;
  color: rgba(255, 107, 53, 0.6);
  font-family: "Chakra Petch", sans-serif;
  text-align: left;
}

.progress-container {
  display: flex;
  align-items: center;
  gap: 12px;
  width: 100%;
}

.progress-percentage {
  color: #ff6b35;
  font-family: "Chakra Petch", sans-serif;
  font-size: 0.9rem;
  min-width: 40px;
}

.progress-bar {
  flex: 1;
  height: 8px;
  background: rgba(255, 107, 53, 0.1);
  border-radius: 4px;
  overflow: hidden;
  margin-bottom: 0;
}

.progress-fill {
  height: 100%;
  background: #ff6b35;
  border-radius: 4px;
  transition: width 0.3s ease;
}

.progress-dots-container {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-top: 8px;
  width: 100%;
}

.progress-dots {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  flex: 1;
  min-height: 24px;
}

.progress-dot {
  width: 16px;
  height: 16px;
  border-radius: 4px;
  border: 2px solid var(--theme-color, #ff6b35);
  background-color: transparent;
  cursor: pointer;
  transition: all 0.2s ease;
  flex-shrink: 0;
}

.progress-dot.filled {
  background-color: var(--theme-color, #ff6b35);
}

.progress-dot:hover {
  transform: scale(1.1);
  box-shadow: 0 0 8px rgba(255, 107, 53, 0.3);
}

.control-buttons {
  display: flex;
  flex-direction: column;
  gap: 4px;
  margin-left: 8px;
  opacity: 0;
  visibility: hidden;
  transition: opacity 0.3s ease, visibility 0.3s ease;
}

.progress-dots-container:hover .control-buttons {
  opacity: 1;
  visibility: visible;
}

.control-btn {
  background: linear-gradient(
    145deg,
    rgba(255, 107, 53, 0.1),
    rgba(255, 107, 53, 0.05)
  );
  border: 1px solid var(--theme-color, #ff6b35);
  color: var(--theme-color, #ff6b35);
  font-size: 0.9rem;
  cursor: pointer;
  width: 16px;
  height: 16px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 4px;
  transition: all 0.2s;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2),
    inset 0 1px 1px rgba(255, 255, 255, 0.1);
  transform: translateY(0);
}

.control-btn:hover {
  background: linear-gradient(
    145deg,
    rgba(255, 107, 53, 0.2),
    rgba(255, 107, 53, 0.1)
  );
  transform: translateY(-1px);
  box-shadow: 0 3px 6px rgba(0, 0, 0, 0.3),
    inset 0 1px 1px rgba(255, 255, 255, 0.2);
}

.remove-btn {
  color: #ff4444;
  border-color: rgba(255, 68, 68, 0.3);
}

.remove-btn:hover {
  background: linear-gradient(
    145deg,
    rgba(255, 68, 68, 0.2),
    rgba(255, 68, 68, 0.1)
  );
}

.delete-btn-container {
  position: absolute;
  top: 1px;
  right: 5px;
  z-index: 20;
  opacity: 0;
  visibility: hidden;
  transition: opacity 0.3s ease, visibility 0.3s ease;
}

.habit-card:hover .delete-btn-container {
  opacity: 1;
  visibility: visible;
}

.delete-confirm-popup {
  position: fixed;
  background: #0a0f1c;
  border: 1px solid rgba(255, 107, 53, 0.3);
  border-radius: 8px;
  padding: 8px 12px;
  width: auto;
  min-width: 80px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.3);
  z-index: 1000;
  animation: fadeIn 0.2s ease-out;
  transform-origin: left center;
  white-space: nowrap;
}

.delete-confirm-text {
  color: #ff6b35;
  font-size: 0.9rem;
  text-align: center;
  margin-bottom: 6px;
  font-family: "Chakra Petch", sans-serif;
}

.delete-confirm-buttons {
  display: flex;
  justify-content: center;
  gap: 12px;
}

.confirm-yes,
.confirm-no {
  padding: 4px 8px;
  border-radius: 4px;
  border: none;
  font-size: 0.8rem;
  cursor: pointer;
  transition: all 0.2s ease;
  background: transparent;
  color: #808080;
  font-family: "Chakra Petch", sans-serif;
}

.confirm-yes:hover {
  color: #4caf50;
  background: rgba(76, 175, 80, 0.1);
}

.confirm-no:hover {
  color: #ff4444;
  background: rgba(255, 68, 68, 0.1);
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateX(-10px);
  }
  to {
    opacity: 1;
    transform: translateX(0);
  }
}

.icon-squares {
  display: flex;
  gap: 8px;
  padding: 8px;
  border-top: 1px solid rgba(255, 107, 53, 0.1);
  margin-top: auto;
  justify-content: center;
  background: rgba(255, 107, 53, 0.05);
  border-bottom-left-radius: 10px;
  border-bottom-right-radius: 10px;
}

.icon-square {
  width: 24px;
  height: 24px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 14px;
  cursor: pointer;
  transition: all 0.2s ease;
  color: rgba(255, 107, 53, 0.6);
  border-radius: 4px;
  background: rgba(255, 107, 53, 0.1);
}

.icon-square:hover {
  transform: scale(1.1);
  color: #ff6b35;
  background: rgba(255, 107, 53, 0.2);
}
</style>
