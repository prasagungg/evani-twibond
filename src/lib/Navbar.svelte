<script>
  // Simple Navbar component
</script>

<nav class="navbar">
  <div class="navbar-content">
    <div class="logo">
      <span class="logo-text">Evani</span>
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
    background-color: rgba(255, 255, 255, 0.9);
    backdrop-filter: blur(10px);
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

  .logo-text {
    font-family: 'Montserrat', sans-serif;
    font-size: 1.5rem;
    font-weight: 800;
    color: var(--primary-red);
    letter-spacing: -0.5px;
  }
</style>
