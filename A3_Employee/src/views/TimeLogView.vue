<template>
  <div>
    <NavBar />
    <!-- <el-input v-model="search" placeholder="Search..." @input="debouncedSearchQuery" clearable /> -->
    <el-button type="primary" @click="showAddDialog = true">Add Time Log</el-button>

    <el-table :data="timeLogs" stripe>
      <el-table-column prop="employeeName" label="Employee Name">
        <template v-slot:default="{ row }">
          <span>{{ row.employeeName || 'Unknown' }}</span>
        </template>
      </el-table-column>

      <el-table-column prop="dateTimeInUtc" label="Clock In Time">
        <template v-slot:default="{ row }">
          <span>{{ formatDate(row.dateTimeInUtc) }}</span>
        </template>
      </el-table-column>

      <el-table-column prop="dateTimeOutUtc" label="Clock Out Time">
        <template v-slot:default="{ row }">
          <span>{{ row.dateTimeOutUtc ? formatDate(row.dateTimeOutUtc) : 'Not Clocked Out' }}</span>
        </template>
      </el-table-column>

      <el-table-column label="Actions">
        <template v-slot:default="{ row }">
          <el-button v-if="!row.dateTimeInUtc" @click="clockIn(row)">Time In</el-button>
          <el-button v-if="row.dateTimeInUtc && !row.dateTimeOutUtc" @click="clockOut(row)"
            >Time Out</el-button
          >
        </template>
      </el-table-column>
    </el-table>

    <el-pagination
      :current-page="pagination.currentPage"
      :page-size="pagination.elementsPerPage"
      :total="pagination.totalElements"
      layout="prev, pager, next"
      @current-change="changePage"
    />

    <el-dialog v-model="showAddDialog" title="Add Time Log" width="50%">
      <el-form :model="newTimeLog">
        <el-form-item label="Employee ID">
          <el-input v-model="newTimeLog.employeeId" />
        </el-form-item>
      </el-form>
      <template v-slot:footer>
        <el-button @click="showAddDialog = false">Cancel</el-button>
        <el-button type="primary" @click="addTimeLog">Add</el-button>
      </template>
    </el-dialog>
  </div>
</template>

<script>
import NavBar from '@/components/NavBar.vue'
import { ElMessage } from 'element-plus'
import axios from 'axios'
import debounce from 'lodash/debounce'

export default {
  name: 'TimeLogsPage',
  components: { NavBar },
  data() {
    return {
      timeLogs: [],
      employees: [],
      search: '',
      pagination: { currentPage: 1, elementsPerPage: 10, totalElements: 0 },
      showAddDialog: false,
      newTimeLog: { employeeId: '' },
      apiBaseUrl: 'https://localhost:5001',
    }
  },
  mounted() {
    this.fetchEmployees().then(() => {
      this.fetchTimeLogs()
    })
  },
  methods: {
    async fetchEmployees() {
      try {
        const { data } = await axios.get(`${this.apiBaseUrl}/employees`, {
          params: {
            search: this.search,
          },
        })
        this.employees = data.data.results.map((emp) => ({
          id: emp.id,
          name: emp.name,
        }))
        console.log('Employees:', this.employees)
      } catch (error) {
        console.error('Error fetching employees:', error)
        ElMessage.error('Failed to fetch employees.')
      }
    },

    async fetchTimeLogs() {
      try {
        console.log('Fetching Time Logs with params:', {
          CurrentPage: this.pagination.currentPage,
          ElementsPerPage: this.pagination.elementsPerPage,
          search: this.search,
        })

        const { data } = await axios.get(`${this.apiBaseUrl}/TimeLogs`, {
          params: {
            CurrentPage: this.pagination.currentPage,
            ElementsPerPage: this.pagination.elementsPerPage,
            search: this.search,
          },
        })

        const timeLogs = await Promise.all(
          data.data.results.map(async (log) => {
            const employeeName = await this.getEmployeeNameById(log.employeeId)
            console.log('Log:', log, 'Employee Name:', employeeName)
            return {
              ...log,
              employeeName: log.empployeeName || employeeName,
            }
          }),
        )

        this.timeLogs = timeLogs
        this.pagination.totalElements = data.data.totalElements
        console.log('Time Logs:', this.timeLogs)
      } catch (error) {
        console.error('Error loading time logs:', error)
        ElMessage.error('Failed to load time logs.')
      }
    },

    async getEmployeeNameById(employeeId) {
      try {
        const { data } = await axios.get(`${this.apiBaseUrl}/employees/${employeeId}`)
        console.log('Employee Data:', data)
        return data.name
      } catch (error) {
        console.error(`Error fetching employee name for ID: ${employeeId}`, error)
        return 'Unknown'
      }
    },

    changePage(newPage) {
      this.pagination.currentPage = newPage
      this.fetchTimeLogs()
    },

    searchQuery() {
      console.log('Search Query:', this.search)
      this.pagination.currentPage = 1
      this.fetchTimeLogs()
    },

    debouncedSearchQuery: debounce(function () {
      this.searchQuery()
    }, 400),

    async clockIn(row) {
      const currentTime = new Date().toISOString()
      try {
        const payload = { employeeId: row.employeeId, dateTimeInUtc: currentTime }
        await axios.post(`${this.apiBaseUrl}/TimeLogs`, payload)
        ElMessage.success('Clocked in successfully!')
        this.fetchTimeLogs()
      } catch (error) {
        console.error('Error clocking in:', error)
        ElMessage.error('Failed to clock in.')
      }
    },

    async clockOut(row) {
      const currentTime = new Date().toISOString()
      try {
        const payload = { dateTimeOutUtc: currentTime }
        await axios.put(`${this.apiBaseUrl}/TimeLogs/${row.employeeTimeLogId}`, payload)
        ElMessage.success('Clocked out successfully!')
        this.fetchTimeLogs()
      } catch (error) {
        console.error('Error clocking out:', error)
        ElMessage.error('Failed to clock out.')
      }
    },

    async addTimeLog() {
      const currentTime = new Date().toISOString()

      try {
        const payload = { employeeId: this.newTimeLog.employeeId, dateTimeInUtc: currentTime }
        await axios.post(`${this.apiBaseUrl}/TimeLogs`, payload)
        ElMessage.success('Time log added successfully!')
        this.showAddDialog = false
        this.newTimeLog = { employeeId: '' }
        this.fetchTimeLogs()
      } catch (error) {
        console.error('Error adding time log:', error)
        ElMessage.error('Failed to add time log.')
      }
    },

    formatDate(utcDate) {
      if (!utcDate) return ''
      const date = new Date(utcDate)
      return date.toLocaleString()
    },
  },
}
</script>
