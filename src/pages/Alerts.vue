<template>
  <div id="settings" v-if="currentGuildId">
    <breadcrumbs :name="$t('sidebar.alerts')" />

    <div class="px-4 py-3 mb-8 bg-white rounded-lg shadow-md dark:bg-gray-800"
         v-if="!defaultCoin">
      {{ $t('alerts.nodefaultcoin') }}
      {{ defaultCoin }}:

    </div>

    <div class="px-4 py-3 mb-8 bg-white rounded-lg shadow-md dark:bg-gray-800"
    v-if="defaultCoin">
      <p class="text-xl mb-4">{{ $t("alerts.customize") }}</p>
      <div>
        <div class="flex-auto">
          <p class="text-md mb-4 mt-6">{{ $t("alerts.alerts") }}</p>
          <div class="mt-3" v-for="alert in alertList" v-bind:key="alert">
            <div class="inline-block">
              <label
                  class="cursor-pointer"
                  @click="toggleShow(alert)"
              >
                <div v-bind:class="{ 'rotate90': show !== alert }" class="inline-block pb-1">
                  <svg width='11px' height='7px' viewBox='0 0 11 7' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'>
                    <g id='Symbols' stroke='none' stroke-width='1' fill='none' fill-rule='evenodd'>
                      <g id='header' transform='translate(-1275.000000, -41.000000)' fill='#FFFFFF'>
                        <g id='Group-3' transform='translate(1151.000000, 22.000000)'>
                          <path d='M124.172823,19.7967723 C123.83576,19.3567271 124.006153,19 124.558414,19 L134.247416,19 C134.797435,19 134.963005,19.3491139 134.61487,19.7827005 L129.893835,25.6625512 C129.546755,26.0948247 128.99033,26.0861517 128.655084,25.6484793 L124.172823,19.7967723 Z' id='switcher'></path>
                        </g>
                      </g>
                    </g>
                  </svg>
                </div>
                <div class="pl-1 pr-1 inline-block hover:underline">
                  {{ $t(`alerts.${alert}`) }}
                </div>
              </label>
            </div>

            <div class="mb-10 ml-8 mt-6" v-if="show === alert">
              <div class="mb-3">
                <label>
                  <input
                      type="checkbox"
                      v-model="settings[alert].enabled"
                  />
                  {{ $t(`alerts.enabled`) }}
                </label>
              </div>
              <table class="whitespace-no-wrap border-2 border-gray-900">
                <thead>
                <tr class="text-xs font-semibold tracking-wide text-left text-gray-500 uppercase border-b dark:border-gray-700 bg-gray-50 dark:text-gray-400 dark:bg-gray-800">
                  <th class="px-4 py-3">{{ $t('alerts.nr') }}</th>
                  <th class="px-4 py-3">{{ $t('alerts.channel') }}</th>
                  <th
                    v-for="(setting, index) in settings[alert].settings" v-bind:key="index"
                    class="px-4 py-3"
                  >
                    {{ $t(`alerts.${setting}`) }}
                  </th>
                  <th class="px-4 py-3" >{{ $t('alerts.action') }}</th>
                  <th class="px-4 py-3" >{{ $t('alerts.delete') }}</th>
                </tr>
                </thead>

                <tbody class="bg-white divide-y dark:divide-gray-700 dark:bg-gray-800">
                <tr
                  v-for="(instance, instance_index) in settings[alert].instances" v-bind:key="instance_index"
                >
                  <td class="text-center px-4 py-3">
                    {{ instance_index + 1}}
                  </td>
                  <td class="px-4 py-3">
                    <label class="inline-block"
                      v-if="instance.edit"
                    >
                      <input
                          class="block w-full mt-1 text-sm dark:border-gray-600 dark:bg-gray-700 focus:border-blue-400 focus:outline-none focus:shadow-outline-blue dark:text-gray-300 dark:focus:shadow-outline-gray form-input"
                          placeholder="channel-name"
                          maxlength="32"
                          v-model="instance.channel"
                          v-bind:disabled="!instance.edit"
                      />
                    </label>
                    <span v-if="!instance.edit">{{ instance.channel }}</span>
                  </td>
                  <td class="px-4 py-3" v-for="(setting, index) in settings[alert].settings" v-bind:key="index">
                    <label
                        class="inline-block"
                        v-if="instance.edit"
                    >
                      <input
                          class="block w-full mt-1 text-sm dark:border-gray-600 dark:bg-gray-700 focus:border-blue-400 focus:outline-none focus:shadow-outline-blue dark:text-gray-300 dark:focus:shadow-outline-gray form-input"
                          v-bind:disabled="!instance.edit"
                          v-model="instance.settings[setting]"
                          type="number"
                          v-bind:placeholder="'placeholder' in settings[alert] ? settings[alert].placeholder[setting]: 'No Limit'"
                      />
                    </label>
                    <span v-if="!instance.edit">{{ 'default' in settings[alert] && !instance.settings[setting] ? settings[alert].default[setting] : instance.settings[setting] ? instance.settings[setting] : 'No Limit' }}</span>
                  </td>
                  <td class="px-4 py-3">
                    <button
                        type="button"
                        class="w-full inline-flex justify-center rounded-md border border-gray-500 shadow-sm px-4 py-2 text-base font-medium focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500 sm:w-auto sm:text-sm"
                        @click="edit(instance)"
                    >
                      {{ $t(`alerts.${instance.edit ? 'save' : 'edit'}`) }}
                    </button>
                  </td>
                  <td class="px-4 py-3">
                    <button
                        @click="delete settings[alert].instances.splice(instance_index, 1)"
                        type="button"
                        class="w-full inline-flex justify-center rounded-md border border-gray-500 shadow-sm px-4 py-2 text-base font-medium focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500 sm:w-auto sm:text-sm"
                    >
                      {{ $t(`alerts.delete`) }}
                    </button>
                  </td>
                </tr>

                <tr>
                  <td class="px-4 py-3 text-center">
                    {{ settings[alert].instances.length + 1}}
                  </td>
                  <td class="px-4 py-3">
                    <label class="inline-block">
                      <input
                          class="block w-full mt-1 text-sm dark:border-gray-600 dark:bg-gray-700 focus:border-blue-400 focus:outline-none focus:shadow-outline-blue dark:text-gray-300 dark:focus:shadow-outline-gray form-input"
                          placeholder="channel-name"
                          maxlength="32"
                          v-model="new_instance.channel"
                      />
                    </label>
                  </td>
                  <td class="px-4 py-3" v-for="(setting, index) in settings[alert].settings" v-bind:key="index">
                    <label class="inline-block">
                      <input
                          type="number"
                          class="block w-full mt-1 text-sm dark:border-gray-600 dark:bg-gray-700 focus:border-blue-400 focus:outline-none focus:shadow-outline-blue dark:text-gray-300 dark:focus:shadow-outline-gray form-input"
                          v-model="new_instance.settings[setting]"
                          v-bind:placeholder="'placeholder' in settings[alert] ? settings[alert].placeholder[setting]: 'No Limit'"
                      />
                    </label>
                  </td>
                  <td class="px-4 py-3">
                    <button
                        @click="add(alert)"
                        type="button"
                        class="w-full inline-flex justify-center rounded-md border border-gray-500 shadow-sm px-4 py-2 text-base font-medium focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500 sm:w-auto sm:text-sm"
                    >
                      {{ $t(`alerts.add`) }}
                    </button>
                  </td>
                  <td class="px-4 py-3"></td>
                </tr>
                </tbody>
              </table>
            </div>
          </div>
        </div>
      </div>

      <div class="flex mt-3">
        <button
            type="button"
            name="apply"
            class="mt-4 bg-red-500 hover:bg-red-600 focus:outline-none rounded-lg px-6 py-2 text-white font-semibold shadow"
            @click="onApply"
        >
          {{ $t(`alerts.apply`) }}
        </button>

      </div>
    </div>
  </div>

