<template>
  <div class="container">
    <div class="app-container">
      <div class="list-icon" @click="toggleDropdown">
        <div class="list-line"></div>
        <div class="list-line"></div>
        <div class="list-line"></div>
        <div v-if="showDropdown" class="dropdown-menu">
          <div class="color-picker-dropdown">
            <div
              class="color-wheel"
              ref="colorWheel"
              @mousedown="startDragging"
            >
              <div
                class="color-picker-handle"
                :style="{
                  left: `${pickerPosition.x}px`,
                  top: `${pickerPosition.y}px`,
                  backgroundColor: currentColor,
                }"
              ></div>
            </div>
            <div
              class="color-preview"
              :style="{ backgroundColor: currentColor }"
            ></div>
            <button class="apply-color-btn" @click="applyThemeColor">
              Apply Color
            </button>
          </div>
        </div>
      </div>

      <div class="title-container">
        <h1 class="title">TaskPill</h1>
      </div>

      <div class="sections">
        <div
          v-for="(section, index) in sections"
          :key="section.id"
          class="section"
        >
          <div class="section-header" @click="toggleSection(index)">
            <h2
              class="section-title"
              @dblclick.stop="startEditing(index, 'title')"
              v-if="!section.isEditingTitle"
            >
              {{ section.name }}
            </h2>
            <input
              v-else
              ref="titleInputs"
              v-model="section.name"
              class="edit-input"
              @blur="finishEditing(index, 'title')"
              @keyup.enter="finishEditing(index, 'title')"
              @keyup.esc="cancelEditing(index, 'title')"
              @dblclick.stop
            />
            <div class="section-actions">
              <span class="arrow" :class="{ 'arrow-down': section.isOpen }"
                >▼</span
              >
            </div>
          </div>
          <div class="delete-btn-container">
            <button
              class="delete-btn"
              @click.stop="(e) => showDeleteConfirm(index, 'section', null, e)"
            >
              ×
            </button>
            <div
              v-if="
                deleteConfirm.show &&
                deleteConfirm.type === 'section' &&
                deleteConfirm.index === index
              "
              class="delete-confirm-popup"
              :style="{
                top: deleteConfirm.y + 'px',
                left: deleteConfirm.x + 'px',
              }"
            >
              <div class="delete-confirm-text">Delete?</div>
              <div class="delete-confirm-buttons">
                <button
                  class="confirm-yes"
                  @click.stop="confirmDelete(index, 'section')"
                >
                  yes
                </button>
                <button class="confirm-no" @click.stop="cancelDelete()">
                  no
                </button>
              </div>
            </div>
          </div>
          <div
            class="section-content"
            :class="{ 'section-open': section.isOpen }"
          >
            <HabitCard
              v-for="(habit, habitIndex) in section.habits"
              :key="habit.name"
              :habit="habit"
              :onToggle="(index) => toggle(habit.name, index)"
              :onRename="(newName) => renameHabit(index, habitIndex, newName)"
              :onDelete="() => deleteHabit(index, habitIndex)"
            />
            <div class="add-habit-container">
              <button class="add-habit-btn" @click.stop="addHabit(index)">
                +
              </button>
            </div>
          </div>
        </div>

        <div class="add-track-container">
          <div
            class="add-track-button"
            @click="showTrackOptions = !showTrackOptions"
          >
            <span class="plus-icon">+</span>
            <span class="add-track-text">Add New Track</span>
          </div>

          <!-- Track Type Selection Options -->
          <div v-if="showTrackOptions" class="track-options">
            <div class="track-option" @click="addNewTrack('checkbox')">
              <div class="option-preview">
                <div class="preview-habit">
                  <span class="preview-habit-name">Example Habit</span>
                  <div class="preview-checkboxes">
                    <div class="preview-checkbox"></div>
                    <div class="preview-checkbox"></div>
                    <div class="preview-checkbox"></div>
                  </div>
                </div>
              </div>
              <span class="option-title">Checkboxes</span>
            </div>

            <div class="track-option" @click="addNewTrack('progress')">
              <div class="option-preview">
                <div class="preview-habit">
                  <span class="preview-habit-name">Example Habit</span>
                  <div class="preview-progress">
                    <div class="progress-bar">
                      <div class="progress-fill" style="width: 60%"></div>
                    </div>
                  </div>
                </div>
              </div>
              <span class="option-title">Progress Bar</span>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import HabitCard from "~/components/HabitCard.vue";
