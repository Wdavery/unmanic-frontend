<template>
  <!-- NOTE: notice the 'directoryBrowserDialogRef' here -->
  <q-dialog
    ref="directoryBrowserDialogRef"
    :maximized="$q.platform.is.mobile"
    transition-show="slide-left"
    transition-hide="slide-right"
    full-height
    position="right"
    @hide="onDialogHide">

    <q-card
      v-touch-swipe.touch.right="hide"
      style="width: 700px; max-width: 90vw;">
      <q-card-section class="bg-grey-2">
        <div class="row items-center no-wrap">
          <div class="col">
            <div class="text-h6 text-blue-10">
              {{ dialogHeader }}
            </div>
          </div>

          <div class="col-auto">
            <q-btn
              color="grey-7"
              dense
              round
              flat
              icon="arrow_forward" v-close-popup>
              <q-tooltip class="bg-white text-primary">{{ $t('tooltips.close') }}</q-tooltip>
            </q-btn>
          </div>
        </div>
      </q-card-section>

      <q-card-section class="q-pt-none">

        <div class="q-pa-md">

          <q-card flat>

            <q-card-section>
              <q-input
                disable
                readonly
                borderless
                v-model="this.currentPath">
                <template v-slot:before>
                  <q-icon name="folder_open"/>
                </template>
              </q-input>
            </q-card-section>

            <q-separator/>

            <q-card-section class="q-pa-none">

              <q-list bordered padding>
                <q-item
                  v-for="(directory, index) in directories"
                  v-bind:key="index"
                  clickable v-ripple
                  @click="fetchDirectoryListing(directory.full_path)">

                  <q-item-section avatar>
                    <q-icon color="primary" name="folder_open"/>
                  </q-item-section>

                  <q-item-section>
                    <q-item-label>{{ directory.name }}</q-item-label>
                  </q-item-section>

                </q-item>
              </q-list>

            </q-card-section>

          </q-card>

        </div>

      </q-card-section>

    </q-card>

  </q-dialog>
</template>

<script>
/*
dialogHeader  - Sets the dialog header
initialPath   - Will populate the initial selected file or directory
listType      - The type of content to list:
                  - files: List only files
                  - directories: List only directories
                  - all: List both directories and files
*/

import axios from "axios";
import { getUnmanicApiUrl } from "src/js/unmanicGlobals";
import { ref } from "vue";

export default {
  name: 'DirectoryBrowserDialog',
  props: {
    dialogHeader: {
      type: String,
      default: ' --- header --- '
    },
    initialPath: {
      type: String,
      default: ' --- initial path --- '
    },
    listType: {
      type: String,
      default: ' --- list type --- '
    }
  },
  emits: [
    // REQUIRED
    'ok', 'hide', 'path'
  ],
  methods: {
    // following method is REQUIRED
    // (don't change its name --> "show")
    show() {
      this.$refs.directoryBrowserDialogRef.show();
      this.fetchDirectoryListing(this.initialPath);
    },

    // following method is REQUIRED
    // (don't change its name --> "hide")
    hide() {
      this.$refs.directoryBrowserDialogRef.hide();
    },

    onDialogHide() {
      // required to be emitted
      // when QDialog emits "hide" event
      this.$emit('ok', { selectedPath: this.currentPath })
      this.$emit('hide')
    },

    fetchDirectoryListing: function (path) {
      this.currentPath = path;

      // Fetch from server
      let data = {
        current_path: this.currentPath,
        list_type: this.listType,
      }
      axios({
        method: 'post',
        url: getUnmanicApiUrl('v2', 'filebrowser/list'),
        data: data
      }).then((response) => {
        this.directories = response.data.directories;
        this.files = response.data.files;
      });
    },
  },
  watch: {
    initialPath(value) {
      if (value.length > 0) {
        this.currentPath = this.initialPath;
      }
    }
  },
  data: function () {
    return {
      maximizedToggle: true,
      currentPath: ref(null),
      directories: ref([]),
      files: ref([]),
    }
  }
}
</script>
