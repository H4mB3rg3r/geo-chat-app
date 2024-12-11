<template>
  <div class="side-panel">
    <!-- Header Section -->
    <section class="side-panel-header">
      <h1>Chat - Geo</h1>
    </section>

    <!-- Channels Section -->
    <section class="channels">
      <div class="section-header">
        <h2>Channels</h2>
        <button @click="editChannel">+</button>
      </div>
      <div class="ul-section">
        <ul>
          <li v-for="(channel, index) in appStore.groupChannels" :key="index"
            :class="{ active: channel.uuid === selectedChannel.uuid }" @click="appStore.setChannel(channel)">
            <div>
              {{ channel.name }}
            </div>
            <button class="edit-button" @click="editChannel(channel)">Edit</button>
          </li>
        </ul>
      </div>
    </section>

    <!-- Friends Section -->
    <section class="friends">
      <div class="section-header">
        <h2>Friends</h2>
        <button @click="editFriend({} as User)">+</button>
      </div>
      <div class="ul-section">
        <ul>
          <li v-for="(friend, index) in appStore.friends" :key="index"
            :class="{ active: friend.uuid === appStore.selectedFriend.uuid }" @click="appStore.setFriend(friend)">
            <div>
              {{ friend.fullname }}
            </div>
            <button class="edit-button" @click="editFriend(friend)">Edit</button>
          </li>
        </ul>
      </div>
    </section>

    <!-- Logout Section -->
    <section class="logout-section">
      <button class="btn btn-secondary" @click="appStore.logoutUser">Logout</button>
      <button class="btn btn-primary" @click="appStore.clickUserBtn">User</button>
    </section>
  </div>

  <!-- Modals -->
  <AddChannelModal :isOpen="showAddChannelModal" :initialChannel="appStore.thisChannel" :friends="appStore.friends"
    @close="showAddChannelModal = false" @add-channel="addChannel" />
  <AddFriendModal :isOpen="showAddFriendModal" :friend="appStore.thisFriend" @close="showAddFriendModal = false"
    @friend-added="addFriend" @unfriended="unfriended" />
</template>

<script setup lang="ts">
  import { computed, ref } from "vue"
  import { useAppStore } from "../stores/app"
  import AddChannelModal from './modal/AddChannelModal.vue' // Adjust path as needed
  import AddFriendModal from "./modal/AddFriendModal.vue"
  import type { Channel, Friend, User } from "../stores/types"
  import { useSeesionStore } from "@/stores/session"
  import { useFriendshipStore } from "@/stores/friendship"
  import { useUserStore } from "@/stores/user"
  import router from "@/router"

  const appStore = useAppStore()
  const sessionStore = useSeesionStore()
  const friendshipStore = useFriendshipStore()
  const userStore = useUserStore()
  const showAddChannelModal = ref(false)
  const showAddFriendModal = ref(false)

  const addChannel = (channel: any) => {
    appStore.addChannel(channel)
  }

  const addFriend = async (email: string) => {
    console.log('Friend email:', email)

    const user1 = userStore.users.find(u => u.email === email)
    if (user1) {
      const newFriendShip = {} as Friend

      newFriendShip.user1_uuid = user1.uuid
      newFriendShip.user2_uuid = appStore.user.uuid
      newFriendShip.created_on = Date.now()

      await friendshipStore.addFriend(newFriendShip)
    }
    // Call backend API to add friend or update friends list
  }

  const unfriended = async (friendEmail: string) => {
    const friendship = friendshipStore.friendships.find((friendship) => {
      // Check if user1 or user2 in the friendship matches the friendEmail
      if (friendship.user1_uuid === appStore.user.uuid && friendship.user2?.email === friendEmail) {
        return true // Match found with user2
      } else if (friendship.user2_uuid === appStore.user.uuid && friendship.user1?.email === friendEmail) {
        return true // Match found with user1
      }
      return false // No match
    })

    if (!friendship) {
      throw new Error(No friendship found for email: ${friendEmail})
    }

    console.log('DELETE FRIENDSHIP ', friendship)
    const _friend = await friendshipStore.deleteFriend(friendship)

    if (_friend) {
      const idx = friendshipStore.friendships.findIndex(f => f.uuid === _friend.uuid)
      if (idx != 1) {
        friendshipStore.friendships.splice(idx, 1)
        appStore.setFriend({} as User)
        appStore.setChannel({} as Channel)
      }
    }

    return friendship // Return the found friendship
  }

  const editChannel = (channel: any) => {
    // Logic for editing a specific channel
    console.log('Edit Channel:', channel)
    appStore.setChannel(channel ?? {} as Channel)
    showAddChannelModal.value = true
  }

  const editFriend = (friend: User) => {
    // Logic for editing a specific friend
    console.log('Edit Friend:', friend)
    appStore.setFriend(friend)
    showAddFriendModal.value = true
  }

  const selectedChannel = computed(() => appStore.selectedChannel)
</script>

