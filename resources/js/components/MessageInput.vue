<template>
    <div class="border-t bg-white p-3">
        <!-- Quick Message Dropdown -->
        <div class="mb-3 flex justify-end">
            <div class="relative inline-block text-left">
                <div>
                    <button
                        type="button"
                        class="inline-flex justify-center items-center rounded-md border border-gray-300 shadow-sm px-4 py-2 bg-white text-sm font-medium text-gray-700 hover:bg-gray-50 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2 transition-colors duration-200"
                        @click="toggleQuickMessages"
                    >
                        <svg class="w-5 h-5 mr-2 text-gray-400" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 10h.01M12 10h.01M16 10h.01M9 16H5a2 2 0 01-2-2V6a2 2 0 012-2h14a2 2 0 012 2v8a2 2 0 01-2 2h-5l-5 5v-5z"></path>
                        </svg>
                        <span>Quick Message</span>
                        <svg class="h-5 w-5 ml-2 -mr-1 text-gray-400" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" fill="currentColor" aria-hidden="true">
                            <path fill-rule="evenodd" d="M5.293 7.293a1 1 0 011.414 0L10 10.586l3.293-3.293a1 1 0 111.414 1.414l-4 4a1 1 0 01-1.414 0l-4-4a1 1 0 010-1.414z" clip-rule="evenodd" />
                        </svg>
                    </button>
                </div>

                <!-- Dropdown Menu -->
                <div
                    v-if="showQuickMessages"
                    class="origin-top-right absolute right-0 mt-2 w-72 rounded-md shadow-lg bg-white ring-1 ring-black ring-opacity-5 focus:outline-none z-10 divide-y divide-gray-100"
                    v-click-outside="closeQuickMessages"
                >
                    <div class="py-1">
                        <button
                            v-for="(message, index) in quickMessages"
                            :key="index"
                            @click="selectQuickMessage(message)"
                            class="block w-full text-left px-4 py-2 text-sm text-gray-700 hover:bg-gray-100 transition-colors duration-150"
                        >
                            {{ message }}
                        </button>
                    </div>
                </div>
            </div>
        </div>

        <!-- Message Input Area -->
        <div class="flex items-center bg-white rounded-lg border shadow-sm">
            <!-- Attachment Button -->
            <button
                type="button"
                class="p-2 text-gray-500 hover:text-gray-700 focus:outline-none transition-colors duration-200"
                aria-label="Attach file"
            >
                <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15.172 7l-6.586 6.586a2 2 0 102.828 2.828l6.414-6.586a4 4 0 00-5.656-5.656l-6.415 6.585a6 6 0 108.486 8.486L20.5 13"></path>
                </svg>
            </button>

            <!-- Text Input -->
            <input
                type="text"
                :value="modelValue"
                @input="updateValue"
                placeholder="Write your message here"
                class="flex-1 py-3 px-4 focus:outline-none text-gray-700 placeholder-gray-400"
                @keyup.enter="sendMessage"
            >

            <!-- Emoji Button -->
            <button
                type="button"
                class="p-2 text-gray-500 hover:text-gray-700 focus:outline-none transition-colors duration-200"
                @click="toggleEmojiPicker"
                aria-label="Add emoji"
            >
                <svg class="w-6 h-6" fill="currentColor" viewBox="0 0 20 20" xmlns="http://www.w3.org/2000/svg">
                    <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zM7 9a1 1 0 100-2 1 1 0 000 2zm7-1a1 1 0 11-2 0 1 1 0 012 0zm-7.536 5.879a1 1 0 001.415 0 3 3 0 014.242 0 1 1 0 001.415-1.415 5 5 0 00-7.072 0 1 1 0 000 1.415z" clip-rule="evenodd"></path>
                </svg>
            </button>

            <!-- Send Button -->
            <button
                type="button"
                class="p-2 text-blue-500 hover:text-blue-700 focus:outline-none transition-colors duration-200"
                @click="sendMessage"
                aria-label="Send message"
                :disabled="!canSend"
                :class="{ 'opacity-50 cursor-not-allowed': !canSend }"
            >
                <svg class="w-6 h-6 transform rotate-90" fill="currentColor" viewBox="0 0 20 20" xmlns="http://www.w3.org/2000/svg">
                    <path d="M10.894 2.553a1 1 0 00-1.788 0l-7 14a1 1 0 001.169 1.409l5-1.429A1 1 0 009 15.571V11a1 1 0 112 0v4.571a1 1 0 00.725.962l5 1.428a1 1 0 001.17-1.408l-7-14z"></path>
                </svg>
            </button>
        </div>

        <!-- Emoji Picker (simplified) -->
        <div
            v-if="showEmojiPicker"
            class="fixed bottom-20 right-6 bg-white shadow-lg rounded-lg p-4 border z-20"
            v-click-outside="closeEmojiPicker"
        >
            <div class="grid grid-cols-8 gap-2">
                <button
                    v-for="emoji in commonEmojis"
                    :key="emoji"
                    @click="addEmoji(emoji)"
                    class="text-xl hover:bg-gray-100 rounded p-1 transition-colors duration-150"
                >
                    {{ emoji }}
                </button>
            </div>
        </div>
    </div>
