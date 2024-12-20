<template>
  <div class="form-wrapper">
    <div :class="['form-wrapper', { 'register-user': !hasUser, 'edit-user': hasUser }]">
      <div class="user-form">
        <h2 class="modal-title">{{ hasUser ? "Edit User" : "Register User" }}</h2>

        <form @submit.prevent="handleSubmit">
          <div class="form-body">
            <div class="user-image">
              <img
                v-if="user.image_url"
                :src="user.image_url"
                alt="User Image"
                class="image-circle"
              />
              <span v-else class="image-circle">
                {{ user.fullname.charAt(0).toUpperCase() }}
              </span>
              <input
                type="file"
                @change="handleImageUpload"
                accept="image/*"
                class="image-upload-btn"
              />
            </div>

            <div class="form-fields">
              <div class="form-group">
                <label class="input-label" for="fullname">Full Name</label>
                <input
                  class="input-text-modal"
                  id="fullname"
                  type="text"
                  v-model="user.fullname"
                  placeholder="Enter full name"
                  required
                />
              </div>

              <div class="form-group">
                <label class="input-label" for="username">Username</label>
                <input
                  class="input-text-modal"
                  id="username"
                  type="text"
                  v-model="user.username"
                  placeholder="Enter username"
                  required
                />
              </div>

              <div class="form-group">
                <label class="input-label" for="email">Email</label>
                <input
                  class="input-text-modal"
                  id="email"
                  type="email"
                  v-model="user.email"
                  placeholder="Enter email"
                  required
                />
              </div>

              <div v-if="!hasUser" class="form-group">
                <label class="input-label" for="password">Password</label>
                <input
                  class="input-text-modal"
                  id="password"
                  type="password"
                  v-model="user.password"
                  placeholder="Enter password"
                  required
                />
              </div>

              <div v-if="!hasUser" class="form-group">
                <label class="input-label" for="confirm-password">Confirm Password</label>
                <input
                  class="input-text-modal"
                  id="confirm-password"
                  type="password"
                  v-model="confirmPassword"
                  placeholder="Confirm password"
                  required
                />
              </div>
            </div>
          </div>
          <div class="success-notification">
            <span v-if="showNotification" >{{ notificationMessage }}</span>
          </div>
          
          <div class="form-actions">
            <button type="submit" class="btn btn-primary">
              {{ hasUser ? "Update User" : "Register User" }}
            </button>
            <button type="button" class="btn btn-secondary" @click="cancel">
              Back
            </button>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>

  
  

<script setup lang="ts">
import { ref, watch, computed, onMounted } from 'vue';
import type { Channel, User } from '../../stores/types'; // Adjust path to your User interface
import { useRoute, useRouter } from 'vue-router';
import { useAppStore } from '../../stores/app';
import { useUserStore } from '@/stores/user';
import { useHelperStore } from '@/stores/helper';

// Props to receive user data (optional for edit mode)
const props = defineProps<{
    initialUser?: User;
}>();

const router = useRouter();
const route = useRoute();
const appStore = useAppStore();
const userStore = useUserStore();
const helperStore = useHelperStore();

const confirmPassword = ref('');
const notificationMessage = ref(''); // Add notification state
const showNotification = ref(false); // Control the display of the notification

// Emit events to notify parent component of actions
const emit = defineEmits(['submit', 'cancel']);

const isEditMode = ref(!!props.initialUser); // Determine if it's edit mode or registration




// Initialize user state
const user = ref<User>({
    uuid: props.initialUser?.uuid || '',
    fullname: props.initialUser?.fullname || '',
    username: props.initialUser?.username || '',
    email: props.initialUser?.email || '',
    password: '',
    created_on: props.initialUser?.created_on || Date.now(),
    image_url: props.initialUser?.image_url || ''
});

const isPasswordMatched = computed(() => {
    return confirmPassword.value === user.value.password && user.value.password.length > 1;
});

// Watch for changes in the props.initialUser prop to update the form (for editing)
watch(
    () => props.initialUser,
    (newUser) => {
        if (newUser) {
            user.value = { ...newUser, password: '' }; // Clear password field on edit
            isEditMode.value = true;
        }
        console.log('initial User', props.initialUser);
    }
);

