<template>
  <div v-if="isOpen" class="modal-overlay" @click="closeModal">
    <div class="modal-content" @click.stop>
      <h2>{{ isEditMode ? "Edit Friend" : "Add Friend" }}</h2>
      <form @submit.prevent="handleAddFriend">
        <div class="form-group">
          <label for="friendEmail">Friend's Email</label>
          <input class="input-text-modal" id="friendEmail" type="email" v-model="friendEmail"
            placeholder="Enter friend's email" required :disabled="isEditMode" />
            <span v-if="!isEmailAddressExists && friendEmail" class="error-message">User's email address not found.</span>
        </div>
        <div v-if="isEditMode" class="form-group">
          <label for="friendEmail">Friend's fullname</label>
          <input class="input-text-modal" id="friendEmail" type="email" v-model="friend.fullname"
            placeholder="Enter friend's email" required :disabled="isEditMode" />
        </div>

        <div class="modal-actions">
          <button type="submit" class="btn btn-primary" :class="{ 'btn-danger': isEditMode }"
            :disabled="!isEmailAddressExists">{{ isEditMode ?
              "Unfriend" : "Add Friend" }}</button>
          <button type="button" class="btn btn-secondary" @click="closeModal">
            Cancel
          </button>
        </div>
      </form>
    </div>
  </div>
</template>

<script setup lang="ts">
  import { ref, defineEmits, watch, computed, onMounted } from 'vue'
  import { type User } from '../../stores/types'
  import { useUserStore } from '@/stores/user'

  // Props
  const props = defineProps<{
    isOpen: boolean
    friend: User
  }>()

  // Emit events
  const emit = defineEmits(['close', 'friend-added', 'unfriended'])

  const friendEmail = ref()
  const isEditMode = computed(() => !!props.friend.uuid)
  const userStore = useUserStore()

  // Close modal
  const closeModal = () => {
    emit('close')
  }

  const isEmailAddressExists = computed(() => {
    return userStore.users.some(user => user.email === friendEmail.value)
  })

  // Handle adding a friend
  const handleAddFriend = () => {
    if (friendEmail.value) {
      if (!isEditMode.value) {
        emit('friend-added', friendEmail.value) // Send the friend's email to parent component
      }
      else {
        emit('unfriended', friendEmail.value) // Send the friend's email to parent component
      }
      friendEmail.value = '' // Clear input
      closeModal()
    }
  }

  watch(() => props.friend, (value, _) => {
    friendEmail.value = value.email ?? ""
  })

  onMounted(async () => {
    userStore.users = await userStore.getUsers() ?? [] as User[]
  })
</script>

<style scoped>
  .error-message {
    color: red;
    font-size: 0.875em;
    margin-top: 0.25em;
    display: block;
  }

  .modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background: rgba(0, 0, 0, 0.6); /* Darker backdrop for better contrast */
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
  }

  .modal-content {
  background: linear-gradient(135deg, #ffffff, #f0f5ff); /* Light gradient background */
  padding: 2rem;
  border-radius: 16px;
  max-width: 400px;
  width: 100%;
  box-shadow: 0 8px 20px rgba(0, 0, 0, 0.2); /* Enhanced shadow for depth */
  border: 2px solid #007bff;
  animation: fadeIn 0.3s ease-out;
  }
  @keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(-20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
    }
  }
  h2 {
  text-align: center;
  font-size: 1.5rem;
  color: #003366; /* Navy blue for contrast */
  margin-bottom: 1rem;
  }


  .form-group {
    margin-bottom: 1.5rem;
  }
  .form-group label {
  font-weight: bold;
  color: #333333;
  margin-bottom: 0.5rem;
  display: block;
  }
  .input-text-modal {
  width: 93%;
  padding: 0.8rem;
  font-size: 1rem;
  border-radius: 8px;
  border: 1px solid #ddd;
  transition: border-color 0.3s;
}
.input-text-modal:focus {
  border-color: #007bff;
  outline: none;
}

.modal-actions {
  display: flex;
  justify-content: space-between;
  gap: 1rem;
  margin-top: 1.5rem;
}
.btn {
  padding: 0.6rem 1.2rem;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-size: 1rem;
  transition: background-color 0.3s ease, color 0.3s ease;
}
.btn-primary {
  background: linear-gradient(135deg, #007bff, #0056b3); /* Vibrant gradient */
  color: white;
}

.btn-primary:hover {
  background: linear-gradient(135deg, #0056b3, #003d80); /* Darker gradient on hover */
}

.btn-secondary {
  background: linear-gradient(135deg, #6c757d, #495057); /* Neutral gradient */
  color: white;
}

.btn-secondary:hover {
  background: linear-gradient(135deg, #495057, #343a40); /* Darker neutral gradient */
}

.btn-danger {
  background: linear-gradient(135deg, #dc3545, #a71d2b); /* Gradient red */
  color: white;
}

.btn-danger:hover {
  background: linear-gradient(135deg, #a71d2b, #800f1a); /* Darker red gradient */
}

  .btn-danger {
    background-color: #dc3545;
    color: white;
  }

  .btn.disabled,
  .btn:disabled {
    background-color: #ccc;
    cursor: not-allowed;
    color: #e9eef1;
  }
</style>