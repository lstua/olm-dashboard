<template>
  <div>
    <b-navbar type="dark" variant="olm-dark">
      <NuxtLink to="/">
        <b-navbar-brand class="pt-0">
          OLM Dashboard
        </b-navbar-brand>
      </NuxtLink>
    </b-navbar>

    <b-row no-gutters>
      <b-col>
        <b-nav tabs justified class="shadow-sm mb-3">
          <b-nav-item to="/" class="link-invisible">My Progress</b-nav-item>
          <b-nav-item to="/ideal" class="link-invisible">Ideal</b-nav-item>
          <b-nav-item to="/me-vs-ideal" class="link-invisible">Me vs Ideal</b-nav-item>
        </b-nav>
      </b-col>
      <b-col class="flex-container-grey"></b-col>
    </b-row>

    <Nuxt/>
  </div>
</template>
