<template>
  <div>
    <v-card
      variant="flat"
      :img="imgGradient"
      style="margin-top: -10px; z-index: 0"
      height="30vh"
      max-height="500px"
      min-height="340px"
    >
      <!-- loading animation -->
      <v-progress-linear
        v-if="!item"
        indeterminate
      />
      <v-img
        width="100%"
        height="100%"
        cover
        class="background-image"
        :src="fanartImage"
        :gradient="
          $vuetify.theme.current.dark
            ? 'to bottom, rgba(0,0,0,.90), rgba(0,0,0,.75)'
            : 'to bottom, rgba(255,255,255,.90), rgba(255,255,255,.75)'
        "
      />
      <v-layout
        v-if="item"
        style="
          margin: 0;
          position: absolute;
          top: 50%;
          -ms-transform: translateY(-50%);
          transform: translateY(-50%);
          padding-left: 15px;
          align-items: center;
          padding-right: 15px;
        "
      >
        <!-- left side: cover image -->
        <div
          v-if="!$vuetify.display.mobile"
          xs5
          pa-5
          style="
            width: 230px;
            height: 230px;
            margin-top: 15px;
            margin-bottom: 15px;
            margin-right: 24px;
          "
        >
          <div
            v-if="
              item.media_type &&
                (item.media_type == MediaType.ARTIST || 'owner' in item)
            "
          >
            <v-avatar size="192">
              <MediaItemThumb
                :item="item"
                height="230px"
                width="230px"
              />
            </v-avatar>
          </div>
          <div v-else>
            <MediaItemThumb
              width="230px"
              :item="item"
            />
          </div>
        </div>

        <div>
          <!-- Main title -->
          <v-card-title>
            {{ item.name }}
          </v-card-title>

          <!-- other details -->
          <div style="padding-bottom: 10px">
            <!-- version -->
            <v-card-subtitle
              v-if="'version' in item && item.version"
              class="caption"
            >
              {{ item.version }}
              <!-- explicit icon -->
              <v-tooltip location="bottom">
                <template #activator="{ props }">
                  <v-icon
                    v-if="parseBool(item.metadata.explicit || false)"
                    v-bind="props"
                    class="listitem-action"
                    icon="mdi-alpha-e-box"
                    width="35"
                  />
                </template>
                <span>{{ $t("tooltip.explicit") }}</span>
              </v-tooltip>
            </v-card-subtitle>

            <!-- item artists -->
            <v-card-subtitle
              v-if="'artists' in item && item.artists"
              class="title accent--text"
            >
              <v-icon
                style="margin-left: -3px; margin-right: 3px"
                small
                color="primary"
                icon="mdi-account-music"
              />
              <span
                v-for="(artist, artistindex) in item.artists"
                :key="artist.item_id"
              >
                <a
                  style="color: accent"
                  @click="artistClick(artist)"
                >{{
                  artist.name
                }}</a>
                <span
                  v-if="artistindex + 1 < item.artists.length"
                  :key="artistindex"
                  style="color: accent"
                >{{ " / " }}</span>
              </span>
            </v-card-subtitle>

            <!-- album artist -->
            <v-card-subtitle
              v-if="'artist' in item && item.artist"
              class="title"
            >
              <v-icon
                style="margin-left: -3px; margin-right: 3px"
                small
                color="primary"
                icon="mdi-account-music"
              />
              <a @click="artistClick((item as Album).artist)">{{
                item.artist.name
              }}</a>
            </v-card-subtitle>

            <!-- playlist owner -->
            <v-card-subtitle
              v-if="'owner' in item && item.owner"
              class="title"
            >
              <v-icon
                color="primary"
                style="margin-left: -3px; margin-right: 3px"
                small
                icon="mdi-account-music"
              />
              <a style="color: primary">{{ item.owner }}</a>
            </v-card-subtitle>

            <v-card-subtitle v-if="'album' in item && item.album">
              <v-icon
                color="primary"
                style="margin-left: -3px; margin-right: 3px"
                small
                icon="mdi-album"
              />
              <a
                style="color: secondary"
                @click="albumClick((item as Track)?.album)"
              >{{ item.album.name }}</a>
            </v-card-subtitle>
          </div>

          <!-- play/info buttons -->
          <div style="display: flex; margin-left: 14px; padding-bottom: 10px">
            <v-menu location="bottom">
              <!-- eslint-disable vue/no-template-shadow -->
              <template #activator="{ props }">
                <!-- eslint-enable vue/no-template-shadow -->
                <v-btn
                  color="primary"
                  v-bind="props"
                  prepend-icon="mdi-play-circle"
                  :disabled="
                    !store.selectedPlayer?.available ||
                      store.blockGlobalPlayMenu
                  "
                >
                  {{ $t("play") }}
                </v-btn>
              </template>

              <v-card min-width="300">
                <v-list
                  lines="one"
                  density="comfortable"
                >
                  <!-- play now -->
                  <v-list-item
                    v-for="menuItem in getPlayMenuItems([item])"
                    :key="menuItem.label"
                    :title="$t(menuItem.label, menuItem.labelArgs)"
                    @click="menuItem.action ? menuItem.action() : ''"
                  >
                    <template #prepend>
                      <v-avatar style="padding-right: 10px">
                        <v-icon :icon="menuItem.icon" />
                      </v-avatar>
                    </template>
                  </v-list-item>
                </v-list>
              </v-card>
            </v-menu>

            <v-btn
              v-if="!$vuetify.display.mobile && !item.in_library"
              style="margin-left: 10px"
              color="primary"
              tile
              prepend-icon="mdi-heart-outline"
              @click="api.addItemsToLibrary([item!])"
            >
              {{ $t("add_library") }}
            </v-btn>
            <v-btn
              v-if="!$vuetify.display.mobile && item.in_library"
              style="margin-left: 10px"
              color="primary"
              tile
              prepend-icon="mdi-heart"
              @click="api.removeItemsFromLibrary([item!])"
            >
              {{ $t("remove_library") }}
            </v-btn>
          </div>

          <!-- Description/metadata -->
          <v-card-subtitle
            v-if="shortDescription"
            class="body-2 justify-left"
            style="padding-bottom: 10px; white-space: pre-line; cursor: pointer"
            @click="showFullInfo = !showFullInfo"
          >
            <!-- eslint-disable vue/no-v-html -->
            <div v-html="shortDescription" />
            <!-- eslint-enable vue/no-v-html -->
          </v-card-subtitle>

          <!-- genres/tags -->
          <div
            v-if="item && item.metadata.genres"
            class="justify-center"
            style="margin-left: 15px; padding-bottom: 20px"
          >
            <v-chip
              v-for="tag of item.metadata.genres.slice(
                0,
                $vuetify.display.mobile ? 15 : 25
              )"
              :key="tag"
              color="blue-grey lighten-1"
              style="margin-right: 5px; margin-bottom: 5px"
              small
              outlined
            >
              {{ tag }}
            </v-chip>
          </div>
        </div>
      </v-layout>
      <v-layout
        v-if="item"
        style="z-index: 800; height: 100%; padding-left: 15px"
      >
        <!-- provider icons -->
        <div style="position: absolute; float: right; right: 15px; top: 15px">
          <provider-icons
            :provider-mappings="item.provider_mappings"
            :height="25"
            :enable-details="true"
            :enable-preview="item.media_type == MediaType.TRACK"
          />
        </div>
      </v-layout>
    </v-card>
    <v-dialog
      v-model="showFullInfo"
      width="auto"
    >
      <v-card>
        <!-- eslint-disable vue/no-v-html -->
        <!-- eslint-disable vue/no-v-text-v-html-on-component -->
        <v-card-text v-html="fullDescription" />
        <!-- eslint-enable vue/no-v-html -->
        <!-- eslint-enable vue/no-v-text-v-html-on-component -->
        <v-card-actions>
          <v-btn
            color="primary"
            block
            @click="showFullInfo = false"
          >
            {{ $t("close") }}
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </div>
</template>

