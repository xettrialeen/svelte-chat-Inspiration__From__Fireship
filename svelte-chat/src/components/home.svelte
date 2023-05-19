<script>
  import { Link } from "svelte-routing";

  import pb from "../lib/pocketbaseInit";
  import { prevent_default } from "svelte/internal";


  let form = {};

  // handle form submission
  const handleRegister = async () => {
try {
  let response = await pb.collection("users").create({
      username: form.username,
      email: form.email,
      password: form.password,
      passwordConfirm: form.password,
    });


} catch (error) {
  
}
  };
</script>

<div class="home-container">
  <h2>Welcome to pocketbase + Svelte Chat APP 🔥</h2>

  <form on:submit|preventDefault>
    <!-- Grid -->
    <div class="grid">
      <!-- Markup example 1: input is inside label -->
      <label for="username">
        Username
        <input
          type="text"
          id="username"
          name="username"
          placeholder="User Name"
          required
          bind:value={form.username}
        />
      </label>
    </div>

    <!-- Markup example 2: input is after label -->
    <label for="email">Email address</label>
    <input
      type="email"
      id="email"
      name="email"
      placeholder="Email address"
      required
      bind:value={form.email}
    />

    <label for="password">Password</label>
    <input
      type="password"
      id="password"
      name="password"
      placeholder="Your Password"
      required
      bind:value={form.password}
    />

    <small>Already a user? 😮 <Link to="/login">Login</Link></small>

    <!-- Button -->
    <button on:click={handleRegister}>Register</button>
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
