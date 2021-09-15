<template>
  <div class="bg-gray-200 min-h-screen w-full h-full p-3 text-center">
    <h1 class="text-gray-900 text-3xl font-extrabold p-5">
      Audio Track Change Tool
    </h1>
    <p class="text-gray-700 truncate p-5">
      Presenting the same video content with multiple audio sources is now
      mainstream. To do the same to your video, feel free to upload the video
      and the relevant audios.
    </p>
    <div class="grid grid-cols-2 gap-5">
      <div class="m-5 px-4 py-8 bg-white shadow rounded">
        <form class="text-center" @submit.prevent="handleVideoUpload">
          <input
            type="file"
            name="file"
            @change="handleVideo"
            accept="video/*"
            required
            class="block mx-auto"
          />
          <button
            type="submit"
            :disabled="uploadingVideo"
            class="
              block
              inline-flex
              items-center
              px-4
              py-2
              border border-transparent
              text-sm
              font-medium
              rounded-md
              text-indigo-700
              bg-indigo-100
              hover:bg-indigo-200
              focus:outline-none
              focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500
              my-5
            "
          >
            Upload Video
          </button>
          <div v-if="uploadingVideo" class="text-sm">Uploading video...</div>
        </form>
      </div>
      <div v-if="video" class="m-5 px-4 py-8 bg-white shadow rounded">
        <form @submit.prevent="handleAudioUpload">
          <input
            type="file"
            name="file"
            @change="handleAudio"
            accept="audio/*"
            required
            class="block mx-auto"
          />
          <button
            type="submit"
            :disabled="uploadingAudio"
            class="
              block
              inline-flex
              items-center
              px-4
              py-2
              border border-transparent
              text-sm
              font-medium
              rounded-md
              text-indigo-700
              bg-indigo-100
              hover:bg-indigo-200
              focus:outline-none
              focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500
              my-5
            "
          >
            Upload Audio
          </button>
          <div v-if="uploadingAudio" class="text-sm">Uploading audio...</div>
        </form>
      </div>
    </div>
    <h2 v-if="video" class="text-gray-900 text-2xl font-semibold p-5">
      Output
    </h2>
    <div v-if="video" class="grid grid-cols-3 gap-5">
      <div class="m-5 px-4 py-8 bg-white shadow rounded">
        <cld-video
          :publicId="video.public_id"
          height="250"
          width="500"
          crop="fill"
          quality="auto"
          controls="true"
        />
        <p class="my-3 mx-0 text-left text-sm font-semibold text-gray-500">
          Video: {{ video.public_id }}
        </p>
        <p class="my-3 mx-0 text-left text-sm font-semibold text-gray-500">
          Audio: Initial
        </p>
      </div>
      <div
        v-for="(track, index) in audios"
        :key="index"
        class="m-5 px-4 py-8 bg-white shadow rounded"
      >
        <cld-video
          :publicId="video.public_id"
          height="250"
          width="500"
          crop="fill"
          quality="auto"
          controls="true"
        >
          <cld-transformation
            :overlay="`video:${track.public_id.replace('/', ':')}`"
          />
          <cld-transformation flags="layer_apply" />
        </cld-video>
        <p class="my-3 mx-0 text-left text-sm font-semibold text-gray-500">
          Video: {{ video.public_id }}
        </p>
        <p class="my-3 mx-0 text-left text-sm font-semibold text-gray-500">
          Audio: {{ track.public_id }}
        </p>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      uploadVideo: null,
      uploadAudio: null,
      uploadingVideo: false,
      uploadingAudio: false,
      video: null,
      audios: [],
    };
  },
  methods: {
    handleVideo(e) {
      this.uploadVideo = e.target.files[0];
    },
    handleAudio(e) {
      this.uploadAudio = e.target.files[0];
    },
    async handleVideoUpload(e) {
      this.uploadingVideo = true;

      const fileData = await this.readData(this.uploadVideo);

      this.video = await this.$cloudinary.upload(fileData, {
        upload_preset: "default-preset",
        folder: "nuxtjs-audio-track-change",
      });

      this.uploadingVideo = false;
    },
    async handleAudioUpload(e) {
      this.uploadingAudio = true;

      const fileData = await this.readData(this.uploadAudio);

      const audio = await this.$cloudinary.upload(fileData, {
        upload_preset: "default-preset",
        folder: "nuxtjs-audio-track-change",
      });

      this.audios.push(audio);

      this.uploadingAudio = false;
    },
    async readData(f) {
      return new Promise((resolve) => {
        const reader = new FileReader();
        reader.onloadend = () => resolve(reader.result);
        reader.readAsDataURL(f);
      });
    },
  },
};
</script>
