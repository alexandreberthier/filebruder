<template>
  <div class="page">
    <h1>File Bruder™</h1>
    <DropDownCheckbox
        :options="mandantOptions"
        v-model:selected-options="selectedOptions"
    />
    <div
        class="drop-area"
        @drop.prevent="handleDrop"
        @dragover.prevent
    >
      <p>Dateien hier ablegen</p>
    </div>

    <div v-if="files.length" class="files">
      <p v-for="file in files" :key="file.name">{{ file.name }}</p>

      <DynamicButton @click="generateSql" text=" SQL generieren"/>
    </div>

    <div v-if="sqlStatements.length" class="sql">
      <div class="sql-header">
        <h3>SQL Inserts ({{ sqlStatements.length }})</h3>

        <div class="sql-actions">
          <DynamicButton @click="copyAll" text="Alles kopieren"/>
          <DynamicButton :button-type="ButtonType.Secondary" @click="resetSql" text="Reset"/>
        </div>
      </div>
      <div class="sql-container">
        <pre>{{ displaySql }}</pre>
      </div>
    </div>
  </div>
</template>


<script setup lang="ts">
import DropDownCheckbox, {type Option} from "@/components/DropDownCheckbox.vue"
import {computed, ref, type Ref} from "vue"
import DynamicButton from "@/components/DynamicButton.vue";
import {ButtonType} from "@/helpers/ButtonType.ts";

const selectedOptions: Ref<number[]> = ref([])
const files = ref<File[]>([])
const sqlStatements = ref<string[]>([])


const mandantOptions: Ref<Option<number>[]> = ref([
  {label: "spusu AT", value: 1},
  {label: "spusu UK", value: 2},
  {label: "spusu CH", value: 3},
  {label: "spusu IT", value: 4},
  {label: "spusu DE", value: 5},
])

const mandantInfos = [
  {id: 1, table: "t_Files_102411", languageId: 0},
  {id: 2, table: "t_Files_10248", languageId: -1},
  {id: 3, table: "t_Files_10252", languageId: -1},
  {id: 4, table: "t_Files_102401", languageId: -1},
  {id: 5, table: "t_Files_10250", languageId: -1},
]

const selectedMandants = computed(() =>
    mandantInfos.filter(m =>
        selectedOptions.value.includes(m.id)
    )
)

/* =====================
   Drag & Drop
===================== */

function handleDrop(event: DragEvent) {
  const dropped = event.dataTransfer?.files
  if (!dropped) return

  for (const file of dropped) {
    files.value.push(file)
  }
}

const fileTypeMap: Record<string, number> = {
  png: 1,
  jpg: 2,
  jpeg: 2,
  pdf: 5,
  css: 6,
  js: 21,
  svg: 23,
}

async function generateSql() {
  if (!selectedMandants.value.length) {
    alert("Kein Mandant kein SQL mein bV")
    return
  }

  sqlStatements.value = []

  for (const file of files.value) {
    const hex = await readFileAsHex(file)

    for (const mandant of selectedMandants.value) {
      sqlStatements.value.push(
          buildSql(file.name, hex, mandant.table, mandant.languageId)
      )
    }
  }

  files.value = []
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

/* =====================
   Helpers
===================== */

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
  await navigator.clipboard.writeText(sqlStatements.value.join("\n\n"))
  alert("SQL kopiert")
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
}

</script>


<style scoped>
.page {
  max-width: 800px;
  margin: 0 auto;
  display: flex;
  flex-direction: column;
  gap: 24px;
}

.drop-area {
  border: 2px dashed #000;
  padding: 40px;
  text-align: center;
  cursor: pointer;
  min-height: 200px;
  display: flex;
  justify-content: center;
  align-items: center;
}

.files, .sql {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

pre {
  background: #f5f5f5;
  padding: 12px;
  white-space: pre-wrap;
}

.sql {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.sql-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.sql-actions {
  display: flex;
  gap: 8px;
}

.sql-container {
  max-height: 400px;
  overflow: auto;
  border: 1px solid #ccc;
  background: #f7f7f7;
  padding: 12px;
}

.sql-container pre {
  margin: 0;
  font-family: monospace;
  font-size: 13px;
  line-height: 1.4;
  white-space: pre-wrap;
  word-break: break-all;
}

</style>
