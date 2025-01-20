<template>
  <nav id="navbar">
    <div class="logo">
      <RouterLink to="/">$pend$mart</RouterLink>
    </div>
    <div class="account">
      <p style="margin-right: 10px;">John Doe</p>
      <span class="material-icons">account_circle</span>
    </div>
  </nav>

  <router-view class="view"/>
</template>

<style scoped lang="scss">
#navbar {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 80px;
  background-color: #024e19;
  color: $text-on-primary;
  display: flex;
  justify-content: space-between;
  align-items: center;
  box-shadow: 0px 4px 10px rgba(0, 0, 0, 0.3);
  z-index: 1000;

  a {
    color: #ccc;
    font-weight: bold;
    text-decoration: none;
  }

  a.router-link-active {
    color: #fff; /* Active link color */
  }

  .logo {
    margin-left: 20px;

    flex: 1;
    display: flex;
    justify-content: flex-start;
    align-items: center;

    font-weight: bold;
    font-size: 2rem;

    flex: 1;

    a {
      text-decoration: none;
    }
  }

  .account {
    margin-right: 20px;

    flex: 1;
    display: flex;
    justify-content: flex-end;
    align-items: center;
  }
}

.view {
  margin-top: 100px;
}
</style>
