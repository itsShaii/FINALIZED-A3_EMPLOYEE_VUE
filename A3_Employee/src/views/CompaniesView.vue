<template>
  <NavBar />
  <div class="title">
    SKANLOG EMPLOYEE MANAGEMENT SYSTEM
    <div class="container">
      <el-input
        class="search-input"
        v-model="search"
        placeholder="Search companies..."
        @input="searchQuery"
        clearable
      />
      <el-button class="add-button" type="primary" @click="showAddDialog = true"
        >Add Company</el-button
      >
      <el-table class="company-table" :data="companies" stripe>
        <el-table-column prop="name" label="Company Name">
          <template v-slot:default="{ row }">
            <el-input v-if="editingRow === row.id" v-model="row.editableName" />
            <span v-else>{{ row.name }}</span>
          </template>
        </el-table-column>
        <el-table-column prop="code" label="Company Code">
          <template v-slot:default="{ row }">
            <el-input v-if="editingRow === row.id" v-model="row.editableCode" />
            <span v-else>{{ row.code }}</span>
          </template>
        </el-table-column>
        <el-table-column label="Employees">
          <template v-slot:default="{ row }">
            <el-link type="primary" @click="viewEmployees(row)">View Employees</el-link>
          </template>
        </el-table-column>
        <el-table-column class="actions" label="Actions">
          <template v-slot:default="{ row }">
            <el-button class="edit" v-if="editingRow !== row.id" @click="startEdit(row)"
              >Edit</el-button
            >
            <el-button class="save" v-if="editingRow === row.id" @click="saveEdit(row)"
              >Save</el-button
            >
            <el-button class="cancel" v-if="editingRow === row.id" @click="cancelEdit(row)"
              >Cancel</el-button
            >
            <el-button class="delete" type="danger" @click="confirmDelete(row.id)"
              >Delete</el-button
            >
          </template>
        </el-table-column>
      </el-table>
      <el-pagination
        class="pagination"
        :current-page="pagination.currentPage"
        :page-size="pagination.elementsPerPage"
        :total="pagination.totalElements"
        layout="prev, pager, next"
        @current-change="handlePageChange"
      />
      <el-dialog v-model="showAddDialog" title="Add Company" width="60%">
        <el-form :model="newCompany">
          <el-form-item label="Company Name"><el-input v-model="newCompany.name" /></el-form-item>
          <el-form-item label="Company Code"><el-input v-model="newCompany.code" /></el-form-item>
        </el-form>
        <template v-slot:footer>
          <el-button @click="showAddDialog = false">Cancel</el-button>
          <el-button type="primary" @click="addCompany">Add</el-button>
        </template>
      </el-dialog>
      <el-dialog
        v-model="showEmployeeDialog"
        :title="`Employees of ${selectedCompany?.name || 'Company'}`"
        width="60%"
      >
        <el-table :data="filteredEmployees" stripe>
          <el-table-column prop="name" label="Employee Name" />
          <el-table-column prop="email" label="Email" />
          <el-table-column prop="position" label="Position" />
        </el-table>
        <template v-slot:footer>
          <el-button @click="showEmployeeDialog = false">Close</el-button>
        </template>
      </el-dialog>
    </div>
  </div>
</template>

<script>
import NavBar from '@/components/NavBar.vue'
import axios from 'axios'
import { ElMessageBox, ElMessage } from 'element-plus'