// Handle form submission
const handleSubmit = async () => {
    console.log('Register User :: ', user.value, route.fullPath);
    if (route.fullPath.includes('register')) {
        // Register user
        await userStore.registerUser(user.value);
        router.push({ name: 'login' });
    } else {
        // Update User
        appStore.user = (await userStore.updateUser(user.value)) ?? user.value;
        emit('submit', { ...user.value }); // Send user data to parent component

        // Show notification
        notificationMessage.value = 'User updated successfully!';
        showNotification.value = true;

        // Hide notification after 3 seconds
        setTimeout(() => {
            showNotification.value = false;
        }, 3000);
    }
};

// Handle cancel action
const cancel = () => {
    if (route.fullPath.includes('register')) {
        router.push('/');
    } else {
        appStore.setChannel({} as Channel);
        appStore.setFriend({} as User);
        router.push({ name: 'home' });
    }
};

const hasUser = computed(() => {
    return !!user.value.uuid;
});

const handleImageUpload = async (event: any) => {
    const fileInput = event.target as HTMLInputElement;
    if (!fileInput.files || fileInput.files.length === 0) return;

    const file = fileInput.files[0];
    // Convert to Base64
    try {
        // const base64String = await convertToBase64(file)
        const filePath = await helperStore.uploadFileViaFormData(file);

        // Save to database (replace this with your actual save logic)
        console.log('filePath String:', filePath);

        // For demonstration, render the Base64 image immediately
        if (filePath) {
            user.value.image_url = filePath;
        }
    } catch (error) {
        console.error('Error converting file to Base64:', error);
    }
};

onMounted(() => {
    if (appStore.user.uuid) {
        user.value = appStore.user;
    }
});
</script>

<style scoped>
.form-wrapper {
  display: flex;
    justify-content: center;
    align-items: center;
}
  .user-form {
    max-width: 400px;
    margin: auto;
    padding: 2rem;
    background-color: white;
    border-radius: 8px;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
  }

  .form-group {
    margin-bottom: 1rem;
  }

  .form-actions {
    display: flex;
    justify-content: space-between;
  }

  .btn {
    padding: 0.5rem 1rem;
    cursor: pointer;
  }

  .btn-primary {
    background-color: #007bff;
    color: white;
    border: none;
  }

  .btn-secondary {
    background-color: #6c757d;
    color: white;
    border: none;
  }

  /* .image-circle {
        width: 10rem;
        height: 10rem;
        border-radius: 50%;
        background-color: #f0f0f0;
        border: 2px solid blue;
        display: flex;
        align-items: center;
        justify-content: center;
        font-size: 5rem;
        font-weight: bold;
        color: blue;
    }

    .user-image {
        display: flex;
        justify-content: center;
        margin: 1rem 0rem 1rem 0rem;
    } */

  .user-image {
    position: relative;
    display: inline-block;
    margin: auto 6rem;
  }

  .image-circle {
    width: 10rem;
    height: 10rem;
    border-radius: 50%;
    background-color: #f0f0f0;
    border: 2px solid blue;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 5rem;
    font-weight: bold;
    color: blue;
  }

  .image-upload-btn {
    position: absolute;
    bottom: 0px;
    right: 2rem;
    width: 6rem;
    height: 2rem;
    opacity: 0;
    cursor: pointer;
    top: 5rem;
  }

  .image-upload-btn:hover {
    opacity: 0.7;
  }

  .btn.disabled,
  .btn:disabled {
    background-color: #ccc;
    cursor: not-allowed;
    color: #6c757d;
  }

   /* Responsive styles for smaller screens */
  @media (max-width: 600px) {
    .user-form {
      padding: 1rem;
    }

    .form-actions {
      flex-direction: column;
      gap: 1rem;
    }

    .btn {
      width: 100%;
    }

    .user-image {
      margin: auto;
    }

    .image-circle {
      width: 8rem;
      height: 8rem;
      font-size: 3rem;
    }

    .image-upload-btn {
      width: 4rem;
      height: 2rem;
      top: 4rem;
      right: 1rem;
    }
  }

  .success-notification {
    margin: 1rem;
    text-align: center;
    background-color: #90EE90;
  }
</style>