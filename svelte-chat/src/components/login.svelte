<script>
  import pb from "../lib/pocketbaseInit";

  import { currentUser } from "../lib/pocketbaseInit";
  import { Link, navigate } from "svelte-routing";
  let form = {};
  const handleLogin = async () => {
    let loginResponse = await pb
      .collection("users")
      .authWithPassword(form.username, form.password);

    if ($currentUser) {
      navigate("/chat", { replace: true });
    }
  };
</script>

<div class="home-container">
  <h2>Welcome to pocketbase + Svelte Chat APP 🔥</h2>

  <form on:submit|preventDefault>
    <!-- Grid -->
    <div class="grid">
      <!-- Markup example 1: input is inside label -->
      <label for="firstname">
        Username
        <input
          type="text"
          id="firstname"
          name="firstname"
          placeholder="First name"
          required
          bind:value={form.username}
        />
      </label>
    </div>

    <label for="password">Password</label>
    <input
      type="password"
      id="password"
      name="password"
      placeholder="Your Password"
      required
      bind:value={form.password}
    />

    <small>Not registered yet? ❤️‍🔥 <Link to="/">Register</Link></small>

    <!-- Button -->
    <button on:click={handleLogin}>Login</button>
  </form>
</div>

<style>
  .home-container {
    padding-top: 50px;
  }

  h2 {
    font-weight: 600;
    font-size: 1.2rem;
  }

  small {
    margin-top: 20px;
  }
</style>
