<template>
  <div id="customize">
    <breadcrumbs :name="$t('sidebar.customize')" />

    <div class="px-4 py-3 mb-8 bg-white rounded-lg shadow-md dark:bg-gray-800" v-if="!currentBotID && currentGuildId && !tokenLock">
      <p class="text-xl mb-4">{{ $t("customize.new_instance") }}</p>
      <div class="flex">
        <label class="block w-1/2 text-sm">
          <span class="text-gray-700 dark:text-gray-400">{{
              $t("customize.token")
            }}</span>
          <input
              class="block w-full mt-1 text-sm dark:border-gray-600 dark:bg-gray-700 focus:border-blue-400 focus:outline-none focus:shadow-outline-blue dark:text-gray-300 dark:focus:shadow-outline-gray form-input"
              v-model="currentToken"
          />
        </label>
        <div
            class="ml-10 block relative inline-block w-12 mr-2 mt-2 align-middle select-none transition duration-200 ease-in"
        >
          </div>
        </div>
      <button
          type="button"
          name="reset"
          class="mt-4 w-full inline-flex justify-center rounded-md border border-gray-500 shadow-sm px-4 py-2 text-base font-medium focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500 sm:w-auto sm:text-sm"
          @click="onTokenSubmit"
      >
        {{ $t("customize.submit") }}
      </button>
    </div>

    <div class="flex flex-wrap px-4 py-3 mb-8 bg-white rounded-lg shadow-md dark:bg-gray-800" v-if="currentBotID || !currentGuildId">
      <div style="max-width: 200px;" class="flex-auto">
        <p class="text-xl mb-4">{{ $t("customize.avatar") }}</p>
          <div class="mb-0 border-4 flex justify-center px-4 py-3 bg-white rounded-mg shadow-lg p-3 mb-5 dark:bg-gray-700">
            <img
                class="cursor-pointer rounded-full shadow-lg center m-auto"
                :src="currentAvatar"
                :key="currentAvatar"
            >
          </div>
          <div class="flex justify-center bottom-0">
            <label>
              <input
                  type="file"
                  accept="image/jpeg, image/png"
                  style="display: none"
                  v-bind:disabled="!currentBotID && !currentGuildId"
                  @change="onBotAvatarChange($event.target.files);"
              >
              <a class="cursor-pointer hover:underline">{{ $t("customize.changeavatar") }}</a>
            </label>
          </div>
      </div>
      <div style="max-width: 40%; width: auto" class="pl-10">
        <p class="text-xl mb-5">{{ $t("customize.name") }}</p>
        <label>
          <input
              style="width:100%"
              v-bind:disabled="!currentBotID && !currentGuildId"
              class="text-sm dark:border-gray-600 dark:bg-gray-700 focus:border-blue-400 focus:outline-none focus:shadow-outline-blue dark:text-gray-300 dark:focus:shadow-outline-gray form-input"
              maxlength="32"
              v-model="currentName"
              @change="onNameChange()"
          />
        </label>
        <div class="flex-auto">
          <p class="text-xl mt-4">{{ $t("customize.activity") }}</p>
          <select
              :style="{ backgroundImage: 'url(' + require('@/assets/dropDownArrow.svg') + ')', width: 'auto' }"
              class="drop_down mt-3 p-2 cursor-pointer text-sm dark:border-gray-600 dark:bg-gray-700 focus:border-blue-400 focus:outline-none focus:shadow-outline-blue dark:text-gray-300 dark:focus:shadow-outline-gray form-input"
              v-bind:disabled="!currentBotID && !currentGuildId"
              v-model="activityOption"
          >
            <option v-for="option in activityOptionsList" v-bind:value="option.value" :key="option.key">
              {{ option.text }}
            </option>
          </select>
          <label class="mt-3 flex">
            <input
                style="width:100%"
                v-bind:disabled="!currentBotID && !currentGuildId"
                class="text-sm dark:border-gray-600 dark:bg-gray-700 focus:border-blue-400 focus:outline-none focus:shadow-outline-blue dark:text-gray-300 dark:focus:shadow-outline-gray form-input"
                maxlength="32"
                v-model="currentActivity"
                @change="onActivityChange()"
            />
          </label>
        </div>
        <div style="flex-basis: 100%;" class="flex bottom-0 mt-4">
          <button
              type="button"
              name="invite"
              class="rounded-md border border-gray-500 shadow-sm px-4 py-2 text-base font-medium focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500 sm:w-auto sm:text-sm"
              @click="onInvite()"
          >
            {{ $t("customize.invite") }}
          </button>
          <button
              type="button"
              name="delete"
              class="rounded-md border border-red-700 shadow-sm px-4 py-2 text-base font-medium focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500 sm:w-auto sm:text-sm"
              @click="onDelete()"
          >
            {{ $t("customize.delete") }}
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<style>
.drop_down {
  -webkit-appearance: none;
  -moz-appearance: none;
  background-repeat: no-repeat;
  background-position-x: 90%;
  background-position-y: 50%;
}
.drop_down::-ms-expand {
  display: none;
}
</style>

