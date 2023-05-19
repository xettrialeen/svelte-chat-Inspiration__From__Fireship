<script>
  import "@picocss/pico";
  import { Router, Link, Route, navigate } from "svelte-routing";
  import Home from "./components/home.svelte";
  import Login from "./components/login.svelte";
  import { currentUser } from "./lib/pocketbaseInit";
  import Chat from "./components/chat.svelte";
  import { onMount } from "svelte";

  // customUser

  onMount(() => {
    if ($currentUser) {
      navigate("/chat", { replace: true });
    }
  });
</script>

<main class="container main-container">
  <Router>
    {#if $currentUser}
      <Route path="/chat" component={Chat} />
    {:else}
      <Route path="/" component={Home} />
      <Route path="/login" component={Login} />
    {/if}
  </Router>
</main>

<style>
  .main-container {
    display: flex;
    justify-content: center;
  }
</style>
