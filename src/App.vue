<template>
  <v-app>
    <v-container>
      <v-layout text-center wrap>
        <v-flex xs12>
          <v-img :src="require('./assets/logo.svg')" class="my-3" contain height="200"></v-img>
        </v-flex>

        <v-flex class="d-flex justify-center mb-10" md6 offset-md-3 offset-xl-4 xl4 xs12>
          <v-btn
            :disabled="loading()"
            :loading="loading()"
            @click="refresh()"
            class="ma-2 white--text"
            color="blue-grey darken-1"
            dark
            x-large
          >
            Atualizar
            <v-icon dark right>refresh</v-icon>
          </v-btn>
        </v-flex>

        <v-flex class="d-flex justify-space-between mb-5" md10 offset-md-1 offset-xl-3 xl6 xs12>
          <h1 class="headline">Serviços Corporativos</h1>
        </v-flex>

        <v-flex class="d-flex justify-space-between" md10 offset-md-1 offset-xl-3 v-if="cCorporate.sum > 0" xl6 xs12>
          <v-progress-linear indeterminate></v-progress-linear>
        </v-flex>

        <v-flex class="d-flex justify-space-between mb-10" md10 offset-md-1 offset-xl-3 xl6 xs12>
          <v-expansion-panels>
            <v-expansion-panel :key="i" v-for="(item,i) in sCorporate">
              <v-expansion-panel-header disable-icon-rotate>
                {{ item.name }}
                <template v-slot:actions>
                  <v-chip
                    class="py-0"
                    color="blue darken-1"
                    label
                    text-color="white"
                    v-if="item.status === undefined || item.status === null"
                  >
                    Verificando...
                    <v-icon right style="color: #fff;">hourglass_empty</v-icon>
                  </v-chip>
                  <v-chip class="py-0" :color="item.status.color" label text-color="white" v-else>
                    {{ item.status.label }}
                    <v-icon right style="color: #fff;">{{ item.status.icon }}</v-icon>
                  </v-chip>
                </template>
              </v-expansion-panel-header>
              <v-expansion-panel-content class="font-weight-light text-start">
                <a :href="item.url" target="_blank">{{ item.url }}</a>
              </v-expansion-panel-content>
            </v-expansion-panel>
          </v-expansion-panels>
        </v-flex>

        <v-flex class="d-flex justify-space-between mb-5" md10 offset-md-1 offset-xl-3 xl6 xs12>
          <h1 class="headline">Unidade Descentralizada</h1>
        </v-flex>

        <v-flex class="d-flex justify-center mb-3" md6 offset-md-3 offset-xl-4 xl4 xs12>
          <v-select
            :disabled="cLocal.sum > 0"
            :hint="'Em ' + unity.local"
            :items="unities"
            append-outer-icon="location_city"
            item-text="name"
            label="Selecione a unidade..."
            persistent-hint
            return-object
            solo
            v-model="unity"
            v-on:change="changeUnity()"
          ></v-select>
        </v-flex>

        <v-flex class="d-flex justify-space-between" md10 offset-md-1 offset-xl-3 v-if="cLocal.sum > 0" xl6 xs12>
          <v-progress-linear indeterminate></v-progress-linear>
        </v-flex>

        <v-flex class="d-flex justify-space-between mb-12" md10 offset-md-1 offset-xl-3 v-if="unity" xl6 xs12>
          <v-expansion-panels>
            <v-expansion-panel :key="i" v-for="(item,i) in sLocal[unity.domain]">
              <v-expansion-panel-header disable-icon-rotate>
                {{ item.name }}
                <template v-slot:actions>
                  <v-chip
                    class="py-0"
                    color="blue darken-1"
                    label
                    text-color="white"
                    v-if="item.status === undefined || item.status === null"
                  >
                    Verificando...
                    <v-icon right style="color: #fff;">hourglass_empty</v-icon>
                  </v-chip>
                  <v-chip class="py-0" :color="item.status.color" label text-color="white" v-else>
                    {{ item.status.label }}
                    <v-icon right style="color: #fff;">{{ item.status.icon }}</v-icon>
                  </v-chip>
                </template>
              </v-expansion-panel-header>
              <v-expansion-panel-content class="font-weight-light text-start">
                <a :href="item.url" target="_blank">{{ item.url }}</a>
              </v-expansion-panel-content>
            </v-expansion-panel>
          </v-expansion-panels>
        </v-flex>

        <v-flex
          class="d-flex justify-center mb-12"
          md10
          offset-md-1
          offset-xl-3
          v-if="sLocal[unity.domain] === undefined"
          xl6
          xs12
        >
          <v-alert type="error">Não há serviços digitais locais cadastrados para a {{ unity.name }}.</v-alert>
        </v-flex>
      </v-layout>

      <v-dialog max-width="290" persistent v-model="dialog">
        <v-card>
          <v-card-text>
            <p class="text-center pt-4">
              <v-icon color="red" large>cloud_off</v-icon>
            </p>Você está sem conexão com a internet! Ela é necessária para utilizar este serviço. Por favor, verifique sua rede de dados.
          </v-card-text>
          <v-card-actions>
            <div class="flex-grow-1"></div>
            <v-btn @click="refresh()" color="green darken-1" text>Tentar Novamente</v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>

      <v-footer absolute class="font-weight-light" color="blue-grey darken-1" dark padless>
        <v-col class="text-center" cols="12">
          &copy;{{ new Date().getFullYear() }} &bull;
          <a
            href="https://www.embrapa.br/gado-de-corte"
            style="color: #FFF; text-decoration: none;"
            target="_blank"
          >
            <strong>Embrapa Gado de Corte</strong>
          </a>

          <v-chip class="ml-2 px-1" color="white" label light small>
            <v-avatar class="mr-2">
              <v-icon>update</v-icon>
            </v-avatar>
            <b>v{{ version }}</b>
          </v-chip>
        </v-col>
      </v-footer>
    </v-container>
  </v-app>