import { ref, nextTick, onMounted, onUnmounted } from "vue";

const sections = ref([
  {
    id: 1,
    name: "Daily Health",
    type: "checkbox",
    habits: [
      {
        id: 1,
        name: "Drink Water",
        progress: [false, false, false],
        type: "checkbox",
      },
      {
        id: 2,
        name: "Exercise",
        progress: [false, false, false],
        type: "checkbox",
      },
    ],
  },
  {
    id: 2,
    name: "Personal Growth",
    type: "checkbox",
    habits: [
      {
        id: 3,
        name: "Read",
        progress: [false, false, false],
        type: "checkbox",
      },
      {
        id: 4,
        name: "Meditate",
        progress: [false, false, false],
        type: "checkbox",
      },
    ],
  },
]);

const titleInputs = ref([]);
const showTrackOptions = ref(false);
const showDropdown = ref(false);
const showColorPicker = ref(false);
const currentColor = ref("#ff6b35");
const pickerPosition = ref({ x: 100, y: 100 });
const isDragging = ref(false);
const colorWheel = ref(null);

const deleteConfirm = ref({
  show: false,
  type: null,
  index: null,
  sectionIndex: null,
  x: 0,
  y: 0,
});

function toggleSection(index) {
  sections.value[index].isOpen = !sections.value[index].isOpen;
}

function toggle(habitName, index) {
  for (const section of sections.value) {
    const habit = section.habits.find((h) => h.name === habitName);
    if (habit) {
      habit.progress[index] = !habit.progress[index];
      break;
    }
  }
}

function startEditing(index, type) {
  if (type === "title") {
    // Store the original name in case we need to cancel
    sections.value[index].originalName = sections.value[index].name;
    sections.value[index].isEditingTitle = true;
    nextTick(() => {
      if (titleInputs.value[index]) {
        titleInputs.value[index].focus();
        titleInputs.value[index].select();
      }
    });
  }
}

function finishEditing(index, type) {
  if (type === "title") {
    const section = sections.value[index];
    if (section.name && section.name.trim() !== "") {
      section.name = section.name.trim();
    } else {
      // If empty, revert to original name or default
      section.name = section.originalName || "New Track";
    }
    section.isEditingTitle = false;
    delete section.originalName;
  }
}

function cancelEditing(index, type) {
  if (type === "title") {
    const section = sections.value[index];
    section.name = section.originalName;
    section.isEditingTitle = false;
    delete section.originalName;
  }
}

function renameHabit(sectionIndex, habitIndex, newName) {
  if (newName && newName.trim() !== "") {
    sections.value[sectionIndex].habits[habitIndex].name = newName.trim();
  }
}

function addNewTrack(type) {
  sections.value.push({
    id: Date.now(),
    name: "New Track",
    type: type,
    isEditingTitle: true, // Start in editing mode
    habits: [],
  });
  showTrackOptions.value = false;

  // Focus on the title input after the DOM updates
  nextTick(() => {
    const newIndex = sections.value.length - 1;
    if (titleInputs.value && titleInputs.value[newIndex]) {
      titleInputs.value[newIndex].focus();
      titleInputs.value[newIndex].select();
    }
  });
}

function showDeleteConfirm(index, type, sectionIndex = null, event) {
  const rect = event.target.getBoundingClientRect();
  deleteConfirm.value = {
    show: true,
    type,
    index,
    sectionIndex,
    x: rect.right + 20,
    y: rect.top - 6,
  };
}

