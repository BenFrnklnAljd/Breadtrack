<template>
  <div class="login-wrap">

    <div class="login-hero">
      <div class="login-logo"> PerishTrack</div>
      <div class="login-tagline">Inventory for perishable goods</div>
    </div>

    <div class="login-card">
      <Transition name="fade" mode="out-in">

        <!-- Sign In -->
        <div v-if="!showReg" key="login">
          <div class="login-card-title">Sign In</div>
          <div class="form-group">
            <label class="field-label">Username</label>
            <input class="field-input" v-model="lf.username" placeholder="e.g. maria" autocomplete="username" @keyup.enter="doLogin" />
          </div>
          <div class="form-group">
            <label class="field-label">Password / PIN</label>
            <input class="field-input" type="password" v-model="lf.password" placeholder="••••" autocomplete="current-password" @keyup.enter="doLogin" />
          </div>
          <p v-if="lErr" class="form-error">{{ lErr }}</p>
          <button class="btn btn-primary btn-block" @click="doLogin">Sign In</button>
          <p class="login-footer">
            No account? <button class="link-btn" @click="showReg = true">Create staff account</button>
          </p>
          <p class="demo-hint">Demo: <strong>admin</strong> / <strong>1234</strong></p>
        </div>

        <!-- Register -->
        <div v-else key="reg">
          <div class="login-card-title">New Account</div>
          <div class="form-group">
            <label class="field-label">Full Name</label>
            <input class="field-input" v-model="rf.name" placeholder="Staff name" />
          </div>
          <div class="form-group">
            <label class="field-label">Username</label>
            <input class="field-input" v-model="rf.username" placeholder="Login username" />
          </div>
          <div class="form-group">
            <label class="field-label">Password (min 4)</label>
            <input class="field-input" type="password" v-model="rf.password" placeholder="••••" />
          </div>
          <p v-if="rErr" class="form-error">{{ rErr }}</p>
          <button class="btn btn-primary btn-block" @click="doReg">Create Account</button>
          <p class="login-footer">
            <button class="link-btn" @click="showReg = false">← Back to Sign In</button>
          </p>
        </div>

      </Transition>
    </div>

  </div>
</template>

<script setup>
import { ref, reactive } from 'vue'
import { useRouter } from 'vue-router'
import { useAuthStore } from '@/stores/auth'
import { useToastStore } from '@/stores/toast'

const auth   = useAuthStore()
const toast  = useToastStore()
const router = useRouter()

const showReg = ref(false)
const lErr = ref('')
const rErr = ref('')
const lf = reactive({ username: '', password: '' })
const rf = reactive({ name: '', username: '', password: '' })

function doLogin() {
  lErr.value = ''
  const res = auth.login(lf.username, lf.password)
  if (!res.ok) { lErr.value = res.error; return }
  toast.success(`Welcome, ${auth.currentUser.name}!`)
  router.push('/dashboard')
}
function doReg() {
  rErr.value = ''
  const res = auth.register(rf)
  if (!res.ok) { rErr.value = res.error; return }
  toast.success('Account created!')
  showReg.value = false
  Object.assign(rf, { name: '', username: '', password: '' })
}
</script>

<style scoped>
/* ── Force dark theme on login page ── */
.login-wrap {
  min-height: 100dvh;
  display: flex;
  flex-direction: column;
  justify-content: center;
  padding: 32px 20px;
  max-width: var(--max-w);
  margin: 0 auto;
  background: #0a0a09;
}

.login-hero { margin-bottom: 32px; }
.login-logo { font-family: var(--font-head); font-size: 28px; font-weight: 800; color: #f5a623; }
.login-tagline { font-size: 14px; color: #5c5853; margin-top: 4px; }

.login-card {
  background: #141412;
  border: 1px solid #282826;
  border-radius: var(--radius);
  padding: 24px 20px;
}
.login-card-title { font-family: var(--font-head); font-size: 17px; font-weight: 700; margin-bottom: 18px; color: #f0ede8; }

/* Override field inputs for dark */
.login-wrap :deep(.field-label) { color: #5c5853; }
.login-wrap :deep(.field-input) {
  background: #1c1c1a;
  border-color: #282826;
  color: #f0ede8;
}
.login-wrap :deep(.field-input:focus) {
  border-color: #f5a623;
  box-shadow: 0 0 0 3px rgba(245,166,35,.15);
}
.login-wrap :deep(.field-input::placeholder) { color: #5c5853; }

/* Override button for dark */
.login-wrap :deep(.btn-primary) {
  background: #f5a623;
  color: #0a0a09;
}

.login-footer { text-align: center; font-size: 13px; color: #5c5853; margin-top: 14px; }
.link-btn { background: none; border: none; color: #f5a623; cursor: pointer; font-size: 13px; font-family: var(--font-body); }
.demo-hint { text-align: center; font-size: 11px; color: #5c5853; margin-top: 8px; }
</style>
