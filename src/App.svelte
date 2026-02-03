<script>
  import { onMount, onDestroy } from "svelte";
  import bell from "./assets/bell.mp3";
  import FooterButtons from "./lib/FooterButtons.svelte";

  // --- CONFIGURATION ---
  const MODES = {
    focus: {
      label: "Focus",
      time: 25 * 60,
      color: "text-primary",
      ringColor: "text-primary",
    },
    break: {
      label: "Break",
      time: 5 * 60,
      color: "text-accent",
      ringColor: "text-accent",
    },
  };

  // --- STATE  ---
  let mode = $state("focus");
  let isRunning = $state(false);
  let timeLeft = $state(MODES.focus.time);
  let sessionsToday = $state(0);

  // --- DERIVED STATE ---
  let currentConfig = $derived(MODES[mode]);
  let progress = $derived((timeLeft / currentConfig.time) * 100);

  let formattedTime = $derived.by(() => {
    const m = Math.floor(timeLeft / 60);
    const s = timeLeft % 60;
    return `${m.toString().padStart(2, "0")}:${s.toString().padStart(2, "0")}`;
  });

  // --- AUDIO & SETUP ---
  let audio;
  onMount(() => {
    audio = new Audio(bell);

    const stored = localStorage.getItem("focus_sessions");
    const storedDate = localStorage.getItem("focus_date");
    const today = new Date().toDateString();

    if (storedDate === today && stored) {
      sessionsToday = parseInt(stored);
    } else {
      localStorage.setItem("focus_date", today);
    }
  });

  // --- LOGIC ---
  let interval;

  function toggleTimer() {
    isRunning = !isRunning;
    if (isRunning) {
      interval = setInterval(tick, 1000);
    } else {
      clearInterval(interval);
    }
  }

  function tick() {
    if (timeLeft > 0) {
      timeLeft -= 1;
      document.title = `(${formattedTime}) ${currentConfig.label}`;
    } else {
      completeSession();
    }
  }

  function completeSession() {
    clearInterval(interval);
    isRunning = false;
    audio.play();

    if (mode === "focus") {
      sessionsToday += 1;
      saveProgress();
      switchMode("break");
    } else {
      switchMode("focus");
    }

    if (Notification.permission === "granted") {
      new Notification(`${currentConfig.label} Finished!`);
    }
  }

  function switchMode(newMode) {
    if (mode === newMode && timeLeft === MODES[newMode].time) return;

    clearInterval(interval);
    isRunning = false;
    mode = newMode;
    timeLeft = MODES[newMode].time;
    document.title = "Focus";
  }

  function saveProgress() {
    localStorage.setItem("focus_sessions", sessionsToday.toString());
    localStorage.setItem("focus_date", new Date().toDateString());
  }

  onDestroy(() => {
    if (interval) clearInterval(interval);
  });

</script>

<main
  class="relative flex h-[100dvh] w-screen flex-col items-center justify-center bg-base-100 text-base-content overflow-hidden font-sans"
>
  <header
    class="absolute top-0 w-full flex flex-col items-center gap-6 pt-[calc(env(safe-area-inset-top)+2rem)] z-20"
  >
    <div class="flex flex-col items-center gap-2">
      <div class="flex gap-3">
        {#each Array(4) as _, i}
          <div
            class="w-3 h-3 rounded-full transition-all duration-500 border-2 border-primary/20"
            class:bg-primary={i < sessionsToday}
            class:scale-110={i < sessionsToday}
            class:bg-transparent={i >= sessionsToday}
          ></div>
        {/each}
      </div>
    </div>

    <div class="flex gap-2">
      <button
        class="btn btn-sm rounded-full transition-all border-none shadow-sm {mode ===
        'focus'
          ? 'btn-primary'
          : 'bg-base-200 text-base-content/40 hover:bg-base-300'}"
        onclick={() => switchMode("focus")}
      >
        Focus
      </button>
      <button
        class="btn btn-sm rounded-full transition-all border-none shadow-sm {mode ===
        'break'
          ? 'btn-accent'
          : 'bg-base-200 text-base-content/40 hover:bg-base-300'}"
        onclick={() => switchMode("break")}
      >
        Break
      </button>
    </div>
  </header>

  <div class="relative flex flex-col items-center justify-center">
    <div
      class="radial-progress text-base-200 absolute"
      style="--value:100; --size:70vmin; --thickness: 2vmin;"
    ></div>

    <div
      class="radial-progress transition-all duration-1000 ease-linear z-10 {currentConfig.ringColor}"
      style="--value:{progress}; --size:70vmin; --thickness: 2vmin;"
      role="progressbar"
    >
      <div class="flex flex-col items-center justify-center">
        <span
          class="font-mono font-bold tracking-tighter text-base-content/50"
          style="font-size: 14vmin;"
        >
          {formattedTime}
        </span>

        <span class="uppercase tracking-[0.2em] opacity-30 text-sm mt-2">
          {isRunning ? "Running" : "Paused"}
        </span>
      </div>
    </div>
  </div>

  <div
    class="absolute bottom-4 flex w-full flex-col items-center gap-6 px-4 z-20 pb-[env(safe-area-inset-bottom)]"
  >
    <button
      onclick={toggleTimer}
      class="btn btn-lg min-w-[200px] rounded-full shadow-lg transition-all active:scale-95 border-2"
      class:opacity-70={isRunning}
      class:btn-soft={isRunning}
      class:btn-primary={!isRunning && mode === "focus"}
      class:btn-accent={!isRunning && mode === "break"}
    >
      {isRunning ? "Pause" : "Start Timer"}
    </button>

    <FooterButtons />
  </div>
</main>
