<template>
  <v-app id="inspire">
    <v-app-bar app color="white" flat>
      <v-container class="py-0 fill-height">
        <v-avatar class="mr-10" color="grey darken-1" size="32"></v-avatar>

        <v-btn v-for="link in links" :key="link" text @click.prevent="go(link)">
          {{ link }}
        </v-btn>

        <v-spacer></v-spacer>

        <v-responsive max-width="260">
          <v-text-field dense flat hide-details rounded solo-inverted></v-text-field>
        </v-responsive>
      </v-container>
    </v-app-bar>

    <v-main class="grey lighten-3">
      <v-container>
        <v-row>
          <v-col cols="2">
            <v-sheet rounded="lg">
              <v-list color="transparent">
                <v-list-item v-for="link in links" :key="link" link @click.prevent="go(link)">
                  <v-list-item-content>
                    <v-list-item-title>
                      {{ link }}
                    </v-list-item-title>
                  </v-list-item-content>
                </v-list-item>

                <v-divider class="my-2"></v-divider>

                <v-list-item link color="grey lighten-4" @click.prevent="go('permisos')">
                  <v-list-item-content>
                    <v-list-item-title>
                      Permisos
                    </v-list-item-title>
                  </v-list-item-content>
                </v-list-item>
              </v-list>
            </v-sheet>
          </v-col>

          <v-col>
            <v-sheet min-height="70vh" rounded="lg">
              <loadingInfo v-if="loading"></loadingInfo>
              <div v-if="!loading">
                <component v-bind:is="currentTabComponent"></component>
              </div>
            </v-sheet>
          </v-col>
        </v-row>
      </v-container>
    </v-main>
  </v-app>
</template>

<script>
import loadingInfo from './loadingInfo';
import defaultView from './sections/defaultView.vue';
import permisosView from './permisosUsuario.vue';
import denegAccess from './sections/denegAccess.vue';

//
import mensajes from './sections/messagesComp.vue';
import galeria from './sections/dashboardComp.vue';

export default {
  data: () => ({
    links: [
      'Galeria',
      'Mensajes',
    ],
    loading: false,
    currentTabComponent: null,
    currentView: 'Home',
    userId: 159,
  }),

  components: {
    loadingInfo,
    defaultView
  },

  mounted() {
    this.currentTabComponent = defaultView;
    this.useCan = true;
  },

  //
  methods: {
    async go(name) {
      switch (name) {
        case 'permisos':
          if (this.currentView === 'permisos') return;

          // Reset
          this.reset();
          this.loading = true;
          this.currentTabComponent = loadingInfo;

          // Tiempo de carga deoquis
          setTimeout(() => {
            this.reset();
            this.currentTabComponent = permisosView;
            this.currentView = 'permisos';
          }, 1000);
          break;
        case 'Galeria':
        case 'Mensajes':
          // Reset
          this.reset();
          this.loading = true;
          this.currentTabComponent = loadingInfo;
          // Revisar componente a cargar
          if (await this.check(name.toLowerCase())) {
            setTimeout(() => {
              this.reset();
              this.currentTabComponent = name.toLowerCase() === 'mensajes' ? mensajes : galeria;
              this.currentView = name.toLowerCase();
            }, 500);
          } else {
            this.reset();
            this.currentTabComponent = denegAccess;
            this.currentView = name.toLowerCase();
          }
          break;
      }
    },
    reset: function () {
      this.loading = false;
      this.currentTabComponent = null;
    },
    async check(screen) {
      const rawResponse = await fetch('http://localhost:3476/v1/tenants/t1/permissions/check', {
        method: 'POST',
        headers: {
          'Accept': 'application/json',
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({
          "metadata": {
            "snap_token": "",
            "schema_version": "",
            "depth": 100
          },
          "entity": {
            "type": "screens",
            "id": screen
          },
          "permission": "view",
          "subject": {
            "type": "user",
            "id": "" + this.userId
          }
        })
      });
      const content = await rawResponse.json();
      if (content.can && content.can === 'RESULT_DENIED') return false;
      return true;
    }
  }
}
</script>