function confirmDelete(index, type) {
  if (type === "section") {
    sections.value.splice(index, 1);
  } else if (type === "habit") {
    sections.value[deleteConfirm.value.sectionIndex].habits.splice(index, 1);
  }
  cancelDelete();
}

function cancelDelete() {
  deleteConfirm.value = {
    show: false,
    type: null,
    index: null,
    sectionIndex: null,
    x: 0,
    y: 0,
  };
}

function deleteSection(index) {
  showDeleteConfirm(index, "section");
}

function deleteHabit(sectionIndex, habitIndex) {
  sections.value[sectionIndex].habits.splice(habitIndex, 1);
}

function addHabit(sectionIndex) {
  const section = sections.value[sectionIndex];
  section.habits.push({
    id: Date.now(),
    name: "New Habit",
    progress: [false, false, false],
    type: section.type || "checkbox", // Inherit type from section
  });
}

function toggleDropdown() {
  showDropdown.value = !showDropdown.value;
}

function openColorPicker() {
  showDropdown.value = false;
  showColorPicker.value = true;
}

function closeColorPicker() {
  showColorPicker.value = false;
}

function startDragging(event) {
  isDragging.value = true;
  handleDragging(event);
  document.addEventListener("mousemove", handleDragging);
  document.addEventListener("mouseup", stopDragging);
}

function handleDragging(event) {
  if (!isDragging.value && event.type === "mousemove") return;

  const rect = colorWheel.value.getBoundingClientRect();
  const x = event.clientX - rect.left;
  const y = event.clientY - rect.top;

  // Keep the picker within the color wheel
  const radius = rect.width / 2;
  const centerX = radius;
  const centerY = radius;
  const distance = Math.sqrt(
    Math.pow(x - centerX, 2) + Math.pow(y - centerY, 2)
  );

  if (distance <= radius) {
    pickerPosition.value = { x, y };
    updateColor(x, y, radius);
  }
}

function stopDragging() {
  isDragging.value = false;
  document.removeEventListener("mousemove", handleDragging);
  document.removeEventListener("mouseup", stopDragging);
}

function updateColor(x, y, radius) {
  const centerX = radius;
  const centerY = radius;

  // Calculate hue (0-360 degrees)
  const angle = Math.atan2(y - centerY, x - centerX);
  let hue = ((angle * 180) / Math.PI + 360) % 360;

  // Calculate saturation (0-100%)
  const distance = Math.sqrt(
    Math.pow(x - centerX, 2) + Math.pow(y - centerY, 2)
  );
  const saturation = Math.min((distance / radius) * 100, 100);

  // Set lightness to 50% for vibrant colors
  const color = `hsl(${hue}, ${saturation}%, 50%)`;
  currentColor.value = color;

  // Update the picker handle color to match the selected color
  const pickerHandle = document.querySelector(".color-picker-handle");
  if (pickerHandle) {
    pickerHandle.style.backgroundColor = color;
  }
}

function applyThemeColor() {
  const color = currentColor.value;

  // Update all theme colors
  document.documentElement.style.setProperty("--theme-color", color);

  // Convert HSL to RGB for transparency
  const hsl = color.match(/\d+/g);
  const h = hsl[0];
  const s = hsl[1];
  const l = hsl[2];

  // Create variations with transparency
  document.documentElement.style.setProperty(
    "--theme-color-light",
    `hsla(${h}, ${s}%, ${l}%, 0.2)`
  );
  document.documentElement.style.setProperty(
    "--theme-color-dark",
    `hsla(${h}, ${s}%, ${l}%, 0.6)`
  );

  // Update all elements that use the theme color
  const elements = document.querySelectorAll('[style*="--theme-color"]');
  elements.forEach((el) => {
    const styles = window.getComputedStyle(el);
    if (styles.color.includes("var(--theme-color)")) {
      el.style.color = color;
    }
    if (styles.backgroundColor.includes("var(--theme-color)")) {
      el.style.backgroundColor = color;
    }
    if (styles.borderColor.includes("var(--theme-color)")) {
      el.style.borderColor = color;
    }
  });

  // Close the dropdown
  showDropdown.value = false;
}