<style scoped>
  .side-panel {
    display: flex;
    flex-direction: column;
    width: auto;
    max-width: 350px;
    background: linear-gradient(145deg, #d4d4d4, #e8e8e8); /* Soft gradient background */
    box-shadow: 0px 4px 10px rgba(0, 0, 0, 0.1); /* Subtle shadow for a floating effect */
    padding: 1rem;
    border-right: 1px solid #ccc;
    gap: 1rem;
    justify-content: space-between;
    height: 100vh;
    border-radius: 10px; /* Rounded corners */
  }

  .side-panel-header {
    text-align: center;
    background: linear-gradient(90deg, rgb(85, 105, 95), rgb(125, 140, 130));
    border-radius: 20px;
    padding: 10px;
    box-shadow: 0px 2px 5px rgba(0, 0, 0, 0.1);
  }

  .side-panel-header h1 {
    font-size: 1.6rem;
    color: #f3f2f2;
    font-weight: bold;
    letter-spacing: 1px;
    text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.2); /* Text shadow for better readability */
  }

  .section-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 10px;
  }

  .section-header h2 {
    font-size: 1.2rem;
    color: #333;
    font-weight: bold;
  }

  button {
  width: 40px; /* Set equal width and height */
  height: 40px;
  border-radius: 50%; /* Makes the button a circle */
  background-color: #08b408; /* Default button background */
  color: white; /* Text color */
  border: none; /* Removes border */
  display: flex; /* Centers content */
  align-items: center;
  justify-content: center;
  font-size: 1.2rem; /* Adjust font size */
  cursor: pointer; /* Changes cursor to pointer on hover */
  transition: background-color 0.3s ease; /* Adds smooth hover effect */
}

button:hover {
  background-color: #018d08; /* Darker shade of blue on hover */
}


  .ul-section {
    height: 45vh;
    overflow-y: auto;
    background: rgba(255, 255, 255, 0.5); /* Subtle background for lists */
    border-radius: 8px;
    padding: 5px;
    box-shadow: inset 0px 2px 5px rgba(0, 0, 0, 0.1); /* Inset shadow for depth */
  }

  ul {
    list-style-type: none;
    padding: 0;
  }

  li {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 10px;
    border-bottom: 2px solid #dcdcdc;
    border-radius: 5px;
    background: white;
    cursor: pointer;
    transition: background-color 0.3s ease, transform 0.2s ease;
  }

  li:hover {
    background-color: #f0faff; /* Light hover effect */
    transform: scale(1.02); /* Slight zoom on hover */
  }

  li.active {
    background: rgb(105, 117, 101);
    color: white;
    font-weight: bold;
  }

  .edit-button {
    width: 40px; /* Set equal width and height */
  height: 40px;
  border-radius: 50%; /* Makes the button a circle */
  background-color: #08b408; /* Default button background */
  color: white; /* Text color */
  border: none; /* Removes border */
  display: flex; /* Centers content */
  align-items: center;
  justify-content: center;
  font-size: 15px; /* Adjust font size */
  cursor: pointer; /* Changes cursor to pointer on hover */
  transition: background-color 0.3s ease; 
  }

  .edit-button:hover {
    background-color: #0056b3;
  }

  .logout-section {
    gap: 2rem;
    margin-top: auto;
    text-align: center;
    margin-bottom: 2rem;
    display: flex;
    align-items: center;
    justify-content: space-evenly;
    background: rgba(255, 255, 255, 0.6); /* Subtle semi-transparent background */
    border-radius: 8px;
    padding: 10px;
  }
 

  .btn {
    padding: 8px 12px;
    font-size: 11px;
    font-weight: bold;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    transition: background-color 0.3s ease, color 0.3s ease;
  }

  .btn-primary {
    background-color: #007bff;
    color: white;
  }

  .btn-primary:hover {
    background-color: #0056b3;
  }

  .btn-secondary {
    background-color: #6c757d;
    color: white;
  }

  .btn-secondary:hover {
    background-color: #ec0606;
  }


  .channels, 
.friends {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  flex-grow: 1; /* Allows both sections to grow and share space equally */
}

.channels .ul-section,
.friends .ul-section {
  flex-grow: 1; /* Ensures scrollable sections fill available space */
  max-height: calc(50vh - 3rem); /* Limit height to half of the viewport minus some padding */
  overflow-y: auto; /* Enable scrolling for overflowing content */
  background: rgba(255, 255, 255, 0.5); /* Subtle background for lists */
  border-radius: 8px;
  padding: 5px;
  box-shadow: inset 0px 2px 5px rgba(0, 0, 0, 0.1); /* Inset shadow for depth */
  scrollbar-width: thin; /* Thin scrollbar for Firefox */
  scrollbar-color: #303336 #f0f0f0; /* Custom colors for Firefox */
}

.channels .ul-section::-webkit-scrollbar,
.friends .ul-section::-webkit-scrollbar {
  width: 8px; /* Width of the scrollbar */
}

.channels .ul-section::-webkit-scrollbar-thumb,
.friends .ul-section::-webkit-scrollbar-thumb {
  background: #007bff; /* Scrollbar thumb color */
  border-radius: 8px; /* Rounded edges for the scrollbar thumb */
}

.channels .ul-section::-webkit-scrollbar-thumb:hover,
.friends .ul-section::-webkit-scrollbar-thumb:hover {
  background: #0056b3; /* Darker color on hover */
}

.channels .ul-section::-webkit-scrollbar-track,
.friends .ul-section::-webkit-scrollbar-track {
  background: #f0f0f0; /* Scrollbar track background */
  border-radius: 8px;
}

.channels ul,
.friends ul {
  list-style-type: none;
  padding: 0;
}

.channels li,
.friends li {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px;
  border-bottom: 2px solid #dcdcdc;
  border-radius: 5px;
  background: white;
  cursor: pointer;
  transition: background-color 0.3s ease, transform 0.2s ease;
}

.channels li:hover,
.friends li:hover {
  background-color: #f0faff; /* Light hover effect */
  transform: scale(1.02); /* Slight zoom on hover */
}

.channels li.active,
.friends li.active {
  background: rgb(105, 117, 101); /* Active state background */
  color: white;
  font-weight: bold;
}

.section-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 10px;
}

</style>