</template>

<script>
import _ from 'lodash'

import unities from './settings/unities.json'

import external from './settings/services/external.json'
import corporate from './settings/services/corporate.json'
import local from './settings/services/local.json'

import pck from '../package.json'

export default {
  data: () => ({
    dialog: false,
    unity: null,
    unities: [],
    sExternal: external,
    sCorporate: corporate,
    sLocal: local,
    cCorporate: { sum: 0 },
    cLocal: { sum: 0 },
    version: pck.version,
    status: {
      OK: { label: 'Operacional', color: 'green', icon: 'done' },
      DOWN: { label: 'Inacessível', color: 'red', icon: 'error' },
      TIMEOUT: { label: 'Timeout', color: 'orange', icon: 'timer_off' },
      SLOW: { label: 'Lentidão', color: 'brown', icon: 'access_time' }
    }
  }),
  localStorage: {
    unity: {
      type: Object,
      default: {
        domain: 'cnpgc',
        name: 'Embrapa Gado de Corte',
        local: 'Campo Grande - MS'
      }
    }
  },
  beforeMount () {
    this.unity = this.$localStorage.get('unity')

    this.unities = _.orderBy(unities, ['name'], ['asc'])
  },
  mounted () {
    this.refresh()
  },
  methods: {
    online () {
      if (typeof window.navigator.onLine === 'undefined') {
        return true
      } else {
        return window.navigator.onLine
      }
    },
    refresh () {
      if (this.online()) {
        this.dialog = false

        this.sCorporate.forEach(item => {
          this.cCorporate.sum++

          item.status = null

          console.log('Checking ' + item.url)

          this.check(item, this.cCorporate)
        })

        this.checkUnitiesServices()
      } else {
        this.dialog = true
      }
    },
    check (item, counter) {
      const start = new Date().getTime()

      this.timeout(10000, fetch(item.url, { method: 'HEAD', mode: 'no-cors' })).then(response => {
        const time = new Date().getTime() - start

        console.log('Ok to ' + item.url + ' in ' + time + ' seconds!')

        item.status = time < 5000 ? this.status.OK : this.status.SLOW
      }).catch(error => {
        const time = new Date().getTime() - start

        console.log('Failed to ' + item.url + ' in ' + time + ' seconds! - ' + JSON.stringify(error))

        item.status = time < 10000 ? this.status.DOWN : this.status.TIMEOUT
      }).finally(() => {
        counter.sum--
        this.$forceUpdate()
      })
    },
    timeout (ms, promise) {
      return new Promise((resolve, reject) => {
        const id = setTimeout(() => {
          reject(new Error('TIMEOUT'))
        }, ms)

        promise.then(
          (res) => {
            clearTimeout(id)
            resolve(res)
          },
          (err) => {
            clearTimeout(id)
            reject(err)
          }
        )
      })
    },
    reload () {
      location.reload()
    },
    changeUnity () {
      this.$localStorage.set('unity', JSON.parse(JSON.stringify(this.unity)))

      this.$forceUpdate()

      this.checkUnitiesServices()
    },
    checkUnitiesServices () {
      if (this.sLocal[this.unity.domain] !== undefined) {
        this.sLocal[this.unity.domain].forEach(item => {
          this.cLocal.sum++

          item.status = null

          console.log('Checking ' + item.url)

          this.check(item, this.cLocal)
        })
      }
    },
    loading () {
      return this.cCorporate > 0 || this.cLocal > 0
    }
  }
}
</script>