export default {
  name: 'CompaniesPage',
  components: { NavBar },
  data() {
    return {
      companies: [],
      employees: [],
      search: '',
      pagination: { currentPage: 1, elementsPerPage: 10, totalElements: 0 },
      editingRow: null,
      showEmployeeDialog: false,
      showAddDialog: false,
      newCompany: { name: '', code: '' },
      apiBaseUrl: 'https://localhost:5001',
      selectedCompany: null,
    }
  },
  computed: {
    filteredEmployees() {
      return this.employees.filter((emp) => emp.companyId === this.selectedCompany?.id)
    },
  },
  mounted() {
    this.fetchCompanies()
  },
  methods: {
    async fetchCompanies() {
      try {
        const { data } = await axios.get(`${this.apiBaseUrl}/companies`, {
          params: {
            CurrentPage: this.pagination.currentPage,
            ElementsPerPage: this.pagination.elementsPerPage,
            search: this.search,
          },
        })
        this.companies = data.data.results.map((company) => ({
          ...company,
          editableName: company.name,
          editableCode: company.code,
        }))
        this.pagination.totalElements = data.data.totalElements
      } catch {
        ElMessage.error('Failed to load companies.')
      }
    },
    async viewEmployees(company) {
      try {
        this.selectedCompany = company
        const { data } = await axios.get(`${this.apiBaseUrl}/employees`, {
          params: { companyId: company.id },
        })
        this.employees = data.data.results || []
        this.showEmployeeDialog = true
      } catch {
        ElMessage.error('Failed to fetch employees.')
      }
    },
    searchQuery() {
      clearTimeout(this.searchTimeout)
      this.searchTimeout = setTimeout(() => {
        this.pagination.currentPage = 1
        this.fetchCompanies()
      }, 500)
    },
    startEdit(row) {
      this.editingRow = row.id
    },
    async saveEdit(row) {
      try {
        await axios.put(`${this.apiBaseUrl}/companies/${row.id}`, {
          id: row.id,
          name: row.editableName,
          code: row.editableCode,
        })

        row.name = row.editableName
        row.code = row.editableCode
        ElMessage.success('Company updated successfully!')
        this.editingRow = null
      } catch {
        ElMessage.error('Failed to update company.')
      }
    },
    cancelEdit(row) {
      row.editableName = row.name
      row.editableCode = row.code
      this.editingRow = null
    },
    confirmDelete(id) {
      ElMessageBox.confirm('Are you sure you want to delete this company?', 'Warning', {
        type: 'warning',
      })
        .then(() => this.deleteCompany(id))
        .catch(() => {})
    },
    async deleteCompany(id) {
      try {
        await axios.delete(`${this.apiBaseUrl}/companies/${id}`)
        ElMessage.success('Deleted successfully!')
        this.fetchCompanies()
      } catch {
        ElMessage.error('Failed to delete company.')
      }
    },
    async addCompany() {
      try {
        await axios.post(`${this.apiBaseUrl}/companies`, this.newCompany)
        ElMessage.success('Company added successfully!')
        this.showAddDialog = false
        this.newCompany = { name: '', code: '' }
        this.fetchCompanies()
      } catch {
        ElMessage.error('Failed to add company.')
      }
    },
    handlePageChange(newPage) {
      this.pagination.currentPage = newPage
      this.fetchCompanies()
    },
  },
}
</script>

<style scoped>
.container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  width: 130%;
  max-width: 1200px;
  margin-top: 20px;
  margin-left: -340px;
  padding: 50px;
  background-color: white;
  border: 2px solid pink;
  border-radius: 20px;
}

.title {
  margin-top: 20px;
  align-items: center;
  margin-left: 650px;
  font-size: larger;
  color: #ff1493;
  font-weight: bolder;
}

.search-input {
  width: 980px;
  margin-bottom: 20px;
  border: 1px solid pink;
  margin-left: -240px;
  height: 40px;
}

.add-button {
  background-color: #ff66b2;
  color: white;
  margin-top: -60px;
  margin-left: 1000px;
  width: 220px;
  height: 40px;
}

.add-button:hover {
  background-color: rgb(210, 134, 210);
  color: white;
}

.edit {
  background-color: #ff66b2;
  color: white;
  width: 80px;
}

.edit:hover {
  background-color: rgb(204, 41, 111);
  color: white;
}

.save {
  background-color: #ff66b2;
  color: white;
  width: 80px;
}

.save:hover {
  background-color: rgb(204, 41, 111);
  color: white;
}

.cancel {
  background-color: #84bdec;
  color: white;
  width: 80px;
}

.cancel:hover {
  background-color: #175a91;
  color: white;
}

.delete {
  background-color: rgb(149, 28, 158);
  color: white;
  width: 80px;
}

.delete:hover {
  background-color: rgb(210, 134, 210);
  color: white;
}

.company-table {
  width: 100%;
  max-width: 1500px;
  margin-top: 50px;
  margin-bottom: 20px;
  align-items: center;
}

.pagination {
  margin: 20px 0;
  width: 100%;
  max-width: 1500px;
  background-color: lightpink;
}

.el-dialog {
  background-color: #fff0f5;
}
</style>
