<template>
    <section id="user-management">
        <SettingsHeader/>

        <div class="user-list">
            <ul>
                <li v-for="user in users" :key="user.id" class="user-item">
                    <b>{{ user.username }}</b>
                    <span>{{ user.email }}</span>
                    <div class="user-actions">
                        <button class="button button-secondary" @click="selectUser(user)">Edit</button>
                        <button class="button button-disabled" @click="deleteUser(user.id)">Delete</button>
                    </div>
                </li>
            </ul>
        </div>

        <div class="forms-container">
            <div class="add-user-form">
                <h3 v-if="!editMode">Add New User</h3>
                <h3 v-else>Edit User</h3>

                <div class="form-group">
                    <label for="username">Username</label>
                    <input v-model="userForm.username" type="text" id="username" placeholder="Enter username" />
                </div>
                <div class="form-group">
                    <label for="email">Email</label>
                    <input v-model="userForm.email" type="email" id="email" placeholder="Enter email" />
                </div>
                <div class="form-group">
                    <label for="password">Password</label>
                    <input v-model="userForm.password" type="password" id="password" placeholder="Enter password" />
                </div>

                <div class="form-actions">
                    <button class="button button-primary" @click="editMode ? updateUser() : addUser()">
                        {{ editMode ? "Update User" : "Add User" }}
                    </button>
                    <button v-if="editMode" class="button button-secondary" @click="cancelEdit">Cancel</button>
                </div>
            </div>

            <MFASetup />
        </div>
    </section>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import axios from 'axios'
import MFASetup from '../components/MFASetup.vue'
import SettingsHeader from '../components/SettingsHeader.vue'

const users = ref([])
const editMode = ref(false)
const userForm = ref({
    id: null,
    username: '',
    email: '',
    password: ''
})

const fetchUsers = async () => {
    try {
        const response = await axios.get('/api/users')
        users.value = response.data
    } catch (error) {
        console.error("Error fetching users:", error)
    }
}

const addUser = async () => {
    try {
        await axios.post('/api/users', userForm.value)
        fetchUsers()
        resetForm()
    } catch (error) {
        console.error("Error adding user:", error)
    }
}

const selectUser = (user) => {
    userForm.value = { ...user, password: '' }
    editMode.value = true
}

const updateUser = async () => {
    try {
        if (!confirm("Are you sure you want to update this user?")) {
            return
        }

        await axios.put(`/api/users/${userForm.value.id}`, userForm.value)
        fetchUsers()
        resetForm()
    } catch (error) {
        console.error("Error updating user:", error)
    }
}

const deleteUser = async (id) => {
    try {
        if (!confirm("Are you sure you want to delete this user?")) {
            return
        }
        await axios.delete(`/api/users/${id}`)
        fetchUsers()
    } catch (error) {
        console.error("Error deleting user:", error)
    }
}

const resetForm = () => {
    userForm.value = { id: null, username: '', email: '', password: '' }
    editMode.value = false
}

const cancelEdit = () => {
    resetForm()
}

onMounted(() => {
    fetchUsers()
})
</script>