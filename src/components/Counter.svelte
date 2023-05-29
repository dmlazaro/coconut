<script lang="ts">
  import { fly } from "svelte/transition";
  import { onMount } from "svelte";

  let currentWebSocket: WebSocket | null = null;
  let rejoined = false;
  let startTime = Date.now();

  let count = 0;
  let prev = count;

  async function rejoin() {
    if (!rejoined) {
      rejoined = true;
      currentWebSocket = null;

      // TODO: Notify disconnected

      let timeSinceLastJoin = Date.now() - startTime;
      if (timeSinceLastJoin < 10000) {
        await new Promise((resolve) =>
          setTimeout(resolve, 10000 - timeSinceLastJoin)
        );
      }

      join();
    }
  }

  async function join() {
    let ws = new WebSocket("ws://coconut-data.cloudflare3834.workers.dev/");
    ws.addEventListener("open", (event) => {
      currentWebSocket = ws;
    });
    ws.addEventListener("message", (event) => {
      let data = JSON.parse(event.data);
      if (data.error) {
        console.error(data.error);
      } else {
        prev = count;
        count = data;
      }
    });
    ws.addEventListener("close", (event) => {
      console.log("WebSocket closed, reconnecting:", event.code, event.reason);
      rejoin();
    });
    ws.addEventListener("error", (event) => {
      console.log("WebSocket error, reconnecting:", event);
      rejoin();
    });
  }

  onMount(join);
</script>

<div class="flex flex-col items-center justify-center gap-8">
  <button
    class="btn btn-circle z-10"
    on:click={() => {
      currentWebSocket?.send("1");
    }}
    ><svg
      xmlns="http://www.w3.org/2000/svg"
      class="icon icon-tabler icon-tabler-plus"
      width="24"
      height="24"
      viewBox="0 0 24 24"
      stroke-width="2"
      stroke="currentColor"
      fill="none"
      stroke-linecap="round"
      stroke-linejoin="round"
    >
      <path stroke="none" d="M0 0h24v24H0z" fill="none" />
      <path d="M12 5l0 14" />
      <path d="M5 12l14 0" />
    </svg></button
  >
  <div class="h-10">
    {#key count}
      <div
        class="text-4xl {count > 0
          ? 'text-green-300'
          : count < 0
          ? 'text-red-300'
          : 'text-neutral'}"
        in:fly={{ y: count > prev ? 25 : -25, delay: 70, duration: 70 }}
        out:fly={{ y: count < prev ? 25 : -25, duration: 70 }}
      >
        {count}
      </div>
    {/key}
  </div>
  <button
    class="btn btn-circle z-10"
    on:click={() => {
      currentWebSocket?.send("-1");
    }}
    ><svg
      xmlns="http://www.w3.org/2000/svg"
      class="icon icon-tabler icon-tabler-minus"
      width="24"
      height="24"
      viewBox="0 0 24 24"
      stroke-width="2"
      stroke="currentColor"
      fill="none"
      stroke-linecap="round"
      stroke-linejoin="round"
    >
      <path stroke="none" d="M0 0h24v24H0z" fill="none" />
      <path d="M5 12l14 0" />
    </svg></button
  >
</div>
