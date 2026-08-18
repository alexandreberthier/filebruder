<template>
  <div class="window">
    <div class="titlebar">
      <span>★ FILEBRUDER.EXE</span>
      <span class="traffic">_ □ ×</span>
    </div>

    <div class="page">
      <h1>
        <span class="sparkle">★</span>
        File Bruder
        <span class="sparkle">★</span>
      </h1>

      <section>
        <div class="section-head">
          <h2>Mandanten</h2>
          <button type="button" class="text-btn" @click="toggleAllMandants">
            {{ allMandantsSelected ? 'Leeren' : 'Alle' }}
          </button>
        </div>
        <DropDownCheckbox
            :options="mandantOptions"
            v-model:selected-options="selectedOptions"
        />
        <p v-if="errorMessage" class="error">{{ errorMessage }}</p>
      </section>

      <section>
        <div class="section-head">
          <h2>Dateien</h2>
          <button v-if="files.length" type="button" class="text-btn" @click="clearFiles">Leeren</button>
        </div>
        <div
            class="drop-zone"
            :class="{ dragging: isDragging }"
            role="button"
            tabindex="0"
            @click="openFilePicker"
            @keydown.enter.prevent="openFilePicker"
            @keydown.space.prevent="openFilePicker"
            @dragenter="handleDragEnter"
            @dragover="handleDragOver"
            @dragleave="handleDragLeave"
            @drop="handleDrop"
        >
          <input
              ref="fileInput"
              class="file-input"
              type="file"
              multiple
              @change="handleFileSelect"
              @click.stop
          />
          <p>{{ isDragging ? '★ LOSLASSEN ★' : 'ABLEGEN ODER KLICKEN' }}</p>
        </div>
        <ul v-if="files.length" class="file-list">
          <li v-for="(file, index) in files" :key="`${file.name}-${file.lastModified}-${index}`">
            <span class="file-name">{{ file.name }}</span>
            <span class="file-size">{{ formatSize(file.size) }}</span>
            <button type="button" class="icon-btn" :aria-label="`${file.name} entfernen`" @click="removeFile(index)">×</button>
          </li>
        </ul>
      </section>

      <DynamicButton
          :disabled="!files.length || isGenerating"
          :loading="isGenerating"
          :text="isGenerating ? 'Erzeuge…' : 'SQL generieren'"
          @click="generateSql"
      />

      <section v-if="sqlStatements.length">
        <div class="section-head">
          <h2>SQL</h2>
          <div class="sql-actions">
            <DynamicButton :text="copyLabel" @click="copyAll"/>
            <DynamicButton :button-type="ButtonType.Secondary" text="Reset" @click="resetSql"/>
          </div>
        </div>
        <pre>{{ displaySql }}</pre>
      </section>
    </div>
  </div>
</template>


<script setup lang="ts">
import DropDownCheckbox, {type Option} from "@/components/DropDownCheckbox.vue"
import {computed, onMounted, onUnmounted, ref, type Ref, useTemplateRef} from "vue"
import DynamicButton from "@/components/DynamicButton.vue";
import {ButtonType} from "@/helpers/ButtonType.ts";

const selectedOptions: Ref<number[]> = ref([])
const files = ref<File[]>([])
const sqlStatements = ref<string[]>([])
const fileInput = useTemplateRef<HTMLInputElement>('fileInput')
const isDragging = ref(false)
const dragDepth = ref(0)
const isGenerating = ref(false)
const copyLabel = ref('Alles kopieren')
const errorMessage = ref('')


const mandantOptions: Ref<Option<number>[]> = ref([
  {label: "spusu AT", value: 1},
  {label: "spusu UK", value: 2},
  {label: "spusu CH", value: 3},
  {label: "spusu IT", value: 4},
  {label: "spusu DE", value: 5},
  {label: "spusu Bikes", value: 6},
  {label: "spusu Wein", value: 7},
])

const mandantInfos = [
  {id: 1, table: "t_Files_102411", languageId: 0},
  {id: 2, table: "t_Files_10248", languageId: -1},
  {id: 3, table: "t_Files_10252", languageId: -1},
  {id: 4, table: "t_Files_102401", languageId: -1},
  {id: 5, table: "t_Files_10250", languageId: -1},
  {id: 6, table: "t_Files_1024138143", languageId: 0},
  {id: 7, table: "t_Files_1024111314", languageId: 0},
]

const selectedMandants = computed(() =>
    mandantInfos.filter(m =>
        selectedOptions.value.includes(m.id)
    )
)

const allMandantsSelected = computed(() =>
    selectedOptions.value.length === mandantOptions.value.length
)

function toggleAllMandants() {
  selectedOptions.value = allMandantsSelected.value
      ? []
      : mandantOptions.value.map(option => option.value)
  errorMessage.value = ''
}

function preventBrowserFileOpen(event: DragEvent) {
  event.preventDefault()
}

onMounted(() => {
  window.addEventListener('dragover', preventBrowserFileOpen)
  window.addEventListener('drop', preventBrowserFileOpen)
})

