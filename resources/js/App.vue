<template>
    <div class="flex flex-col h-screen bg-gray-50">
        <!-- Header Component -->
        <Header
            :user="currentUser"
            :notifications="notifications"
            @toggle-notifications="toggleNotifications"
            @search="searchGlobal"
            class="shadow-sm z-10"
        />

        <!-- Main Content -->
        <div class="flex flex-1 overflow-hidden">
            <!-- Sidebar Component - smaller on larger screens -->
            <Sidebar
                :contacts="filteredContacts"
                :selectedContactId="selectedContactId"
                @select-contact="selectContact"
                class="w-1/4 md:w-1/5 lg:w-1/6 border-r bg-white overflow-y-auto"
            />

            <!-- Chat Window Component - larger on larger screens -->
            <div class="flex flex-col w-3/4 md:w-4/5 lg:w-5/6">
                <ChatWindow
                    v-if="selectedContactId"
                    :messages="messages"
                    :currentUser="currentUser"
                    :selectedContact="getSelectedContact()"
                    class="flex-1 overflow-hidden"
                    id="messagelist"
                />
                <div v-else class="flex-1 flex items-center justify-center bg-gray-50">
                    <p class="text-gray-500">Select a contact to start chatting</p>
                </div>

                <!-- Message Input Component -->
                <MessageInput
                    v-if="selectedContactId"
                    v-model="newMessage"
                    @send="sendMessage"
                    class="border-t"
                />
            </div>
        </div>

        <!-- Notification Modal (Optional) -->
        <NotificationModal
            v-if="showNotifications"
            :notifications="notifications"
            @close="showNotifications = false"
            @mark-read="markNotificationAsRead"
            @mark-all-read="markAllNotificationsAsRead"
        />
    </div>
</template>

<script>
import axios from 'axios';
import { ref, computed, onMounted, nextTick } from 'vue';
import Header from './components/Header.vue';
import Sidebar from './components/Sidebar.vue';
import ChatWindow from './components/ChatWindow.vue';
import MessageInput from './components/MessageInput.vue';
import NotificationModal from './components/NotificationModal.vue';

