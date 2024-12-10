  <script>
  import { onMount, afterUpdate, onDestroy } from 'svelte';

  let prompt = '';
  let loading = false;
  let error = '';
  let chatHistory = [];

  // IMPORTANT: Remove your API key from client-side code
  // Move API calls to a secure backend
  const OPENAI_API_KEY = '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';


  // Reference to the chat container for auto-scrolling
  let chatContainer;

  // Logo Animation Variables
  let logoSrc = '/ChatGPT-Logo.png'; // Default logo
  let logoInterval;

 async function sendPrompt() {
  if (!prompt.trim()) {
    console.log('Debug: No prompt entered.');
    alert('Please enter a prompt.');
    return;
  }

  console.log('Debug: Prompt entered:', prompt);
  loading = true;
  error = '';

  // Add user prompt to chat history
  const userMessage = { role: 'user', content: prompt };
  chatHistory = [...chatHistory, userMessage].slice(-6); // Keep only the last 6 messages
  console.log('Debug: Updated chat history with user message:', chatHistory);

  try {
    console.log('Debug: Sending request to OpenAI API...');
    const res = await fetch('https://api.openai.com/v1/chat/completions', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${OPENAI_API_KEY}`
      },
      body: JSON.stringify({
        model: "gpt-4",
        messages: chatHistory,
        temperature: 0.7
      })
    });

    console.log('Debug: Response received:', res);
    if (!res.ok) {
      const errorData = await res.json();
      console.error('Error response from OpenAI API:', errorData);
      throw new Error(errorData.error?.message || `Error: ${res.status} ${res.statusText}`);
    }

    const data = await res.json();
    console.log('Debug: Parsed response data:', data);

    const assistantMessage = data.choices[0].message.content;
    console.log('Debug: Assistant response:', assistantMessage);

    // Display the response word by word
    let displayedMessage = '';
    const words = assistantMessage.split(' ');
    for (const word of words) {
      displayedMessage += (displayedMessage ? ' ' : '') + word;
      chatHistory = [
        ...chatHistory.filter((msg) => msg.role !== 'assistant'),
        { role: 'assistant', content: displayedMessage }
      ].slice(-6);
      console.log('Debug: Updated chat history with assistant message:', displayedMessage);

      await new Promise((resolve) => setTimeout(resolve, 100)); // Delay in ms
    }

    prompt = ''; // Clear the prompt after sending
    console.log('Debug: Prompt cleared.');
  } catch (err) {
  

</script>

<main class="flex flex-col h-screen bg-gray-900 text-white">
  <!-- Header -->
  <header class="flex items-center justify-center h-16 bg-gray-800 border-b border-gray-700 w-full fixed top-0 left-0 z-10">
    <img src={logoSrc} alt="ChatGPT Logo" class="h-20 mr-2">
    <h1 class="text-xl font-semibold">LucasGPT</h1>
  </header>

  <!-- Main Content Wrapper -->
  <div class="flex-1 flex justify-center items-center pt-16"> <!-- pt-16 to offset the fixed header height -->
    <div class="w-full max-w-2xl flex flex-col h-full pb-10">
      <!-- Chat Area -->
      <div bind:this={chatContainer} class="flex-1 overflow-y-auto p-4 space-y-4 bg-gray-900 rounded-lg">
        {#each chatHistory as message, index}
          {#if message.role === 'user'}
            <div class="flex justify-end">
              <div class="max-w-xs bg-blue-600 text-white p-3 rounded-lg rounded-br-none">
                {message.content}
              </div>
            </div>
          {:else}
            <div class="flex justify-start">
              <div class="max-w-xs bg-gray-700 text-white p-3 rounded-lg rounded-bl-none">
                {message.content}
              </div>
            </div>
          {/if}
        {/each}
      </div>

      <!-- Input Area -->
      <div class="flex items-center mt-4">
        <textarea
          bind:value={prompt}
          placeholder="Type your message..."
          rows="1"
          class="flex-1 bg-gray-800 text-white p-2 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
          on:keydown={(e) => {
            if (e.key === 'Enter' && !e.shiftKey) {
              e.preventDefault();
              sendPrompt();
            }
          }}
        ></textarea>
        <button
          on:click={sendPrompt}
          disabled={loading}
          class="ml-2 bg-blue-600 hover:bg-blue-700 text-white font-semibold py-2 px-4 rounded-md disabled:opacity-50 disabled:cursor-not-allowed"
        >
          {loading ? 'Sending...' : 'Send'}
        </button>
      </div>
    </div>
  </div>

  {#if error}
    <div class="absolute bottom-20 left-1/2 transform -translate-x-1/2 bg-red-600 text-white p-3 rounded-md">
      <p>Error: {error}</p>
    </div>
  {/if}
</main>

<style>
  /* Optional: Smooth scrollbar for chat area */
  ::-webkit-scrollbar {
    width: 8px;
  }

  ::-webkit-scrollbar-thumb {
    background-color: rgba(255, 255, 255, 0.2);
    border-radius: 4px;
  }

  ::-webkit-scrollbar-thumb:hover {
    background-color: rgba(255, 255, 255, 0.4);
  }

  /* Ensure textarea does not exceed a certain height */
  textarea {
    max-height: 150px;
  }
</style>
