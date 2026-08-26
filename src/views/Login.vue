<template lang="pug">
div.login-view
  div.login-card
    img.login-logo(src="/logo.png", alt="logo")
    h3.mb-1 {{ $t('app.name') }}
    p.text-muted.mb-3 Introduce tu token de acceso
    b-form(@submit.prevent="save")
      b-form-input.mb-2(
        v-model="token",
        type="password",
        placeholder="API token",
        autofocus,
        :state="error ? false : null"
      )
      b-button.w-100(type="submit", variant="primary", :disabled="loading") Entrar
    b-button.w-100.mt-2(v-if="loggedIn", variant="outline-secondary", @click="logout") Cerrar sesión
    p.text-danger.mt-2.mb-0(v-if="error") {{ error }}
</template>

<script>
import { setApiToken, clearApiToken, hasApiToken, getClient } from '../util/awclient';

export default {
  name: 'Login',
  data() {
    return { token: '', error: '', loading: false, loggedIn: hasApiToken() };
  },
  methods: {
    async save() {
      const t = (this.token || '').trim();
      if (!t) {
        this.error = 'Introduce un token';
        return;
      }
      this.error = '';
      this.loading = true;
      setApiToken(t);
      try {
        // Validate the token against the server before proceeding.
        await getClient().req.get('/api/0/info');
        const redirect = this.$route.query.redirect || '/';
        this.$router.push(redirect);
      } catch (e) {
        clearApiToken();
        this.error = 'Token inválido o servidor no disponible';
      } finally {
        this.loading = false;
      }
    },
    logout() {
      clearApiToken();
      this.loggedIn = false;
      this.token = '';
      this.error = '';
    },
  },
};
</script>

<style scoped lang="scss">
.login-view {
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: 70vh;
  padding: 1rem;
}
.login-card {
  width: 100%;
  max-width: 340px;
  padding: 2rem 1.5rem;
  border: 1px solid rgba(0, 0, 0, 0.1);
  border-radius: 12px;
  text-align: center;
  background: var(--bs-body-bg, #fff);
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.06);
}
.login-logo {
  width: 72px;
  height: 72px;
  margin-bottom: 0.75rem;
}
</style>
