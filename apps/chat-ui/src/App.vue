<template>
    <div class="flex flex-col h-screen bg-slate-50 p-4">
        <!-- Header with Configuration -->
        <div class="bg-gradient-to-r from-blue-300 to-blue-200 p-2 flex items-center justify-between text-white shadow-lg">
            <button @click="deployAIChat"
                    class="bg-blue-200 hover:bg-blue-900 text-white font-bold py-2 px-4 rounded-lg transition-all duration-300">
                部署我的AIChat
            </button>
            <div>
                <select id="model-select" v-model="model" class="select p-2 rounded-lg bg-white text-gray-800 shadow-lg border-none focus:outline-none focus:ring-2 focus:ring-blue-400">
                    <option disabled selected>选择模型</option>
                    <option v-for="(modelOption, index) in modelOptions" :key="index" :value="modelOption.value">
                        {{ modelOption.label }}
                    </option>
                </select>
            </div>
        </div>

        <!-- Center Column (Chat) -->
        <div class="flex-grow flex flex-col w-full bg-gray-100 text-white p-6 overflow-auto relative">
            <!-- Background Text -->
            <div class="absolute inset-0 flex items-center justify-center pointer-events-none select-none">
                <span class="text-6xl sm:text-8xl md:text-9xl lg:text-[10rem] font-extrabold text-white/10">AI Chat</span>
            </div>

            <!-- Chat Content -->
            <div class="flex-grow overflow-y-auto mb-4 relative z-10">
                <ul class="space-y-4">
                    <li v-for="(conv, index) in conversations" :key="index" class="bg-gray-400 p-4 rounded-xl shadow-md">
                        <div class="flex items-center justify-between">
                            <strong class="text-blue-400">用户:</strong>
                            <span>{{ conv.user }}</span>
                        </div>
                        <div class="mt-2">
                            <strong class="text-green-300">AI:</strong>
                            <span>{{ conv.ai }}</span>
                        </div>
                    </li>
                </ul>
            </div>
        </div>

        <!-- Footer with Input -->
        <div class="bg-white p-4 flex items-center p-4 shadow-inner rounded-t-3xl">
            <textarea v-model="message"
                      placeholder="输入您的消息"
                      class="flex-grow mr-2 p-3 rounded-lg bg-white text-gray-800 resize-none border border-gray-300 focus:outline-none focus:ring-2 focus:ring-blue-300"
                      @keyup.enter="sendMessage"
                      rows="1"></textarea>
            <button @click="sendMessage"
                    :disabled="!message || !model || loading"
                    class="bg-blue-600 hover:bg-blue-700 text-white font-bold py-2 px-6 rounded-lg shadow-lg disabled:opacity-50 transition-all duration-300">
                {{ loading ? '发送中...' : '发送' }}
            </button>
        </div>
    </div>
</template>


<script>
    import axios from 'axios';

    export default {
        data() {
            return {
                message: '',
                model: null, // Ensure this is null or an invalid value initially
                models: [
                    { provider: 'baidu', name: 'ernie-4.0-8k-latest' },
                    { provider: 'baidu', name: 'ernie-3.0' }
                ],
                conversations: [],
                loading: false // Loading state for send button
            };
        },
        computed: {
            modelOptions() {
                return this.models.map((model) => ({
                    value: model,
                    label: `${model.name} (${model.provider})`
                }));
            }
        },
        methods: {
            deployAIChat() {
                window.open('https://functioncat.cn/console/clone-system?sourceOrgSlug=template&sourceSystemSlug=aichat3', '_blank');
            },
            async sendMessage() {
                console.log('sendMessage called');

                // Check if message and model are valid
                if (!this.message || !this.model) {
                    console.log('Message or model is empty');
                    return;
                }

                const apiUrl = `${process.env.VUE_APP_FUNCTIONCAT_SYSTEM_URL}/apps/chatapi/chat`;
                console.log('API URL:', apiUrl);

                try {
                    this.loading = true; // Set loading state to true
                    const response = await axios.post(apiUrl, {
                        message: this.message,
                        provider: this.model.provider,
                        model: this.model.name
                    });

                    console.log('Response:', response.data);

                    this.conversations.push({
                        user: this.message,
                        ai: response.data.response
                    });

                    this.message = '';
                } catch (error) {
                    console.error('Error sending message:', error);
                } finally {
                    this.loading = false; // Reset loading state
                }
            }
        },
        created() {
            // Automatically select the first model when the component is created
            if (this.models.length > 0) {
                this.model = this.models[0]; // Ensure this correctly sets the model
            }
        }
    };
</script>