// Close dropdown when clicking outside
onMounted(() => {
  document.addEventListener("click", (event) => {
    const target = event.target;
    const listIcon = document.querySelector(".list-icon");
    const dropdownMenu = document.querySelector(".dropdown-menu");

    if (listIcon && dropdownMenu) {
      if (!listIcon.contains(target) && !dropdownMenu.contains(target)) {
        showDropdown.value = false;
      }
    }
  });
});

onUnmounted(() => {
  document.removeEventListener("click", handleClickOutside);
});
</script>

<style scoped>
@import url("https://fonts.cdnfonts.com/css/pop-core");
@import url("https://fonts.googleapis.com/css2?family=Orbitron:wght@400;500;600&display=swap");
@import url("https://fonts.googleapis.com/css2?family=Chakra+Petch:wght@300;400;500;600&display=swap");

.container {
  padding: 2rem;
  background-color: #0a0f1c;
  min-height: 100vh;
  position: relative;
}

.app-container {
  padding: 2rem;
  background-color: #0a0f1c;
  min-height: 100vh;
}

.list-icon {
  position: fixed;
  top: 20px;
  left: 20px;
  width: 30px;
  height: 24px;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  cursor: pointer;
  z-index: 1000;
  padding: 4px;
  border-radius: 4px;
  transition: background-color 0.2s ease;
}

.list-icon:hover {
  background-color: rgba(255, 107, 53, 0.1);
}

.list-line {
  width: 100%;
  height: 2px;
  background-color: #ff6b35;
  border-radius: 2px;
  transition: transform 0.2s ease, opacity 0.2s ease;
}

.title-container {
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 2rem;
  position: relative;
}

.title {
  font-family: "Pop Core", sans-serif;
  font-size: 4.5rem;
  color: #ff6b35;
  text-align: center;
  margin-bottom: 3rem;
  position: relative;
  transform: scale(1);
  transition: transform 0.2s;
  letter-spacing: 2px;
  font-weight: bold;
  z-index: 1;
  text-shadow: 0 0 5px rgba(255, 107, 53, 0.5), -4px -4px 0 #000,
    4px -4px 0 #000, -4px 4px 0 #000, 4px 4px 0 #000, -3px -3px 0 #000,
    3px -3px 0 #000, -3px 3px 0 #000, 3px 3px 0 #000, -2px -2px 0 #000,
    2px -2px 0 #000, -2px 2px 0 #000, 2px 2px 0 #000, -1px -1px 0 #000,
    1px -1px 0 #000, -1px 1px 0 #000, 1px 1px 0 #000,
    0 0 10px rgba(255, 107, 53, 0.3);
}

.title::before {
  content: "TaskPill";
  position: absolute;
  left: 0;
  right: 0;
  z-index: -1;
  color: transparent;
  text-shadow: 0 0 13px rgba(255, 107, 53, 0.26),
    0 0 26px rgba(255, 107, 53, 0.19), 0 0 38px rgba(255, 107, 53, 0.13),
    0 0 51px rgba(255, 107, 53, 0.06), 0 0 64px rgba(255, 107, 53, 0.03);
  animation: glow 2s ease-in-out infinite alternate;
}

.sections {
  max-width: 800px;
  margin: 0 auto;
}

.section {
  margin-bottom: 1.5rem;
  background: rgba(255, 255, 255, 0.05);
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.3), 0 0 5px rgba(255, 107, 53, 0.2);
  position: relative;
}