<script setup lang="ts">
import ProviderIcons from "./ProviderIcons.vue";
import { store } from "@/plugins/store";
import { useDisplay } from "vuetify";
import { api } from "@/plugins/api";
import { ImageType, Track, MediaType } from "@/plugins/api/interfaces";
import type {
  Album,
  Artist,
  ItemMapping,
  MediaItemType,
} from "@/plugins/api/interfaces";
import { computed, ref, watch, onBeforeUnmount } from "vue";
import MediaItemThumb from "./MediaItemThumb.vue";
import { getImageThumbForItem } from "./MediaItemThumb.vue";
import { useRouter } from "vue-router";
import { parseBool } from "../utils";
import {
  getPlayMenuItems,
  getContextMenuItems,
} from "./MediaItemContextMenu.vue";

// properties
export interface Props {
  item?: MediaItemType;
}
const compProps = defineProps<Props>();
const showFullInfo = ref(false);
const fanartImage = ref();
const { mobile } = useDisplay();

const imgGradient = new URL("../assets/info_gradient.jpg", import.meta.url)
  .href;

const router = useRouter();

watch(
  () => compProps.item,
  async (val) => {
    if (val) {
      store.topBarTitle = val.name;

      fanartImage.value =
        (getImageThumbForItem(compProps.item, ImageType.FANART)) ||
        (getImageThumbForItem(compProps.item, ImageType.THUMB));

      store.topBarContextMenuItems = getContextMenuItems([val], val);
    }
  },
  { immediate: true }
);

onBeforeUnmount(() => {
  store.topBarContextMenuItems = [];
});

const albumClick = function (item: Album | ItemMapping) {
  // album entry clicked
  router.push({
    name: "album",
    params: {
      itemId: item.item_id,
      provider: item.provider,
    },
  });
};
const artistClick = function (item: Artist | ItemMapping) {
  // album entry clicked
  console.log("artistClick", item);
  router.push({
    name: "artist",
    params: {
      itemId: item.item_id,
      provider: item.provider,
    },
  });
};
const fullDescription = computed(() => {
  let desc = "";
  if (!compProps.item) return "";
  if (compProps.item.metadata && compProps.item.metadata.description) {
    desc = compProps.item.metadata.description;
  } else if (compProps.item.metadata && compProps.item.metadata.copyright) {
    desc = compProps.item.metadata.copyright;
  } else if ("artists" in compProps.item) {
    compProps.item.artists.forEach(function (artist: Artist | ItemMapping) {
      if ("metadata" in artist && artist.metadata.description) {
        desc = artist.metadata.description;
      }
    });
  }
  desc = desc.replace("\r\n", "<br /><br /><br />");
  desc = desc.replace("\r", "<br /><br />");
  desc = desc.replace("\n", "<br /><br />");
  return desc;
});
const shortDescription = computed(() => {
  const maxChars = mobile.value ? 160 : 260;
  if (fullDescription.value.length > maxChars) {
    return fullDescription.value.substring(0, maxChars) + "...";
  }
  return fullDescription.value;
});
</script>

<style>
.background-image {
  position: absolute;
}

.background-image .v-img__img--cover {
  object-position: 50% 20%;
}
</style>