onUnmounted(() => {
  window.removeEventListener('dragover', preventBrowserFileOpen)
  window.removeEventListener('drop', preventBrowserFileOpen)
})

function addFiles(list: FileList | File[] | null | undefined) {
  if (!list?.length) return

  for (const file of Array.from(list)) {
    const exists = files.value.some(existing =>
        existing.name === file.name &&
        existing.size === file.size &&
        existing.lastModified === file.lastModified
    )
    if (!exists) {
      files.value.push(file)
    }
  }

  errorMessage.value = ''
}

function handleDragEnter(event: DragEvent) {
  event.preventDefault()
  event.stopPropagation()
  dragDepth.value += 1
  isDragging.value = true
}

function handleDragOver(event: DragEvent) {
  event.preventDefault()
  event.stopPropagation()
  if (event.dataTransfer) {
    event.dataTransfer.dropEffect = 'copy'
  }
  isDragging.value = true
}

function handleDragLeave(event: DragEvent) {
  event.preventDefault()
  event.stopPropagation()
  dragDepth.value = Math.max(0, dragDepth.value - 1)
  if (dragDepth.value === 0) {
    isDragging.value = false
  }
}

function handleDrop(event: DragEvent) {
  event.preventDefault()
  event.stopPropagation()
  isDragging.value = false
  dragDepth.value = 0
  addFiles(event.dataTransfer?.files)
}

function openFilePicker() {
  fileInput.value?.click()
}

function handleFileSelect(event: Event) {
  const input = event.target as HTMLInputElement
  addFiles(input.files)
  input.value = ''
}

function removeFile(index: number) {
  files.value.splice(index, 1)
}

function clearFiles() {
  files.value = []
}

function formatSize(bytes: number): string {
  if (bytes < 1024) return `${bytes} B`
  if (bytes < 1024 * 1024) return `${(bytes / 1024).toFixed(1)} KB`
  return `${(bytes / (1024 * 1024)).toFixed(1)} MB`
}

const fileTypeMap: Record<string, number> = {
  png: 1,
  jpg: 2,
  jpeg: 2,
  pdf: 5,
  css: 6,
  js: 21,
  svg: 23,
  webp: 0
}

async function generateSql() {
  if (!selectedMandants.value.length) {
    errorMessage.value = 'Bitte zuerst mindestens einen Mandanten auswählen.'
    return
  }

  if (!files.value.length) {
    errorMessage.value = 'Bitte zuerst Dateien hinzufügen.'
    return
  }

  isGenerating.value = true
  errorMessage.value = ''

  try {
    const statements: string[] = []

    for (const file of files.value) {
      const hex = await readFileAsHex(file)

      for (const mandant of selectedMandants.value) {
        statements.push(
            buildSql(file.name, hex, mandant.table, mandant.languageId)
        )
      }
    }

    sqlStatements.value = statements
  } finally {
    isGenerating.value = false
  }
}

function buildSql(
    fileName: string,
    hex: string,
    table: string,
    languageId: number
): string {
  const ext = fileName.split(".").pop()?.toLowerCase() ?? ""
  const type = fileTypeMap[ext] ?? 0

  return `INSERT INTO \`DB_CMS\`.\`${table}\`
(\`strKey\`, \`strFileName\`, \`nLanguageId\`, \`strContent\`, \`nType\`)
VALUES ('${fileName}', '${fileName}', ${languageId}, 0x${hex}, ${type});`
}

function readFileAsHex(file: File): Promise<string> {
  return new Promise((resolve, reject) => {
    const reader = new FileReader()
    reader.onload = () => {
      const buffer = reader.result as ArrayBuffer
      const hex = [...new Uint8Array(buffer)]
          .map(b => b.toString(16).padStart(2, "0"))
          .join("")
      resolve(hex)
    }
    reader.onerror = reject
    reader.readAsArrayBuffer(file)
  })
}

async function copyAll() {
  try {
    await navigator.clipboard.writeText(sqlStatements.value.join("\n\n"))
    copyLabel.value = 'Kopiert!'
    window.setTimeout(() => {
      copyLabel.value = 'Alles kopieren'
    }, 1800)
  } catch {
    errorMessage.value = 'Kopieren in die Zwischenablage hat nicht geklappt.'
  }
}

function shortenHexInSql(sql: string, visibleBytes = 32): string {
  return sql.replace(
      /0x([0-9a-fA-F]+)/,
      (_, hex: string) => {
        if (hex.length <= visibleBytes * 2) {
          return `0x${hex}`
        }

        const start = hex.slice(0, visibleBytes)
        const end = hex.slice(-visibleBytes)

        return `0x${start}...${end}`
      }
  )
}


const displaySql = computed(() =>
    sqlStatements.value
        .map(sql => shortenHexInSql(sql))
        .join("\n")
)


function resetSql() {
  sqlStatements.value = []
  selectedOptions.value = []
  files.value = []
  errorMessage.value = ''
  copyLabel.value = 'Alles kopieren'
}

</script>


