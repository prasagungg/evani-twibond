<script>
  // Simple Navbar component
</script>

<nav class="navbar">
  <div class="navbar-content">
    <div class="logo">
      <img src="/logo.png" alt="Evani Logo" class="logo-img" />
    </div>
  </div>
</nav>

<style>
  .navbar {
    position: fixed;
    top: 20px;
    left: 50%;
    transform: translateX(-50%);
    width: 90%;
    max-width: 480px; /* Constrain for mobile view */
    background: rgba(255, 255, 255, 0.4);
    backdrop-filter: blur(16px);
    -webkit-backdrop-filter: blur(16px);
    border: 1px solid rgba(255, 255, 255, 0.5);
    border-radius: var(--radius-pill);
    box-shadow: 0 4px 20px rgba(0, 0, 0, 0.05);
    z-index: 1000;
    padding: 10px 24px;
  }

  .navbar-content {
    display: flex;
    justify-content: center; /* Center the logo since there are no other links */
    align-items: center;
  }

  .logo {
    display: flex;
    align-items: center;
  }

  .logo-img {
    height: 36px;
    width: auto;
    object-fit: contain;
  }
</style>
