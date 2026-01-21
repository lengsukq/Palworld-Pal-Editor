<template>
  <div
    v-show="palStore.SHOW_BATCH_TEMPLATE_MODAL"
    class="modal-overlay"
    @click.self="close"
  >
    <div class="modal-content">
      <button class="close-btn" @click="close" :disabled="palStore.LOADING_FLAG">×</button>

      <div class="header">
        <h2 class="title">
          {{ palStore.getTranslatedText("BatchTemplate_Title") }}
        </h2>
        <p class="subtitle">
          {{ palStore.getTranslatedText("BatchTemplate_Subtitle") }}
        </p>
      </div>

      <div class="section">
        <div class="label-row">
          <p class="label">
            {{ palStore.getTranslatedText("BatchTemplate_Target") }}
            <span class="count">
              ({{ selectedCount }}/{{ candidatePals.length }})
            </span>
          </p>
          <div class="action-row">
            <input
              v-model="palSearch"
              class="pal-search"
              type="text"
              :placeholder="palStore.getTranslatedText('BatchApply_TargetPals_Search')"
              :disabled="palStore.LOADING_FLAG"
            />
            <button
              class="mini-btn"
              type="button"
              @click="selectAllSameSpecies"
              :disabled="palStore.LOADING_FLAG || !hasSelectedPal"
            >
              {{ palStore.getTranslatedText("BatchApply_SelectSameSpecies") }}
            </button>
            <button
              class="mini-btn ghost"
              type="button"
              @click="palStore.clearBatchPalSelection"
              :disabled="palStore.LOADING_FLAG || !palStore.BATCH_PASSIVE_SELECTED_PAL_IDS.length"
            >
              {{ palStore.getTranslatedText("BatchApply_ClearSelection") }}
            </button>
          </div>
        </div>

        <div class="pal-list">
          <div
            v-for="pal in candidatePals"
            :key="pal.InstanceId"
            class="pal-row"
            @click="togglePal(pal)"
          >
            <input
              type="checkbox"
              class="pal-checkbox"
              :checked="palStore.isPalSelectedForBatch(pal.InstanceId)"
              @change.stop="togglePal(pal)"
            />
            <img
              class="pal-icon"
              :src="`/image/pals/${pal.IconAccessKey}`"
              alt="pal icon"
            />
            <span class="pal-name">{{ pal.DisplayName }}</span>
          </div>
          <p class="hint small">
            {{ palStore.getTranslatedText("BatchTemplate_Target_Hint") }}
          </p>
        </div>
      </div>

      <div
        v-if="palStore.BATCH_TEMPLATE_IN_PROGRESS && palStore.BATCH_TEMPLATE_TOTAL > 0"
        class="section"
      >
        <p class="label">
          {{ palStore.getTranslatedText("BatchTemplate_Progress") }}
          <span class="count">
            ({{ palStore.BATCH_TEMPLATE_PROGRESS }}/{{ palStore.BATCH_TEMPLATE_TOTAL }})
          </span>
        </p>
        <div class="progress-bar">
          <div class="progress-bar-inner" :style="{ width: progressPercent + '%' }"></div>
        </div>
      </div>

      <div class="section">
        <p class="label">{{ palStore.getTranslatedText("BatchTemplate_Sections") }}</p>
        <div class="sections-grid">
          <label class="check-row">
            <input
              type="checkbox"
              v-model="palStore.BATCH_TEMPLATE_INCLUDE_PASSIVE"
              :disabled="palStore.LOADING_FLAG"
            />
            <span>{{ palStore.getTranslatedText("BatchTemplate_Section_Passive") }}</span>
          </label>
          <label class="check-row">
            <input
              type="checkbox"
              v-model="palStore.BATCH_TEMPLATE_INCLUDE_SUITABILITY"
              :disabled="palStore.LOADING_FLAG"
            />
            <span>{{ palStore.getTranslatedText("BatchTemplate_Section_Suitability") }}</span>
          </label>
          <label class="check-row">
            <input
              type="checkbox"
              v-model="palStore.BATCH_TEMPLATE_INCLUDE_TALENT"
              :disabled="palStore.LOADING_FLAG"
            />
            <span>{{ palStore.getTranslatedText("BatchTemplate_Section_Talent") }}</span>
          </label>
          <label class="check-row">
            <input
              type="checkbox"
              v-model="palStore.BATCH_TEMPLATE_INCLUDE_SOULS"
              :disabled="palStore.LOADING_FLAG"
            />
            <span>{{ palStore.getTranslatedText("BatchTemplate_Section_Souls") }}</span>
          </label>
          <label class="check-row">
            <input
              type="checkbox"
              v-model="palStore.BATCH_TEMPLATE_INCLUDE_CONDENSER"
              :disabled="palStore.LOADING_FLAG"
            />
            <span>{{ palStore.getTranslatedText("BatchTemplate_Section_Condenser") }}</span>
          </label>
        </div>
        <p class="hint small">
          {{ palStore.getTranslatedText("BatchTemplate_Sections_Hint") }}
        </p>
      </div>

      <div class="footer">
        <button
          class="btn secondary"
          @click="close"
          :disabled="palStore.LOADING_FLAG || palStore.BATCH_TEMPLATE_IN_PROGRESS"
        >
          {{ palStore.getTranslatedText("Cancel") }}
        </button>
        <button
          class="btn primary"
          @click="apply"
          :disabled="palStore.LOADING_FLAG || !canApply || palStore.BATCH_TEMPLATE_IN_PROGRESS"
        >
          {{ palStore.getTranslatedText("BatchTemplate_Confirm") }}
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed, ref, watch } from "vue";
import { usePalEditorStore } from "@/stores/paleditor";