export default {
    components: {
        Header,
        Sidebar,
        ChatWindow,
        MessageInput,
        NotificationModal
    },
    setup() {
        // Current user (would be fetched from API in a real app)
        const currentUser = ref({
            id: 1,
            name: 'Dea Novita',
            role: 'Admin',
            avatar: '/images/default-avatar.png'
        });

        // Messages data
        const messages = ref([]);
        const newMessage = ref('');

        // Sample contacts data (would be fetched from API in a real app)
        const contacts = ref([
            {
                id: 1,
                name: 'Arya Wibawa',
                lastMessage: 'Yes, sure! I will fill it out now.',
                timestamp: '10:20',
                avatar: '/images/default-avatar.png',
                unread: false
            },
            {
                id: 2,
                name: 'Vita Darmawan',
                lastMessage: 'Yes, sure! I will fill it out now.',
                timestamp: '10:18',
                avatar: '/images/default-avatar.png',
                unread: true
            },
            {
                id: 3,
                name: 'Purnami Aksa',
                lastMessage: 'Yes, sure! I will fill it out now.',
                timestamp: '10:17',
                avatar: '/images/default-avatar.png',
                unread: false
            },
            {
                id: 4,
                name: 'Angel Mawar',
                lastMessage: 'Yes, sure! I will fill it out now.',
                timestamp: '10:15',
                avatar: '/images/default-avatar.png',
                unread: true
            },
            {
                id: 5,
                name: 'Lily Indrawan',
                lastMessage: 'Yes, sure! I will fill it out now.',
                timestamp: '10:12',
                avatar: '/images/default-avatar.png',
                unread: false
            }
        ]);

        // Search query for filtering contacts
        const searchQuery = ref('');

        // Filtered contacts based on search query
        const filteredContacts = computed(() => {
            if (!searchQuery.value) return contacts.value;

            const query = searchQuery.value.toLowerCase();
            return contacts.value.filter(contact =>
                contact.name.toLowerCase().includes(query) ||
                contact.lastMessage.toLowerCase().includes(query)
            );
        });

        // Notifications
        const notifications = ref([
            {
                id: 1,
                title: 'New message',
                text: 'You have received a new message from Arya Wibawa',
                timestamp: '10:24',
                read: false
            },
            {
                id: 2,
                title: 'Appointment reminder',
                text: 'Your next appointment is scheduled for tomorrow at 2 PM',
                timestamp: '09:15',
                read: true
            }
        ]);

        // Track selected contact
        const selectedContactId = ref(1); // Default to first contact

        // Show/hide notifications modal
        const showNotifications = ref(false);

        // Methods
        const selectContact = (contactId) => {
            selectedContactId.value = contactId;

            // Mark as read (in real app, send API request)
            const contact = contacts.value.find(c => c.id === contactId);
            if (contact) contact.unread = false;

            // Get messages for selected contact
            getMessages();
        };

        const getSelectedContact = () => {
            return contacts.value.find(c => c.id === selectedContactId.value) || null;
        };

        const searchGlobal = (query) => {
            searchQuery.value = query;
            // In a real app, you might want to perform a more comprehensive search
        };

        // Integrate with your existing message functions
        const getMessages = async () => {
            try {
                // In a real app, you'd include the selected contact ID in the request
                // const response = await axios.get(`/messages/${selectedContactId.value}`);
                // messages.value = response.data;

                // For demo purposes, we'll use sample data
                messages.value = [
                    {
                        id: 1,
                        user: { id: 1, name: 'Dea Novita' },
                        text: 'Hi Arya Wibawa,\n\nWe hope your recent visit to our clinic was pleasant. We\'d love to hear your feedback.',
                        time: '10:20'
                    },
                    {
                        id: 2,
                        user: { id: 1, name: 'Dea Novita' },
                        text: 'Please take a moment to share your thoughts with us here. Your feedback helps us improve our services. Thank you! 😊',
                        time: '10:20'
                    },
                    {
                        id: 3,
                        user: { id: 2, name: 'Arya Wibawa' },
                        text: 'Yes, sure! I will fill it out now.',
                        time: '10:24'
                    }
                ];

                // Scroll to bottom after messages are loaded
                scrollToBottom();
            } catch (err) {
                console.error('Error fetching messages:', err.message);
            }
        };

        const sendMessage = async () => {
            if (newMessage.value.trim() === '') return;

            try {
                // In a real app, you'd send the message to the server
                // await axios.post('/message', {
                //   recipient_id: selectedContactId.value,
                //   text: newMessage.value.trim()
                // });

                // For demo purposes, we'll add the message locally
                const message = {
                    id: Date.now(),
                    user: { id: currentUser.value.id, name: currentUser.value.name },
                    text: newMessage.value.trim(),
                    time: new Date().toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' })
                };

                messages.value.push(message);

                // Update the last message in contacts
                const contact = contacts.value.find(c => c.id === selectedContactId.value);
                if (contact) {
                    contact.lastMessage = newMessage.value.trim();
                    contact.timestamp = message.time;
                }

                // Clear input
                newMessage.value = '';

                // Scroll to bottom
                scrollToBottom();
            } catch (err) {
                console.error('Error sending message:', err.message);
            }
        };

        const scrollToBottom = () => {
            nextTick(() => {
                const messageList = document.getElementById('messagelist');
                if (messageList) {
                    messageList.scrollTop = messageList.scrollHeight;
                }
            });
        };

        const toggleNotifications = () => {
            showNotifications.value = !showNotifications.value;
        };

        const markNotificationAsRead = (id) => {
            const notification = notifications.value.find(n => n.id === id);
            if (notification) notification.read = true;

            // In a real app, you'd send an API request to update the server
        };

        const markAllNotificationsAsRead = () => {
            notifications.value.forEach(notification => {
                notification.read = true;
            });

            // In a real app, you'd send an API request to update the server
        };

        // Lifecycle hooks
        onMounted(() => {
            // Initialize messages for the default selected contact
            getMessages();

            // Set up Echo for real-time updates
            if (window.Echo) {
                window.Echo.private("channel_for_everyone")
                    .listen('GotMessage', (e) => {
                        console.log('Received message:', e);
                        // Reload messages to include the new one
                        getMessages();

                        // If the message is from the currently selected contact,
                        // mark it as read. Otherwise, update unread state.
                        if (e.message && e.message.user_id !== selectedContactId.value) {
                            const contact = contacts.value.find(c => c.id === e.message.user_id);
                            if (contact) contact.unread = true;
                        }
                    });
            }
        });

        return {
            currentUser,
            contacts,
            filteredContacts,
            selectedContactId,
            messages,
            newMessage,
            notifications,
            showNotifications,
            selectContact,
            getSelectedContact,
            sendMessage,
            searchGlobal,
            toggleNotifications,
            markNotificationAsRead,
            markAllNotificationsAsRead
        };
    }
};
</script>

<style>
/* Base styles */
html, body {
    height: 100%;
    margin: 0;
    padding: 0;
    font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
    color: #333;
}

/* Scrollbar styling */
::-webkit-scrollbar {
    width: 6px;
    height: 6px;
}

::-webkit-scrollbar-track {
    background: transparent;
}

::-webkit-scrollbar-thumb {
    background: #d1d5db;
    border-radius: 3px;
}

::-webkit-scrollbar-thumb:hover {
    background: #9ca3af;
}

/* Animations */
.fade-enter-active,
.fade-leave-active {
    transition: opacity 0.3s ease;
}

.fade-enter-from,
.fade-leave-to {
    opacity: 0;
}

.slide-right-enter-active,
.slide-right-leave-active {
    transition: transform 0.3s ease;
}

.slide-right-enter-from,
.slide-right-leave-to {
    transform: translateX(-20px);
    opacity: 0;
}
</style>
