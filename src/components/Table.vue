<template>
  <h1 class="text-6xl text-center font-light p-3" style="color: #D7DAE5">📄 files.dyna.host 📄</h1>
  <div class="container mx-auto">
    <div v-if="getUser()">
      <div class="flex flex-col">
        <div class="-my-2 overflow-x-auto sm:-mx-6 lg:-mx-8">
          <div class="py-2 align-middle md:inline-block min-w-full sm:px-6 lg:px-8">
            <div class="shadow overflow-hidden border-b border-gray-200 sm:rounded-lg" v-if="files.length">
              <table class="min-w-full divide-y divide-gray-200">
                <thead class="bg-gray-50">
                  <tr>
                    <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Name</th>
                    <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider invisible md:visible">Type</th>
                    <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider invisible md:visible">State</th>
                    <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider invisible md:visible">Size</th>
                    <th scope="col" class="relative px-6 py-3"><span class="sr-only"></span></th>
                  </tr>
                </thead>
                <tbody class="bg-white divide-y divide-gray-200">
                  <tr v-for="file in files" :key="file.id" class="hover:bg-gray-100">
                    <td class="px-6 py-4 whitespace-nowrap cursor-pointer" v-on:click="download(file.name, file.id)">
                      <div class="flex items-center">
                        <div class="flex-shrink-0 h-10 w-10">
                          <img class="h-10 w-10" :src="file.img" alt="" />
                        </div>
                        <div class="ml-4">
                          <div class="text-sm font-medium text-gray-900">{{ file.name }}</div>
                          <div class="text-sm text-gray-500">{{ formatTime(file.updated_at) }}</div>
                        </div>
                      </div>
                    </td>
                    <td class="px-6 py-4 whitespace-nowrap invisible md:visible">
                      <div class="text-sm text-gray-900">{{ file.metadata.mimetype }}</div>
                    </td>
                    <td class="px-6 py-4 whitespace-nowrap invisible md:visible">
                      <span class="px-2 inline-flex text-xs leading-5 font-semibold rounded-full bg-green-100 text-green-800">
                        Online
                      </span>
                    </td>
                    <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500 invisible md:visible">
                      {{ formatBytes(file.metadata.size) }}
                    </td>
                    <td class="px-6 py-4 whitespace-nowrap text-right text-sm font-medium cursor-pointer" v-on:click="deleteFile(file.name)">
                      <a href="#" class="text-indigo-600 hover:text-indigo-900" v-if="!file.downloading">Delete</a>
                      <span class="text-red-900" v-if="file.downloading">Delete</span>
                    </td>
                  </tr>
                  <tr>
                    <td class="px-6 py-4 whitespace-nowrap">
                      <div class="flex items-center">
                        <div class="flex-shrink-0 h-10 w-10">
                          <!-- <img class="h-10 w-10 rounded-full" :src="file.image" alt="" /> -->
                        </div>
                        <div class="ml-4">
                          <div class="text-sm font-medium text-gray-900">
                            <input type="file" id="file" ref="file" v-on:change="upload()" />
                          </div>
                        </div>
                      </div>
                    </td>
                    <td colspan="4"></td>
                  </tr>
                </tbody>
              </table>
            </div>
            <div v-else>
              <h3 style="color: #D7DAE5" class="text-center">
                I'm sorry, but <b>{{ getUser().email }}</b> has no access to files.dyna.host
              </h3>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<script lang="ts">
import { supabase } from "../lib/supabase";
import moment from "moment";
import prettyBytes from "pretty-bytes";
var prettyFileIcons = require("pretty-file-icons");
import { defineComponent } from "vue";
import { FileObject } from "@supabase/storage-js";
import { User } from "@supabase/gotrue-js";

export default defineComponent({
  name: "Table",
  async mounted() {
    await this.getUser();
    await this.getFilesInStorage();
  },
  data() {
    return {
      files: new Array<FileObject>()
    };
  },
  methods: {
    /**
     * Inside a browser context, will return the user data, if there is a logged in user.
     */
    getUser(): User | Promise<User | null> | null {
      //If user is still signed in use that session, otherwise init google popup
      const user = supabase.auth.user() ? supabase.auth.user() : this.signInWithGoogle();
      return user;
    },
    /**
     * Sign a user, displays a popup
     */
    async signInWithGoogle(): Promise<User | null> {
      const { user, session, error } = await supabase.auth.signIn({
        provider: "google"
      });
      console.log(user, session, error);
      return user;
    },

    /**
     * Get files from Supabase
     */
    async getFilesInStorage(): Promise<void> {
      console.log("getting files");
      const { data, error } = await supabase.storage.from("files").list("", {
        limit: 100,
        offset: 0,
        sortBy: { column: "created_at", order: "desc" }
      });
      //add the path to fileicon
      if (data) {
        data.map((file: any) => (file.img = "/svg/" + prettyFileIcons.getIcon(file.name, "svg")));
        this.files = data;
      }
      error ? console.log(error) : null;
    },

    /**
     * Deletes a file from Supabase
     * @param {string} name, the filename to delete
     */
    async deleteFile(name: string): Promise<void> {
      const { data, error } = await supabase.storage.from("files").remove([name]);
      if (data) {
        data.length === 1 ? console.log(`file deleted`) : console.log(error);
      }
      await this.getFilesInStorage();
    },

    /**
     * Upload a file to Supabase
     * @returns {boolean}
     */
    async upload(): Promise<boolean> {
      const files = this.$refs.file as any;
      const file = files.files[0];
      const { data, error } = await supabase.storage.from("files").upload(file.name, file, {
        upsert: false
      });
      if (data) {
        data.Key ? console.log(`file uploaded`) : console.log(error);
        this.getFilesInStorage();
        return true;
      } else {
        return false;
      }
    },

    /**
     * Download a file from Supabase
     * @param {string} filename, the filename of the file to download
     */
    async download(filename: string): Promise<void> {
      //get the file from supabase
      const { data, error } = await supabase.storage.from("files").download(filename);

      //Save the file to the users computer
      if (data && this.saveFile(filename, data as Blob)) {
        console.log(`file downloaded`);
      } else {
        console.log(error);
      }
    },

    /**
     * Save a blob to users computer
     * @param {string} filename
     * @param {Blob} data, the file in blob format
     * @returns {boolean}
     */
    saveFile(filename: string, data: Blob): boolean {
      const e = document.createEvent("MouseEvents"),
        a = document.createElement("a");
      a.download = filename;
      a.href = window.URL.createObjectURL(data);
      a.dataset.downloadurl = ["text/json", a.download, a.href].join(":");
      e.initEvent("click", true, false);
      a.dispatchEvent(e);
      return true;
    },

    /**
     * Format a timestamp with momentJS
     * @param {string} time timestamp
     * @returns {string} readable time
     */
    formatTime(time: string): string {
      return moment(time).format("MMMM Do YYYY, h:mm:ss a");
    },

    /**
     * Format bytes into a readable format
     * @param {number} bytes input a byte value
     * @returns {string} a readable string
     */
    formatBytes(bytes: number): string {
      return prettyBytes(bytes);
    }
  }
});
</script>