const palStore = usePalEditorStore();

const palSearch = ref("");

const sameSpeciesIds = computed(() => palStore.getSameSpeciesPalIds());

const candidatePals = computed(() => {
  const all = Array.from(palStore.PAL_MAP.values()).filter(
    (pal) =>
      sameSpeciesIds.value.includes(pal.InstanceId) &&
      pal.InstanceId !== palStore.SELECTED_PAL_DATA?.InstanceId &&
      !palStore.isFilteredPal(pal)
  );
  if (!palSearch.value) return all;
  const keyword = palSearch.value.toLowerCase();
  return all.filter((pal) =>
    pal.DisplayName?.toLowerCase().includes(keyword)
  );
});

const selectedCount = computed(() => {
  return candidatePals.value.filter((pal) =>
    palStore.isPalSelectedForBatch(pal.InstanceId)
  ).length;
});

const hasSelectedPal = computed(() => !!palStore.SELECTED_PAL_DATA);

const progressPercent = computed(() => {
  if (!palStore.BATCH_TEMPLATE_TOTAL) return 0;
  return Math.min(
    100,
    Math.round(
      (palStore.BATCH_TEMPLATE_PROGRESS / palStore.BATCH_TEMPLATE_TOTAL) * 100
    )
  );
});

watch(
  () => palStore.SHOW_BATCH_TEMPLATE_MODAL,
  (isOpen) => {
    if (isOpen) {
      palStore.clearBatchPalSelection();
      palStore.selectSameSpeciesForBatch();
      palSearch.value = "";
    }
  }
);

function togglePal(pal) {
  palStore.toggleBatchPalSelection(pal.InstanceId);
}

function selectAllSameSpecies() {
  palStore.selectSameSpeciesForBatch();
}

const canApply = computed(() => {
  return (
    palStore.BATCH_PASSIVE_SELECTED_PAL_IDS.length > 0 &&
    (palStore.BATCH_TEMPLATE_INCLUDE_PASSIVE ||
      palStore.BATCH_TEMPLATE_INCLUDE_SUITABILITY ||
      palStore.BATCH_TEMPLATE_INCLUDE_TALENT ||
      palStore.BATCH_TEMPLATE_INCLUDE_SOULS ||
      palStore.BATCH_TEMPLATE_INCLUDE_CONDENSER)
  );
});

function close() {
  palStore.SHOW_BATCH_TEMPLATE_MODAL = false;
}

async function apply() {
  await palStore.batchApplyTemplateToSameSpecies();
  palStore.SHOW_BATCH_TEMPLATE_MODAL = false;
}
</script>

<style scoped>
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(22, 27, 34, 0.8);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.modal-content {
  background: #12151a;
  color: #c9d1d9;
  position: relative;
  padding: 18px;
  border-radius: 10px;
  width: min(860px, 92vw);
  max-height: 84vh;
  overflow-y: auto;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.2);
  border: 1px solid #30363d;
}