<style scoped>
.window {
  background: #c0c0c0;
  border: 3px solid;
  border-color: #fff #404040 #404040 #fff;
  box-shadow: 8px 8px 0 #000, 0 0 24px #ff2bd6;
}

.titlebar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 6px 8px;
  background: linear-gradient(90deg, #000080, #1084d0 55%, #ff2bd6);
  color: #fff;
  font-size: 12px;
  font-weight: 700;
  letter-spacing: 0.08em;
  text-shadow: 1px 1px 0 #000;
}

.traffic {
  letter-spacing: 0.3em;
}

.page {
  display: flex;
  flex-direction: column;
  gap: 28px;
  padding: 20px;
  background:
      repeating-linear-gradient(
          0deg,
          rgba(255, 255, 255, 0.08) 0 1px,
          transparent 1px 4px
      ),
      linear-gradient(180deg, #e4e0d8, #c8c4bc);
}

h1 {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 10px;
  font-family: Impact, Haettenschweiler, "Arial Black", sans-serif;
  font-size: clamp(32px, 8vw, 52px);
  font-weight: 400;
  letter-spacing: 0.04em;
  text-align: center;
  text-transform: uppercase;
  background: linear-gradient(#fff, #c0c0c0 40%, #808080 70%, #fff);
  -webkit-background-clip: text;
  background-clip: text;
  color: transparent;
  filter: drop-shadow(2px 2px 0 #ff2bd6) drop-shadow(-2px -1px 0 #00f0ff);
}

.sparkle {
  color: #c8ff00;
  filter: none;
  -webkit-text-fill-color: #c8ff00;
  animation: blink 1s steps(2, end) infinite;
}


h2 {
  font-size: 14px;
  font-weight: 700;
  letter-spacing: 0.12em;
  text-transform: uppercase;
  color: #fff;
  background: #000080;
  padding: 4px 8px;
  border: 2px solid #00f0ff;
  box-shadow: 3px 3px 0 #ff2bd6;
}

section {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.section-head {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
}

.text-btn {
  border: 2px solid;
  border-color: #fff #404040 #404040 #fff;
  background: #c0c0c0;
  color: #000;
  cursor: pointer;
  font-size: 12px;
  font-weight: 700;
  padding: 4px 8px;
}

.text-btn:active {
  border-color: #404040 #fff #fff #404040;
}

.error {
  color: #fff;
  background: #ff003c;
  font-size: 13px;
  font-weight: 700;
  padding: 6px 8px;
  border: 2px solid #000;
}

.drop-zone {
  position: relative;
  min-height: 140px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  color: #00f0ff;
  font-weight: 700;
  letter-spacing: 0.08em;
  background:
      repeating-linear-gradient(
          45deg,
          #12002a 0 10px,
          #1c0044 10px 20px
      );
  border: 3px dashed #ff2bd6;
  box-shadow: inset 0 0 0 3px #00f0ff, 4px 4px 0 #000;
}

.drop-zone:hover,
.drop-zone:focus-visible,
.drop-zone.dragging {
  color: #c8ff00;
  border-color: #c8ff00;
  outline: none;
}

.drop-zone > * {
  pointer-events: none;
}

.file-input {
  position: absolute;
  width: 1px;
  height: 1px;
  opacity: 0;
  pointer-events: none;
}

.file-list {
  list-style: none;
  margin: 0;
  padding: 0;
  background: #fff;
  border: 2px solid;
  border-color: #404040 #fff #fff #404040;
}

.file-list li {
  display: grid;
  grid-template-columns: 1fr auto auto;
  gap: 12px;
  align-items: center;
  padding: 8px 10px;
  border-bottom: 1px solid #808080;
}

.file-list li:nth-child(even) {
  background: #e8e0ff;
}

.file-name {
  font-size: 13px;
  word-break: break-all;
}

.file-size {
  font-size: 12px;
  color: #000080;
  font-weight: 700;
}

.icon-btn {
  width: 28px;
  height: 28px;
  border: 2px solid;
  border-color: #fff #404040 #404040 #fff;
  background: #c0c0c0;
  color: #000;
  cursor: pointer;
  font-size: 16px;
  line-height: 1;
}

.icon-btn:hover {
  background: #ff003c;
  color: #fff;
}

.sql-actions {
  display: flex;
  gap: 8px;
}

pre {
  margin: 0;
  max-height: min(420px, 60vh);
  overflow: auto;
  padding: 12px;
  background: #000;
  color: #c8ff00;
  border: 3px solid #00f0ff;
  box-shadow: 4px 4px 0 #ff2bd6;
  font-family: "Courier New", Courier, monospace;
  font-size: 12px;
  line-height: 1.5;
  white-space: pre-wrap;
  word-break: break-all;
}

@keyframes blink {
  50% {
    opacity: 0;
  }
}

@media (max-width: 639px) {
  .page {
    padding: 14px;
    gap: 20px;
  }

  .sql-actions {
    flex-wrap: wrap;
  }

  .sql-actions :deep(.button) {
    width: 100%;
  }
}
</style>
