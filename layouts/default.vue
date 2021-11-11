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
        <b-nav class="shadow-sm mb-3" justified tabs>
          <b-nav-item class="link-invisible" exact exact-active-class="active" to="/">My Progress</b-nav-item>
          <b-nav-item class="link-invisible" exact exact-active-class="active" to="/ideal">Ideal</b-nav-item>
          <b-nav-item class="link-invisible" exact exact-active-class="active" to="/me-vs-ideal">Me vs Ideal
          </b-nav-item>
        </b-nav>
      </b-col>
      <b-col class="flex-container-grey"></b-col>
    </b-row>

    <Nuxt/>
  </div>
</template>
