<template>
  <q-layout view="lHh Lpr lFf">
    <q-header>
      <q-btn
        class="absolute q-ma-lg bg-blur"
        color="white"
        dense
        flat
        round
        icon="menu"
        aria-label="Menu"
        @click="toggleLeftDrawer"
      />
    </q-header>

    <q-drawer
      v-model="leftDrawerOpen"
      class="column"
      :width="100"
      show-if-above
      bordered
    >
      <!-- <div class="text-h6 q-pa-md">Welcome Sam</div> -->
      <q-list dense>
        <login-btn />
        <q-separator inset />
        <div v-if="!user" class="col-12 column items-stretch">
          <!-- <login-google-btn
            class="fn-xs"
            unelevated
            v-bind="$attrs"
            provider="fb"
          /> -->
          <login-google-btn class="fn-xs" unelevated v-bind="$attrs" />
          <!-- <login-google-btn
            class="fn-xs"
            unelevated
            v-bind="$attrs"
            provider="email"
          /> -->
        </div>
      </q-list>
      <q-list v-if="user" dense>
        <q-item
          v-for="route in routes"
          :key="route.name"
          clickable
          v-ripple
          :to="route.route"
          active-class="bg-info"
          class="row justify-center q-py-lg"
        >
          <div class="column items-center q-py-sm">
            <q-icon color="secondary" :name="route.icon" size="2rem" />
            <div class="text-secondary">{{ route.name }}</div>
          </div>
        </q-item>
      </q-list>
      <q-space />
      <q-item
        v-if="user"
        clickable
        @click="logout"
        v-ripple
        class="row justify-center q-py-lg op-20"
      >
        <div class="column items-center q-py-sm">
          <q-icon color="primary" name="logout" size="2rem" />
          <div>Logout</div>
        </div>
      </q-item>
    </q-drawer>

    <q-page-container class="full-height bg">
      <q-page>
        <div class="q-pt-xl">
          <router-view />
        </div>
      </q-page>
    </q-page-container>
  </q-layout>
</template>

<style lang="scss" scoped>
.bg {
  background: rgb(23, 77, 98);
  background: linear-gradient(
    54deg,
    rgba(23, 77, 98, 1) 0%,
    rgba(38, 80, 149, 1) 20%,
    rgba(88, 58, 142, 1) 100%
  );
}
</style>

<script lang="ts">
import { defineComponent, ref, computed, watchEffect } from 'vue';
import LoginBtn from 'src/components/auth/LoginBtn.vue';
import LoginGoogleBtn from 'src/components/auth/LoginGoogleBtn.vue';
import { useFirebaseUser } from '@gcto/firebase-hooks/lib';

import firebase from 'firebase';
import { useRouter } from 'vue-router';

export default defineComponent({
  name: 'MainLayout',
  components: { LoginBtn, LoginGoogleBtn },
  setup() {
    const _user = useFirebaseUser();
    const user = computed(() => _user.data.value);

    const router = useRouter();
    watchEffect(() => {
      if (user.value?.uid) {
        if (router.currentRoute.value.path == '/portal/start')
          router.push('/portal/dashboard');
      } else {
        if (user.value?.uid == null) {
          console.log('C');
          router.push('/portal/start');
        }
      }
    });

    const leftDrawerOpen = ref(false);
    const routes = [
      { name: 'Dashboard', icon: 'dashboard', route: '/portal/dashboard' },

      { name: 'Wifi QRs', icon: 'wifi', route: '/portal/wifi' },
      { name: 'Settings', icon: 'settings', route: '/portal/settings' },
    ];
    const logout = () => {
      firebase.auth().signOut();
    };
    return {
      leftDrawerOpen,
      toggleLeftDrawer() {
        leftDrawerOpen.value = !leftDrawerOpen.value;
      },
      logout,
      routes,
      user,
    };
  },
});
</script>
