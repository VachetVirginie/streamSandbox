<template>
  <div>
    <v-container>
      <v-row class="mb-2">
        <v-col class="d-flex justify-start">
          <v-btn v-if="!isStreaming" color="success" large :disabled="isStreaming === undefined" @click="startStreaming">
            Start
          </v-btn>
          <v-btn v-else color="error" large @click="stopStreaming">
            Stop
          </v-btn>
           <v-btn color="error" large @click="getSceneSources">
            Scenes
          </v-btn>
          <v-btn class="ml-2" large :disabled="isStreaming !== false" @click="showSettingsForm = !showSettingsForm">
            Config
          </v-btn>
        </v-col>
        <v-col class="d-flex justify-end">
          <span v-if="isStreaming === undefined" class="mt-1">
            <v-icon color="grey">fas fa-times</v-icon>
            Non connecter
          </span>
          <span v-else-if="isStreaming === false" class="mt-1">
            <v-icon color="grey">fas fa-circle</v-icon>
            Live Off
          </span>
          <span v-else class="mt-1">
            <v-icon color="red">fas fa-circle</v-icon>
            Live On
          </span>
        </v-col>
      </v-row>
      <v-row>
        <v-col v-for="scene in availableScenes" :key="scene.name" md="4" sm="4" :class="[{'current-scene': currentSceneName === scene.name}]" class="text-center" @click="switchScene(scene.name)">
          <v-img src="https://static.vecteezy.com/system/resources/previews/000/536/526/non_2x/camera-recording-screen-vector-illustration-background.jpg">
            <span class="scene-name">
              {{ scene.name }}
            </span>
          </v-img>
        </v-col>
      </v-row>
    </v-container>
    <v-dialog v-model="showSettingsForm">
      <v-card width="500px">
        <v-card-title class="text-h5">
          Update
        </v-card-title>
        <v-form @submit.prevent="updateStreamSettings">
          <v-card-text>
            <v-select
              :options="availableStreamingServices"
              :value="availableStreamingServices.value"
              name="streamingService"
              label="Streaming Service"
            />
            <v-text-field
             v-model="streamingConfig.key"
             name="streamingKey"
            label="Streaming Key"
            placeholder="Streaming Key"
          ></v-text-field>
          </v-card-text>
          <v-card-actions>
            <v-btn color="alert" @click="showSettingsForm = false">
              Annuler
            </v-btn>
            <v-spacer />
            <v-btn color="success" type="submit">
              Valider
            </v-btn>
          </v-card-actions>
        </v-form>
      </v-card >
    </v-dialog>
  </div >
</template>

<script>
const OBSWebSocket = require('obs-websocket-js')
const obs = new OBSWebSocket()
export default {
  middleware: ['field'],
  data () {
    return {
      isStreaming: undefined,
      currentSceneName: undefined,
      streamingConfig: {
        key: null,
        service: undefined
      },
      showSettingsForm: false,
      availableScenes: [],
      availableStreamingServices: [
        { text: 'Facebook', value: 'Facebook Live' },
        { text: 'Twitch', value: 'Twitch' },
        { text: 'Youtube', value: navigator.appVersion.includes('Win') || navigator.appVersion.includes('Mac') ? 'YouTube - RTMPS' : 'Youtube / YouTube Gaming' }
      ],
      server: ''
    }
  },
  mounted () {
     const baseUrl = '192.168.1.11:8000' 
    obs.connect({
      address: baseUrl.replace(':8000', ':4444'),
      password: 'NGTV@ngtv'
    })
      .then(() => {
        obs.send('GetStreamingStatus').then((data) => { this.isStreaming = data.streaming })
        obs.send('GetStreamSettings').then((data) => { this.streamingConfig = data.settings })
        obs.send('GetSceneList').then((data) => { (this.availableScenes) = data.scenes })
        obs.send('GetCurrentScene').then((data) => { this.currentSceneName = data.name })
        // adress flux video cam 1
        // obs.send('GetSourceSettings', {'sourceName': 'cam1', 'sourceType':'ffmpeg_source'}).then((data) => { console.log(data.sourceSettings.input) })
      })
  },
  beforeUnmount () {
    obs.disconnect()
  },
  methods: {
    getSceneSources() {
      // console.log(this.availableScenes)
                this.availableScenes.forEach((scene) => {
            const sceneName = scene.name
            console.log(sceneName)
          })
    },
    switchScene (sceneName) {
      obs.send('SetCurrentScene', { 'scene-name': sceneName }).then(() => { this.currentSceneName = sceneName })
    },
    startStreaming () {
      obs.send('StartStreaming').then(() => { this.isStreaming = true })
    },
    stopStreaming () {
      obs.send('StopStreaming').then(() => { this.isStreaming = false })
    },
    updateStreamSettings () {
            this.server = {
              'Facebook Live': 'rtmps://live-api-s.facebook.com:443/rtmp/',
              Twitch: 'rtmp://mrs02.contribute.live-video.net/app/',
              'Youtube / YouTube Gaming': 'rtmp://a.rtmp.youtube.com/live2',
              'YouTube - RTMPS': 'rtmp://a.rtmp.youtube.com/live2'
            }
            obs.send('SetStreamSettings', {
              type: 'rtmp_common',
              settings: {
                key: this.streamingConfig.key,
                service: this.streamingConfig.service,
                server: this.server[this.streamingConfig.service]
              },
              save: true
            })
        
        .finally(() => {
          this.showSettingsForm = false
        })
    }
  }
}
</script>

<style lang="scss" scoped>
.current-scene {
  border: 2px solid red;
}
.scene-name {
  padding: 4px;
  background: rgba(0,0,0, 0.7);
  color:beige;
}
</style>