</template>

<script>
import { ref, computed } from 'vue';

export default {
    name: 'MessageInput',
    props: {
        modelValue: {
            type: String,
            default: ''
        },
        placeholder: {
            type: String,
            default: 'Write your message here'
        }
    },
    emits: ['update:modelValue', 'send'],
    setup(props, { emit }) {
        const showQuickMessages = ref(false);
        const showEmojiPicker = ref(false);

        // Quick message templates
        const quickMessages = [
            "Thanks for your feedback!",
            "We'll get back to you as soon as possible.",
            "Could you please provide more details?",
            "Is there anything else I can help you with?",
            "Thank you for reaching out to us.",
            "I'll check this and update you shortly."
        ];

        // Common emoji selection
        const commonEmojis = [
            "😊", "👍", "❤️", "🎉", "👏", "🙏", "💯", "🔥",
            "😂", "😍", "🤔", "👀", "💪", "✅", "⭐", "🚀",
            "😁", "🤗", "😉", "🥳", "🤩", "💖", "👌", "🙌"
        ];

        // Check if can send message
        const canSend = computed(() => props.modelValue && props.modelValue.trim().length > 0);

        // Update the v-model value
        const updateValue = (event) => {
            emit('update:modelValue', event.target.value);
        };

        const sendMessage = () => {
            if (canSend.value) {
                emit('send', props.modelValue);
            }
        };

        const toggleQuickMessages = () => {
            showQuickMessages.value = !showQuickMessages.value;
            if (showQuickMessages.value) {
                showEmojiPicker.value = false;
            }
        };

        const closeQuickMessages = () => {
            showQuickMessages.value = false;
        };

        const toggleEmojiPicker = () => {
            showEmojiPicker.value = !showEmojiPicker.value;
            if (showEmojiPicker.value) {
                showQuickMessages.value = false;
            }
        };

        const closeEmojiPicker = () => {
            showEmojiPicker.value = false;
        };

        const selectQuickMessage = (message) => {
            emit('update:modelValue', message);
            showQuickMessages.value = false;
        };

        const addEmoji = (emoji) => {
            emit('update:modelValue', (props.modelValue || '') + emoji);
        };

        // Custom directive for handling clicks outside
        const vClickOutside = {
            mounted(el, binding) {
                el.clickOutsideEvent = (event) => {
                    if (!(el === event.target || el.contains(event.target))) {
                        binding.value(event);
                    }
                };
                document.addEventListener('click', el.clickOutsideEvent);
            },
            unmounted(el) {
                document.removeEventListener('click', el.clickOutsideEvent);
            }
        };

        return {
            quickMessages,
            commonEmojis,
            showQuickMessages,
            showEmojiPicker,
            canSend,
            updateValue,
            sendMessage,
            toggleQuickMessages,
            closeQuickMessages,
            toggleEmojiPicker,
            closeEmojiPicker,
            selectQuickMessage,
            addEmoji,
            vClickOutside
        };
    }
};
</script>

<style scoped>
/* Input focus ring */
input:focus {
    box-shadow: 0 0 0 2px rgba(59, 130, 246, 0.2);
}

/* Quick message dropdown animation */
.origin-top-right {
    transform-origin: top right;
    animation: dropdown-appear 0.15s ease-out;
}

@keyframes dropdown-appear {
    from {
        transform: scale(0.95);
        opacity: 0;
    }
    to {
        transform: scale(1);
        opacity: 1;
    }
}

/* Emoji picker animation */
.emoji-picker {
    animation: slide-up 0.2s ease-out;
}

@keyframes slide-up {
    from {
        transform: translateY(10px);
        opacity: 0;
    }
    to {
        transform: translateY(0);
        opacity: 1;
    }
}

/* Button hover effects */
button:not(.opacity-50) {
    transition: all 0.2s ease;
}

button:not(.opacity-50):hover {
    transform: translateY(-1px);
}

button:not(.opacity-50):active {
    transform: translateY(0);
}
</style>