.section-header {
  padding: 1rem 1.5rem;
  background: linear-gradient(
    145deg,
    rgba(255, 107, 53, 0.15),
    rgba(255, 107, 53, 0.05)
  );
  cursor: pointer;
  display: flex;
  justify-content: space-between;
  align-items: center;
  transition: all 0.3s;
  position: relative;
  border: 1px solid rgba(255, 107, 53, 0.3);
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.3), 0 0 5px rgba(255, 107, 53, 0.2),
    inset 0 1px 1px rgba(255, 255, 255, 0.1);
  transform: translateY(0);
}

.section-header:hover {
  background: linear-gradient(
    145deg,
    rgba(255, 107, 53, 0.2),
    rgba(255, 107, 53, 0.1)
  );
  transform: translateY(-2px);
  box-shadow: 0 6px 12px rgba(0, 0, 0, 0.4), 0 0 8px rgba(255, 107, 53, 0.3),
    inset 0 1px 1px rgba(255, 255, 255, 0.2);
}

.section-header::before {
  content: "";
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 1px;
  background: linear-gradient(90deg, transparent, #ff6b35, transparent);
  box-shadow: 0 0 8px #ff6b35;
}

.section-header::after {
  content: "";
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  height: 1px;
  background: linear-gradient(90deg, transparent, #ff6b35, transparent);
  box-shadow: 0 0 8px #ff6b35;
}

.section-title {
  font-family: "Chakra Petch", sans-serif;
  color: #ff6b35;
  font-size: 1.5rem;
  margin: 0;
  cursor: text;
}

.edit-input {
  font-family: "Chakra Petch", sans-serif;
  color: #ff6b35;
  font-size: 1.5rem;
  background: transparent;
  border: none;
  border-bottom: 1px solid #ff6b35;
  outline: none;
  width: 100%;
  padding: 0;
  margin: 0;
}

.section-actions {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
  position: relative;
  width: 30px;
  margin-left: auto;
}

.arrow {
  color: transparent;
  transition: transform 0.3s;
  font-size: 1.2rem;
  position: relative;
  width: 20px;
  height: 20px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.arrow::before {
  content: "▼";
  position: absolute;
  color: transparent;
  -webkit-text-stroke: 1px #ff6b35;
  text-shadow: 0 0 5px rgba(255, 107, 53, 0.5);
}

.arrow-down {
  transform: rotate(180deg);
}

.section-content {
  max-height: 0;
  overflow: hidden;
  transition: max-height 0.3s ease-out;
  margin-top: 0;
}

.section-content.section-open {
  max-height: 2000px;
  margin-top: 1rem;
}

.add-track-container {
  margin-top: 2rem;
}

.track-options {
  display: flex;
  gap: 2rem;
  justify-content: center;
  margin-top: 1rem;
  padding: 1rem;
  background: #0a0f1c;
  border-radius: 12px;
  box-shadow: 0 0 20px rgba(0, 0, 0, 0.5), 0 0 10px rgba(255, 107, 53, 0.3);
  border: 1px solid rgba(255, 107, 53, 0.2);
}

.add-track-button {
  display: flex;
  align-items: center;
  justify-content: center;
  background: linear-gradient(
    145deg,
    rgba(255, 107, 53, 0.15),
    rgba(255, 107, 53, 0.05)
  );
  border-radius: 12px;
  padding: 1rem;
  margin-top: 2rem;
  cursor: pointer;
  transition: all 0.3s ease;
  border: 2px dashed rgba(255, 107, 53, 0.3);
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.3), 0 0 5px rgba(255, 107, 53, 0.2),
    inset 0 1px 1px rgba(255, 255, 255, 0.1);
  transform: translateY(0);
}

.add-track-button:hover {
  background: linear-gradient(
    145deg,
    rgba(255, 107, 53, 0.2),
    rgba(255, 107, 53, 0.1)
  );
  border-color: rgba(255, 107, 53, 0.5);
  transform: translateY(-2px);
  box-shadow: 0 6px 12px rgba(0, 0, 0, 0.4), 0 0 8px rgba(255, 107, 53, 0.3),
    inset 0 1px 1px rgba(255, 255, 255, 0.2);
}

.plus-icon {
  font-size: 1.5rem;
  color: #4caf50;
  margin-right: 0.5rem;
  font-weight: bold;
}

.add-track-text {
  color: #ff6b35;
  font-family: "Chakra Petch", sans-serif;
  font-size: 1.2rem;
  font-weight: 500;
}

.delete-btn {
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
  position: absolute;
  top: 1px;
  right: 2px;
  z-index: 20;
  opacity: 0;
  visibility: hidden;
  transition: opacity 0.3s ease, visibility 0.3s ease, transform 0.2s ease;
  padding: 0;
  line-height: 1;
}

.section:hover .delete-btn {
  opacity: 1;
  visibility: visible;
}

.delete-btn:hover {
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

@keyframes glow {
  from {
    text-shadow: 0 0 13px rgba(255, 107, 53, 0.26),
      0 0 26px rgba(255, 107, 53, 0.19), 0 0 38px rgba(255, 107, 53, 0.13),
      0 0 51px rgba(255, 107, 53, 0.06);
  }
  to {
    text-shadow: 0 0 19px rgba(255, 107, 53, 0.32),
      0 0 32px rgba(255, 107, 53, 0.26), 0 0 45px rgba(255, 107, 53, 0.19),
      0 0 58px rgba(255, 107, 53, 0.13), 0 0 71px rgba(255, 107, 53, 0.06);
  }
}

.title:hover {
  transform: scale(1.05);
}

.add-habit-container {
  display: flex;
  justify-content: center;
  padding: 0.5rem;
  position: relative;
  height: 40px;
}

.add-habit-btn {
  background: linear-gradient(
    145deg,
    rgba(255, 107, 53, 0.1),
    rgba(255, 107, 53, 0.05)
  );
  border: 1px solid rgba(255, 107, 53, 0.3);
  color: #4caf50;
  font-size: 1.2rem;
  cursor: pointer;
  width: 30px;
  height: 30px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 6px;
  transition: all 0.2s;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2),
    inset 0 1px 1px rgba(255, 255, 255, 0.1);
  transform: translateY(0);
  opacity: 0;
  visibility: hidden;
  transition: opacity 0.3s ease, visibility 0.3s ease, transform 0.2s ease;
}

.add-habit-container:hover .add-habit-btn {
  opacity: 1;
  visibility: visible;
}

.add-habit-btn:hover {
  background: linear-gradient(
    145deg,
    rgba(255, 107, 53, 0.2),
    rgba(255, 107, 53, 0.1)
  );
  transform: translateY(-1px);
  box-shadow: 0 3px 6px rgba(0, 0, 0, 0.3),
    inset 0 1px 1px rgba(255, 255, 255, 0.2);
}

.track-options-modal {
  display: none;
}

.track-option {
  cursor: pointer;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1rem;
  padding: 1rem;
  border-radius: 8px;
  transition: all 0.3s ease;
  background: rgba(255, 107, 53, 0.05);
  border: 1px solid rgba(255, 107, 53, 0.1);
  width: 220px;
}

.track-option:hover {
  background: rgba(255, 107, 53, 0.1);
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(255, 107, 53, 0.2);
}

.option-preview {
  width: 200px;
  height: 120px;
  background: #0d1326;
  border-radius: 8px;
  padding: 1rem;
  display: flex;
  flex-direction: column;
  gap: 1rem;
  border: 1px solid rgba(255, 107, 53, 0.2);
}

.preview-habit {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.preview-habit-name {
  color: #ff6b35;
  font-family: "Chakra Petch", sans-serif;
  font-size: 0.9rem;
}

.preview-checkboxes {
  display: flex;
  gap: 0.5rem;
}

.preview-checkbox {
  width: 16px;
  height: 16px;
  border: 2px solid #ff6b35;
  border-radius: 4px;
  background: transparent;
}

.preview-progress {
  width: 100%;
  height: 8px;
  background: rgba(255, 107, 53, 0.1);
  border-radius: 4px;
  overflow: hidden;
}

.progress-bar {
  width: 100%;
  height: 100%;
  background: rgba(255, 107, 53, 0.1);
  border-radius: 4px;
  overflow: hidden;
}

.progress-fill {
  height: 100%;
  background: #ff6b35;
  border-radius: 4px;
}

.option-title {
  color: #ff6b35;
  font-family: "Chakra Petch", sans-serif;
  font-size: 1.1rem;
  font-weight: 500;
}

.title-container:hover .list-icon {
  opacity: 1;
}

.option {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.5rem;
  padding: 1rem;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.3s ease;
  background: linear-gradient(
    145deg,
    rgba(255, 107, 53, 0.1),
    rgba(255, 107, 53, 0.05)
  );
  border: 1px solid rgba(255, 107, 53, 0.3);
  color: #ff6b35;
}

.option:hover {
  background: linear-gradient(
    145deg,
    rgba(255, 107, 53, 0.2),
    rgba(255, 107, 53, 0.1)
  );
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.3),
    inset 0 1px 1px rgba(255, 255, 255, 0.2);
}

.option-title {
  font-family: "Chakra Petch", sans-serif;
  font-size: 0.9rem;
  color: #ff6b35;
  text-align: center;
}

.dropdown-menu {
  position: fixed;
  top: 55px;
  left: 20px;
  background: #0a0f1c;
  border: 1px solid rgba(255, 107, 53, 0.3);
  border-radius: 8px;
  padding: 16px;
  min-width: 220px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.3);
  animation: slideDown 0.2s ease-out;
  transform-origin: top left;
  z-index: 1000;
}

.color-picker-dropdown {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 12px;
}

.color-wheel {
  width: 180px;
  height: 180px;
  border-radius: 50%;
  position: relative;
  cursor: crosshair;
  background: conic-gradient(
    hsl(0, 100%, 50%),
    hsl(60, 100%, 50%),
    hsl(120, 100%, 50%),
    hsl(180, 100%, 50%),
    hsl(240, 100%, 50%),
    hsl(300, 100%, 50%),
    hsl(360, 100%, 50%)
  );
  box-shadow: 0 0 20px rgba(0, 0, 0, 0.3);
  border: 2px solid rgba(255, 255, 255, 0.1);
  overflow: hidden;
}

.color-picker-handle {
  width: 16px;
  height: 16px;
  border: 2px solid white;
  border-radius: 50%;
  position: absolute;
  transform: translate(-50%, -50%);
  pointer-events: none;
  box-shadow: 0 0 4px rgba(0, 0, 0, 0.5);
  transition: all 0.1s ease;
  background-color: var(--theme-color);
}

.color-preview {
  width: 50px;
  height: 50px;
  border-radius: 8px;
  border: 2px solid rgba(255, 255, 255, 0.1);
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.3);
  transition: background-color 0.1s ease;
  background-color: var(--theme-color);
}

.apply-color-btn {
  padding: 8px 16px;
  border-radius: 6px;
  border: 1px solid rgba(255, 107, 53, 0.3);
  background: linear-gradient(
    145deg,
    rgba(255, 107, 53, 0.1),
    rgba(255, 107, 53, 0.05)
  );
  color: #ff6b35;
  font-family: "Chakra Petch", sans-serif;
  cursor: pointer;
  transition: all 0.2s;
  width: 100%;
  text-align: center;
}

.apply-color-btn:hover {
  background: linear-gradient(
    145deg,
    rgba(255, 107, 53, 0.2),
    rgba(255, 107, 53, 0.1)
  );
  transform: translateY(-1px);
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
}

.delete-btn-container {
  position: absolute;
  top: 1px;
  right: 2px;
  z-index: 20;
  opacity: 0;
  visibility: hidden;
  transition: opacity 0.3s ease, visibility 0.3s ease;
}

.section:hover .delete-btn-container,
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
</style>
