<script setup>
import AppHeader from './components/AppHeader.vue'
import AttendanceForm from './components/AttendanceForm.vue'
import AttendanceList from './components/AttendanceList.vue'
import AppFooter from './components/AppFooter.vue'
import { ref, onMounted } from 'vue'

// ========================================
// RECORD STATE
// ========================================

const records = ref([])

const editingRecord = ref(null)

const showDeleteModal = ref(false)

const recordToDelete = ref(null)


// ========================================
// LOAD FROM LOCALSTORAGE
// ========================================

onMounted(() => {

  const savedRecords =
    localStorage.getItem('attendance-records')

  if (savedRecords) {

    try {

      records.value =
        JSON.parse(savedRecords)

    } catch (error) {

      console.error(
        'Unable to load attendance records:',
        error
      )

      records.value = []

    }

  }

})


// ========================================
// SAVE TO LOCALSTORAGE
// ========================================

function saveRecords() {

  localStorage.setItem(
    'attendance-records',
    JSON.stringify(records.value)
  )

}


// ========================================
// CREATE
// ========================================

function addRecord(record) {

  const newRecord = {
    id: Date.now(),
    ...record
  }

  records.value.push(newRecord)

  saveRecords()

}


// ========================================
// EDIT
// ========================================

function editRecord(record) {

  editingRecord.value = {
    ...record
  }

}


// ========================================
// UPDATE
// ========================================

function updateRecord(updatedRecord) {

  const index =
    records.value.findIndex(
      record =>
        record.id === updatedRecord.id
    )

  if (index !== -1) {

    records.value[index] = {
      ...updatedRecord
    }

  }

  editingRecord.value = null

  saveRecords()

}


// ========================================
// DELETE
// ========================================

function deleteRecord(id) {

  const record =
    records.value.find(
      record => record.id === id
    )

  if (!record) {
    return
  }

  recordToDelete.value = record

  showDeleteModal.value = true

}


// ========================================
// CONFIRM DELETE
// ========================================

function confirmDelete() {

  if (!recordToDelete.value) {
    return
  }

  const id =
    recordToDelete.value.id

  records.value =
    records.value.filter(
      record => record.id !== id
    )

  editingRecord.value = null

  saveRecords()

  showDeleteModal.value = false

  recordToDelete.value = null

}

</script>