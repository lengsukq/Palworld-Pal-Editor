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
        <p class="label">
          {{ palStore.getTranslatedText("BatchTemplate_Target") }}
          <span class="count">
            ({{ targetCount }} {{ palStore.getTranslatedText("BatchTemplate_Target_CountUnit") }})
          </span>
        </p>
        <p class="hint small">
          {{ palStore.getTranslatedText("BatchTemplate_Target_Hint") }}
        </p>
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
        <button class="btn secondary" @click="close" :disabled="palStore.LOADING_FLAG">
          {{ palStore.getTranslatedText("Cancel") }}
        </button>
        <button
          class="btn primary"
          @click="apply"
          :disabled="palStore.LOADING_FLAG || !canApply"
        >
          {{ palStore.getTranslatedText("BatchTemplate_Confirm") }}
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed } from "vue";
import { usePalEditorStore } from "@/stores/paleditor";

const palStore = usePalEditorStore();

const targetCount = computed(() => palStore.getSameSpeciesPalIds().length);

const canApply = computed(() => {
  return (
    targetCount.value > 0 &&
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
  width: min(620px, 92vw);
  max-height: 80vh;
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
</style>

