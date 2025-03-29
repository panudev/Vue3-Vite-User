<script setup>
import { ref, onMounted, computed } from 'vue'
import { Edit, Trash2 } from 'lucide-vue-next'
import Swal from 'sweetalert2'
import axios from 'axios'

const users = ref([])
const loading = ref(false)
const error = ref(null)
const newUser = ref({ name: '', email: '' })
const currentPage = ref(1)
const pageSize = 5

const addUser = async () => {
    try {
        await axios.post('/api/users', newUser.value)
        await fetchUsers() 
        newUser.value = { name: '', email: '' }
        Swal.fire({
            icon: 'success',
            title: 'เพิ่มผู้ใช้สำเร็จ',
            showConfirmButton: false,
            timer: 1500,
        })
    } catch (err) {
        Swal.fire({
            icon: 'error',
            title: 'เกิดข้อผิดพลาด',
            text: err?.response?.data?.message || err.message || 'ไม่สามารถเพิ่มผู้ใช้ได้',
        })
    }
}

const totalPages = computed(() => Math.ceil(users.value.length / pageSize))

const paginatedUsers = computed(() => {
    const start = (currentPage.value - 1) * pageSize
    return users.value.slice(start, start + pageSize)
})

const fetchUsers = async () => {
    try {
        const res = await axios.get('/api/users')
        users.value = res.data
    } catch (err) {
        error.value = err.message
    } finally {
        loading.value = false
    }
}

const editUser = async (user) => {
    const { value: formValues } = await Swal.fire({
        title: 'แก้ไขผู้ใช้',
        html:
        `<input id="swal-input1" class="swal2-input" placeholder="ชื่อ" value="${user.name}">` +
        `<input id="swal-input2" class="swal2-input" placeholder="อีเมล" value="${user.email}">`,
        focusConfirm: false,
        showCancelButton: true,
        preConfirm: () => {
            return {
                name: document.getElementById('swal-input1').value,
                email: document.getElementById('swal-input2').value
            }
        }
    })
    if (formValues) {
        try {
            await axios.put(`/api/users/${user.id}`, formValues)
            await fetchUsers()
            Swal.fire({
                icon: 'success',
                title: 'แก้ไขผู้ใช้สำเร็จ',
                showConfirmButton: false,
                timer: 1500,
            })
        } catch (err) {
            Swal.fire({
                icon: 'error',
                title: 'เกิดข้อผิดพลาด',
                text: err?.response?.data?.message || err.message || 'ไม่สามารถแก้ไขผู้ใช้ได้',
            })
        }
    }
}

const deleteUser = async (id) => {
    const result = await Swal.fire({
        title: 'ยืนยันการลบผู้ใช้ ?',
        text: 'คุณแน่ใจหรือไม่ว่าต้องการลบผู้ใช้นี้ ?',
        icon: 'warning',
        showCancelButton: true,
        confirmButtonColor: '#d33',
        cancelButtonColor: '#3085d6',
    })
    if (result.isConfirmed) {
        try {
            await axios.delete(`/api/users/${id}`)
            await fetchUsers()
            Swal.fire({
                icon: 'success',
                title: 'ลบสำเร็จ',
                text: 'ผู้ใช้ถูกลบเรียบร้อยแล้ว',
                showConfirmButton: false,
                timer: 1500
            })
        } catch (err) {
            Swal.fire({
                icon: 'error',
                title: 'เกิดข้อผิดพลาด',
                text: err?.response?.data?.message || err.message || 'ไม่สามารถลบผู้ใช้ได้',
            })
        }
    }
}

onMounted(fetchUsers)

</script>

<template>
    <div class="p-6">
      
      <form @submit.prevent="addUser" class="mb-6 bg-white p-4 rounded-xl shadow-md flex flex-col md:flex-row gap-4 items-start md:items-end">
        
        <div class="flex-1 w-full">
            <label class="block text-sm font-medium text-gray-700 mb-1">NAME</label>
            <input
            v-model="newUser.name"
            type="text"
            placeholder="John Doe"
            class="w-full px-4 py-2 border border-gray-300 text-gray-700 rounded focus:outline-none focus:border-blue-500 focus:ring focus:ring-blue-200"
            requir
            />
        </div>
        <div class="flex-1 w-full">
            <label class="block text-sm font-medium text-gray-700 mb-1">EMAIL</label>
            <input
            v-model="newUser.email"
            type="email"
            placeholder="john@example.com"
            class="w-full px-4 py-2 border border-gray-300 text-gray-700 rounded focus:outline-none focus:border-blue-500 focus:ring focus:ring-blue-200"
            required
            />
        </div>

        <button
            type="submit"
            class="bg-black text-white px-6 py-2 rounded-lg hover:bg-gray-800"
        >
            ➕ ADD USER
        </button>
      </form>


      <h2 class="text-3xl font-semibold text-gray-800 mb-6">User List</h2>
      <div v-if="loading" class="text-gray-500 text-center mt-10">Loading...</div>
      <div v-else-if="error" class="text-red-500 text-center mt-10">Error: {{ error }}</div>
  
      <div v-else class="overflow-x-auto shadow-lg rounded-xl border border-gray-200 bg-white">
        <table class="min-w-full table-fixed text-sm text-gray-700">
          <thead class="bg-gray-50 border-b text-gray-600 uppercase tracking-wider">
            <tr>
              <th class="py-3 px-6 text-center w-[40px]">#</th>
              <th class="py-3 px-6 text-center w-[180px]">Name</th>
              <th class="py-3 px-6 text-center w-[240px]">Email</th>
              <th class="py-3 px-6 text-center">Actions</th>
            </tr>
          </thead>
          <tbody>
            <tr
              v-for="(user, index) in paginatedUsers"
              :key="user.id"
              class="hover:bg-gray-100 transition-all"
            >
              <td class="py-3 px-4">{{ index + 1 + (currentPage - 1) * pageSize }}</td>
              <td class="py-3 px-4 w-[240px] truncate overflow-hidden whitespace-nowrap">{{ user.name }}</td>
              <td class="py-3 px-4 w-[240px] truncate overflow-hidden whitespace-nowrap">{{ user.email }}</td>
              <td class="py-3 px-4 space-x-2">
                <button @click="editUser(user)" class="px-3 py-1 bg-yellow-400 text-white rounded hover:bg-yellow-500">
                    <Edit class="w-5 h-5 text-blue-500 hover:text-blue-700" />
                </button>
                <button @click="deleteUser(user.id)" class="px-3 py-1 bg-red-500 text-white rounded hover:bg-red-600">
                    <Trash2 class="w-5 h-5 text-blue-500 hover:text-blue-700" />
                </button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
  
      <div class="mt-6 flex justify-center items-center gap-2 flex-wrap">
        <button
            v-for="page in totalPages"
            :key="page"
            @click="currentPage = page"
            :class="[
            'px-3 py-1 border rounded',
            page === currentPage ? 'bg-blue-500 text-white' : 'bg-white text-gray-700 hover:bg-gray-100']"
        >
            {{ page }}
        </button>
        </div>
    </div>
</template>