<script>
import { mapState, mapGetters } from "vuex";
import Breadcrumbs from "@/components/Breadcrumbs";

import config from "@/config";
import fetch from "@/utils/fetch";
import queryString from "@/utils/queryString";

export default {
  name: "Customize",
  components: {
    Breadcrumbs,
  },
  computed: {
    ...mapState(["currentGuildId", "token"]),
    ...mapGetters({ auth: "ifAuthenticated" }),
  },
  data() {
    return {
      currentAvatar: "",
      currentName: "",
      currentToken: "",
      currentBotID: "",
      currentActivity: "",
      activityOption: "playing",
      activityOptionsList: [
        { value: 'playing', text: 'Playing', key: 1},
        { value: 'listening', text: 'Listening', key: 2},
        { value: 'competing', text: 'Competing', key: 3},
        { value: 'watching', text: 'Watching', key: 4},

      ],
      avatarTimeout: 0,
      nameTimeout: 0,
      tokenLock: 0,
    };
  },
  methods: {
    onActivityChange() {
      if (!this.currentGuildId || !this.auth || !this.currentToken) return;
      fetch(
          `${config.botApi}/mappings/bot_activity${queryString({
            guildId: this.currentGuildId,
          })}`,
          {
            method: "post",
            headers: {
              authorization: this.token,
            },
            body: JSON.stringify({ activity_type: this.activityOption, activity_text: this.currentActivity }),
          }
      )
          .then((res) => res.json())
          .then((response) => {
            if (response.success) {
              this.$toast.success("Bot's activity has been changed");
            } else
              this.$toast.error("An error was encountered. Please try again");
          })
          .catch(() => this.$toast.warning("Failed to change bot's activity. Are you offline?"));
    },
    onDelete() {
      if (!this.currentGuildId || !this.auth || !this.currentToken) return;

      fetch(
          `${config.botApi}/mappings/bot_instance${queryString({
            guildId: this.currentGuildId,
          })}`,
          {
            method: "delete",
            headers: {
              authorization: this.token,
            },
            body: JSON.stringify({ bot_instance: this.currentToken }),
          }
      )
          .then((res) => res.json())
          .then((response) => {
            if (response.deleted) {
              this.currentBotID = ''
              this.currentToken = ''
              this.$toast.success("Bot has been Deleted");
            } else
              this.$toast.error("An error was encountered. Please try again");
          })
          .catch(() => this.$toast.warning("Failed to delete bot. Are you offline?"));
    },
    onInvite() {
      if (!this.currentGuildId || !this.auth || !this.currentToken) return;
      window.open(`https://discord.com/oauth2/authorize?client_id=${this.currentBotID}&permissions=268438560&scope=bot`, "_blank");
    },
    onNameChange() {
      if (this.nameTimeout == 1) {
        return this.$toast.error("You are changing the bot's name too many times too fast.");
      }

      if (!this.currentGuildId || !this.auth || !this.currentToken) return;
      fetch(
          `${config.botApi}/mappings/bot_name${queryString({
            guildId: this.currentGuildId,
          })}`,
          {
            method: "post",
            headers: {
              authorization: this.token,
            },
            body: JSON.stringify({ bot_name: this.currentName }),
          }
      )
          .then((res) => res.json())
          .then((response) => {
            if (response.name_timeout) {
              this.nameTimeout = response.name_timeout;
            } else {
              this.nameTimeout = 0
            }

            if (response.bot_name) {
              this.currentName = response.bot_name;
              if (this.nameTimeout) {
                return this.$toast.error("You are changing the bot's name too many times too fast.");
              }
              this.$toast.success("Bot name has been changed");
            } else
              this.$toast.error("An error was encountered. Please try again");
          })
          .catch(() => this.$toast.warning("Failed to change bot name. Are you offline?"));
    },
    onTokenSubmit() {
      if (!this.currentGuildId || !this.auth || !this.currentToken) return;

      fetch(
          `${config.botApi}/mappings/bot_instance${queryString({
            guildId: this.currentGuildId,
          })}`,
          {
            method: "post",
            headers: {
              authorization: this.token,
            },
            body: JSON.stringify({ bot_instance: this.currentToken }),
          }
      )
          .then((res) => res.json())
          .then((response) => {
            if (response.bot_instance) {
              this.currentToken = response.bot_instance;
              this.$toast.success("Bot instance has been created");
            } else
              this.$toast.error("An error was encountered. Please try again");
          })
          .catch(() => this.$toast.warning("Failed to create bot instance. Are you offline?"));

      this.botIDWait(0)

    },
    botIDWait(x) {
      if (x === 5) {
        this.tokenLock = 0
        return '';
      }
      this.tokenLock = 1
      fetch(`${config.botApi}/mappings/bot_instance/${this.currentGuildId}`, {
        headers: {
          authorization: this.token,
        },
      })
          .then((res) => res.json())
          .then((response) => {
            if (response.bot_id) {
              this.currentBotID = response.bot_id;
              // setTimeout(() => {
                this.tokenLock = 0
                this.refresh(this.currentGuildId)
              // }, )
              return ''
            } else {
              setTimeout(() => {
                this.botIDWait(x + 1)
              }, 5000)
            }
          })
    },
    onBotAvatarChange(files) {
      if (!this.currentGuildId || !this.auth) return;

      if (this.avatarTimeout == 1) {
        return this.$toast.error("You are changing the bot's avatar too many times too fast.");
      }

      let file = files[0];
      let formData = new FormData();
      formData.append('file', file);

      let fileReader = new FileReader()
      fileReader.addEventListener('load', () => {
        this.currentAvatar = fileReader.result

        fetch(
            `${config.botApi}/mappings/bot_avatar${queryString({
              guildId: this.currentGuildId,
            })}`,
            {
              method: "post",
              headers: {
                authorization: this.token,
              },
              body: JSON.stringify({ bot_avatar: this.currentAvatar }),
            }
        )
            .then((res) => res.json())
            .then((response) => {
              if (response.avatar_timeout) {
                this.avatarTimeout = response.avatar_timeout
                return this.$toast.error("You are changing the bot's avatar too many times too fast.");
              } else {
                this.avatarTimeout = 0
              }

              if (response.guildId) {
                this.$toast.success("Bot avatar has been updated");
              } else
                this.$toast.error("An error was encountered. Please try again");
            })
            .catch(() => this.$toast.warning("Failed to change bot avatar. Are you offline?"));
      })
      fileReader.readAsDataURL(file)
    },
    refresh(val) {
      if (!this.auth) return;

      this.currentToken = ''

      fetch(`${config.botApi}/mappings/bot_instance/${val}`, {
        headers: {
          authorization: this.token,
        },
      })
          .then((res) => res.json())
          .then((response) => {
            if (response.bot_id) {
              this.currentToken = response.bot_instance;
              this.currentBotID = response.bot_id;
            } else {
              this.currentBotID = ''
            }
          })

      fetch(`${config.botApi}/mappings/bot_avatar/${val}`, {
        headers: {
          authorization: this.token,
        },
      })
          .then((res) => res.json())
          .then((response) => {
            if (response.avatar_timeout) {
              this.avatarTimeout = response.avatar_timeout
            } else {
              this.avatarTimeout = 0
            }

            if (response.bot_avatar) {
              this.currentAvatar = response.bot_avatar;
            } else {
              this.currentAvatar = "https://rallybot.app/img/space.5424f731.png"
            }
          })

      fetch(`${config.botApi}/mappings/bot_name/${val}`, {
        headers: {
          authorization: this.token,
        },
      })
          .then((res) => res.json())
          .then((response) => {
            if (response.name_timeout) {
              this.nameTimeout = response.name_timeout;
            } else {
              this.nameTimeout = 0
            }

            if (response.bot_name) {
              this.currentName = response.bot_name;
            }
          })
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