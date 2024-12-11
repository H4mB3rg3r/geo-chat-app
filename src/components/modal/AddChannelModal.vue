<template>
  <div class="modal-backdrop" v-if="isOpen">
    <div class="modal">
      <h2>{{ isEditMode ? "Edit Channel" : "Add New Channel" }}</h2>
      <form @submit.prevent="handleSubmit">
        <div class="form-group">
          <label for="name">Channel Name</label>
          <input class="input-text-modal" v-model="channel.name" id="name" type="text" placeholder="Enter channel name"
            required />
        </div>

        <div class="form-group">
          <label>Select Users</label>
          <div class="checkbox-group">
            <div v-for="friend in friends" :key="friend.uuid" class="checkbox-item">
              <input type="checkbox" :id="`user-${friend.uuid}`" :value="friend.uuid" v-model="channel.user_uuids" />
              <label :for="`user-${friend.uuid}`">{{ friend.fullname }}</label>
            </div>
          </div>
        </div>

        <div class="form-actions">
          <button type="submit" class="btn btn-primary">
            {{ isEditMode ? "Save Changes" : "Add Channel" }}
          </button>
          <button type="button" class="btn btn-secondary" @click="closeModal">
            Cancel
          </button>
          <button v-if="isEditMode" type="button" class="btn btn-danger" @click="handleDelete">
            Delete Channel
          </button>
        </div>
      </form>
    </div>
  </div>
</template>

<script setup lang="ts">
  import { ref, computed, onMounted, watch } from 'vue'
  import { CHANNEL_TYPE, type User, type Channel } from '../../stores/types'
  import { useSeesionStore } from '@/stores/session'
  import { useChannelStore } from '@/stores/channel'

  const sessionStore = useSeesionStore()
  const channelStore = useChannelStore()

  // Props from parent
  const props = defineProps<{
    isOpen: boolean
    friends: User[]
    initialChannel: Channel | null
  }>()

  // Emit events to parent
  const emit = defineEmits(['close', 'add-channel', 'delete-channel'])

  const initialChannel = ref({} as Channel)

  // Check if editing mode or creating a new channel
  const isEditMode = computed(() => !!initialChannel.value.uuid)

  // Initialize channel state based on whether it's a new or existing channel
  const channel = ref<Channel>(
    initialChannel.value
      ? { ...initialChannel.value }
      : {
        uuid: '',
        name: '',
        user_uuids: [],
        type: CHANNEL_TYPE.GROUP,
        created_on: new Date().toISOString(),
      }
  )

  watch(() => props.isOpen, (value, _) => {
    console.log('@_____ SHOW CHANNEL MODAL', props)
    if (value) {
      initialChannel.value = props.initialChannel ?? {} as Channel
      channel.value = initialChannel.value.uuid
        ? { ...initialChannel.value }
        : {
          uuid: '',
          name: '',
          user_uuids: [],
          type: CHANNEL_TYPE.GROUP,
          created_on: new Date().toISOString(),
        }
    }
  })

  // Handle form submission
  const handleSubmit = () => {
    const user = sessionStore.session?.user
    if (user) {
      if (!channel.value.user_uuids.includes(user?.uuid)) {
        channel.value.user_uuids.push(user?.uuid)
      }
    }
    if (!channel.value.user_uuids) return
    channel.value.type = CHANNEL_TYPE.GROUP
    emit('add-channel', { ...channel.value })
    closeModal()
  }

  // Handle deleting channel
  const handleDelete = async () => {
    if (confirm('Are you sure you want to delete this channel?')) {
      const _channel = await channelStore.deleteChannel({ uuid: channel.value.uuid } as Channel)
      if (_channel) {
        const idx = channelStore.channels.findIndex((channel) => channel.uuid == _channel.uuid)
        if (idx != -1) {
          channelStore.channels.splice(idx, 1)
          closeModal()
        }
      }
    }
  }

  // Close modal
  const closeModal = () => {
    emit('close')
  }
</script>

<style scoped>
   .modal-backdrop {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.6); /* Darker backdrop for better contrast */
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 999;
  }

.modal {
  background: linear-gradient(135deg, #ffffff, #f0f5ff); /* Light gradient background */
  padding: 2rem;
  border-radius: 16px;
  width: 400px;
  max-width: 100%;
  box-shadow: 0 8px 20px rgba(0, 0, 0, 0.2); /* Enhanced shadow for depth */
  border: 2px solid #007bff;
  animation: slideIn 0.3s ease-out;
  }

  @keyframes slideIn {
  from {
    transform: translateY(-20px);
    opacity: 0;
  }
  to {
    transform: translateY(0);
    opacity: 1;
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

  .input-text-modal:focus {
  border-color: #007bff;
  outline: none;
  }

  .checkbox-group {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
  height: 150px;
  overflow-y: auto;
  padding: 0.5rem;
  background: #f9f9f9;
  border-radius: 8px;
  border: 1px solid #ddd;
}

  .checkbox-item {
    display: flex;
    align-items: center;
  }

  .checkbox-item input {
    margin-right: 0.5rem;
  }

  .form-actions {
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

</style>