</template>

<style>

.rotate90 {
  transform: rotate(-90deg);
}

.hover_red:hover {
  border: 1px solid #f05252;
}

</style>

<script>
import { mapState, mapGetters } from "vuex";
import Breadcrumbs from "@/components/Breadcrumbs";
import config from "@/config";
import fetch from "@/utils/fetch";
import queryString from "@/utils/queryString";


function initialData() {
  let alertList = ['buy', 'donate', 'transfer', 'convert', 'redeem', 'daily_stats']
  let settings = {}
  alertList.map((el) => {
    settings[el] = {
      'instances': [],
      'enabled': false,
      'settings': ['minamount', 'maxamount']
    }
    if (el === 'daily_stats') {
      settings[el]['settings'] = ['timezone']
      settings[el]['placeholder'] = {}
      settings[el]['default'] = {}
      settings[el]['placeholder']['timezone'] = '-12 - +12'
      settings[el]['default']['timezone'] = '0'
    }
  })

  return {
    'alertList': alertList,
    'settings': settings,
  };
}


export default {
  name: "Alerts",
  components: {
    Breadcrumbs,
  },
  computed: {
    ...mapState(["currentGuildId", "token", "coins", "currentCoin", "defaultCoin"]),
    ...mapGetters({ auth: "ifAuthenticated" }),
  },
  data() {
    let _data = initialData()
    return {
      'show': '',
      'alertList': _data.alertList,
      'settings': _data.settings,
      'new_instance': {
        'channel': '',
        'settings': {}
      }
    }
  },
  methods: {
    edit(instance) {
      if (!('edit' in instance)) {
        instance.edit = false
      }
      instance.edit = !instance.edit
    },
    add(alert) {
      if (!this.auth || !this.currentGuildId) return;
      if (!this.new_instance.channel) {
        this.$toast.warning("Can't add without setting a channel")
      }

      let taken_channels = []
      this.settings[alert].instances.map((el) => taken_channels.push(el.channel))

      if (taken_channels.includes(this.new_instance.channel)) {
        return this.$toast.error("Can't have duplicate channels");
      }

      this.settings[alert].settings.map((el) => {
        if (!this.new_instance.settings[el]) {
          this.new_instance.settings[el] = ''
        }

      })

      this.new_instance['edit'] = false

      this.settings[alert].instances.push(this.new_instance)
      this.new_instance = {
        'channel': '',
        'settings': {}
      }
    },
    toggleShow(alert) {
      if (!this.auth || !this.currentGuildId) return;

      if (this.show === alert) {
        this.show = ""
      } else {
        this.show = alert
      }

      this.new_instance = {
        'channel': '',
        'settings': {}
      }
    },
    onApply() {
      if (!this.auth || !this.currentGuildId) return;
      fetch(`${config.botApi}/mappings/alerts_settings${queryString({
            guildId: this.currentGuildId,
          })}`,
          {
            method: "post",
            headers: {
              authorization: this.token,
            },
            body: JSON.stringify({
              settings: this.settings
            }),
          }
      )
          .then((res) => res.json())
          .then((response) => {
            if (response.guildId) {
              this.$toast.success(`Alerts settings have been updated successfully`);
            } else
              this.$toast.error("An error was encountered. Please try again");
          })
          .catch(() => {
            this.$toast.warning("Failed to update alerts settings. Are you offline?")
          });
    },
    refresh(val) {
      if (!this.auth || !this.currentGuildId) return;

      this.show = ''

      fetch(`${config.botApi}/mappings/alerts_settings/${val}`, {
        headers: {
          authorization: this.token,
        },
      })
          .then((res) => res.json())
          .then((response) => {
            if (response.settings) {
              Object.assign(this.$data.settings, response.settings)
            } else if (!response.guildId) {
              this.$toast.error("An error was encountered. Please try again");
            } else {
              let _data = initialData()
              Object.assign(this.$data, {
                'show': '',
                'alertList': _data.alertList,
                'settings': _data.settings,
                'new_instance': {
                  'channel': '',
                  'settings': {}
                }
              });
            }
          })
          .catch(() => {
            let _data = initialData()
            Object.assign(this.$data, {
              'show': '',
              'alertList': _data.alertList,
              'settings': _data.settings,
              'new_instance': {
                'channel': '',
                'settings': {}
              }
            });
            this.$toast.warning("Failed to load settings. Are you offline?")
          });

    },
  },
  watch: {
    currentGuildId(val) {
      this.refresh(val);
    },
  },
  mounted() {
    this.currentGuildId && this.refresh(this.currentGuildId);
  },
};
</script>