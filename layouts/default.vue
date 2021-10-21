<template>
  <div>
    <b-navbar type="dark" variant="olm-dark">
      <NuxtLink to="/">
        <b-navbar-brand class="pt-0">
          INFO1112
        </b-navbar-brand>
      </NuxtLink>
    </b-navbar>

    <b-row no-gutters>
      <b-col>
        <b-nav tabs justified class="shadow-sm mb-3">
          <b-nav-item to="/">My Progress</b-nav-item>
          <b-nav-item to="/ideal">Ideal</b-nav-item>
          <b-nav-item to="/me-vs-ideal">Me vs Ideal</b-nav-item>
        </b-nav>
      </b-col>
      <b-col class="flex-container-grey"></b-col>
    </b-row>

    <Nuxt/>
  </div>
</template>