.close-btn {
  position: absolute;
  top: 10px;
  right: 10px;
  background: none;
  border: none;
  color: #c9d1d9;
  font-size: 1.5rem;
  cursor: pointer;
}

.close-btn:hover {
  color: #f85149;
}

.header {
  margin-bottom: 14px;
  padding-right: 32px;
}

.title {
  margin: 0;
  font-size: 1.25rem;
}

.subtitle {
  margin: 6px 0 0;
  opacity: 0.85;
  font-size: 0.95rem;
}

.section {
  border-top: 1px solid #30363d;
  padding-top: 12px;
  margin-top: 12px;
}

.label {
  margin: 0 0 8px;
  font-weight: 700;
}

.count {
  font-size: 0.85rem;
  opacity: 0.8;
  margin-left: 4px;
}

.hint {
  margin: 6px 0 0;
  opacity: 0.8;
  font-size: 0.9rem;
}

.hint.small {
  font-size: 0.8rem;
  opacity: 0.75;
}

.sections-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
  gap: 8px;
}

.check-row {
  display: flex;
  align-items: center;
  gap: 8px;
  cursor: pointer;
}

.footer {
  display: flex;
  justify-content: flex-end;
  gap: 10px;
  margin-top: 16px;
  padding-top: 12px;
  border-top: 1px solid #30363d;
}

.btn {
  cursor: pointer;
  border: 1px solid #30363d;
  border-radius: 8px;
  padding: 8px 14px;
  background: #1b2028;
  color: #c9d1d9;
}

.btn:disabled {
  opacity: 0.55;
  cursor: not-allowed;
}

.btn.primary {
  background: #2d7d46;
  border-color: #2d7d46;
  color: white;
}

.btn.primary:hover {
  background: #2a6f3f;
}

.btn.secondary:hover {
  background: #242a34;
}

.label-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 12px;
  flex-wrap: wrap;
}

.label-row .label {
  margin: 0;
}

.action-row {
  display: flex;
  gap: 8px;
  align-items: center;
  flex-wrap: wrap;
}

.mini-btn {
  cursor: pointer;
  border-radius: 999px;
  border: 1px solid #3b3f4a;
  padding: 4px 10px;
  background: #1b2028;
  color: #c9d1d9;
  font-size: 0.8rem;
}

.mini-btn.ghost {
  background: transparent;
}

.mini-btn:hover {
  background: #242a34;
}

.mini-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.progress-bar {
  position: relative;
  width: 100%;
  height: 8px;
  border-radius: 999px;
  background: #161b22;
  border: 1px solid #30363d;
  overflow: hidden;
  margin-top: 6px;
}

.progress-bar-inner {
  position: absolute;
  top: 0;
  left: 0;
  height: 100%;
  background: linear-gradient(90deg, #2d7d46, #3fb950);
  transition: width 0.15s ease-out;
}

.pal-search {
  flex: 1 1 140px;
  min-width: 0;
  display: flex;
  align-items: center;
  background-color: #34353a;
  height: 1.6rem;
  padding: 0.2rem 0.8rem;
  border-radius: 1rem;
  color: rgb(208, 212, 226);
  box-shadow: 2px 2px 10px rgb(38, 38, 38);
  border: none;
  outline: none;
  font-size: 0.8rem;
}

.pal-search:focus {
  background-color: #b4b7be;
  color: rgb(0, 0, 0);
}

.pal-list {
  margin-top: 8px;
  max-height: 260px;
  padding: 6px;
  border-radius: 8px;
  background: #0d1117;
  border: 1px solid #30363d;
  overflow-y: auto;
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(210px, 1fr));
  gap: 4px;
}

.pal-row {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 4px 8px;
  border-radius: 6px;
  cursor: pointer;
  transition: background 0.12s ease-in-out;
}

.pal-row:hover {
  background: #161b22;
}

.pal-checkbox {
  flex-shrink: 0;
}

.pal-icon {
  width: 1.8rem;
  height: 1.8rem;
  border-radius: 50%;
  object-fit: cover;
  flex-shrink: 0;
}

.pal-name {
  font-size: 0.9rem;
  color: #e6edf3;
  white-space: nowrap;
  text-overflow: ellipsis;
  overflow: hidden;
}
</style>

