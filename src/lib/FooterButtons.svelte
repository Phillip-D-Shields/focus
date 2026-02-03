<script>
  import { onMount } from "svelte";

  let { coffeeUrl = "https://www.buymeacoffee.com/phill_shields" } = $props();

  let deferredPrompt = $state(null);
  let isStandalone = $state(false);

  onMount(() => {
    // Check if the app is already installed/running in standalone mode
    isStandalone = window.matchMedia('(display-mode: standalone)').matches || 
                   (window.navigator).standalone === true;

    // Capture the install prompt for Chromium-based browsers
    window.addEventListener("beforeinstallprompt", (e) => {
      e.preventDefault();
      deferredPrompt = e;
    });
  });

  async function handleInstall() {
    if (deferredPrompt) {
      deferredPrompt.prompt();
      const { outcome } = await deferredPrompt.userChoice;
      if (outcome === 'accepted') deferredPrompt = null;
    } else {
      // Fallback for iOS/Desktop Safari/Firefox
      const modal = document.getElementById('install_modal');
      if (modal) modal.showModal();
    }
  }
</script>

<div class="flex justify-around w-full mt-auto py-4">
  <a
    href={coffeeUrl}
    target="_blank"
    rel="noreferrer"
    class="btn btn-ghost btn-xs gap-2 opacity-40 hover:opacity-100 transition-all"
  >
    <svg xmlns="http://www.w3.org/2000/svg" class="h-3 w-3" fill="none" viewBox="0 0 24 24" stroke="currentColor">
      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4.318 6.318a4.5 4.5 0 000 6.364L12 20.364l7.682-7.682a4.5 4.5 0 00-6.364-6.364L12 7.636l-1.318-1.318a4.5 4.5 0 00-6.364 0z" />
    </svg>
    Support Dev
  </a>

  {#if !isStandalone}
    <button
      onclick={handleInstall}
      class="btn btn-ghost btn-xs gap-2 opacity-40 hover:opacity-100 transition-all"
    >
      <svg xmlns="http://www.w3.org/2000/svg" class="h-3 w-3" fill="none" viewBox="0 0 24 24" stroke="currentColor">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-4l-4 4m0 0l-4-4m4 4V4" />
      </svg>
      Install App
    </button>
  {/if}
</div>

<dialog id="install_modal" class="modal modal-bottom sm:modal-middle">
  <div class="modal-box bg-base-100 border border-primary/10">
    <h3 class="font-bold text-lg text-center mb-6">Add to Home Screen</h3>
    
    <div class="space-y-4">
      <div class="flex items-start gap-4 p-4 bg-base-200 rounded-2xl">
        <div class="bg-primary text-primary-content rounded-full h-6 w-6 flex items-center justify-center shrink-0 font-bold text-xs">1</div>
        <p class="text-sm">Tap the <strong>Share</strong> button in your browser's menu (usually at the bottom or top bar).</p>
      </div>

      <div class="flex items-start gap-4 p-4 bg-base-200 rounded-2xl">
        <div class="bg-primary text-primary-content rounded-full h-6 w-6 flex items-center justify-center shrink-0 font-bold text-xs">2</div>
        <p class="text-sm">Scroll down and select <strong>"Add to Home Screen"</strong>.</p>
      </div>
    </div>

    

    <div class="modal-action">
      <form method="dialog" class="w-full">
        <button class="btn btn-primary w-full rounded-full">Got it</button>
      </form>
    </div>
  </div>
  <form method="dialog" class="modal-backdrop">
    <button>close</button>
  </form>
</dialog>