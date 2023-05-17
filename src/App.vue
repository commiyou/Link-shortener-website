<script lang="ts">
import { defineComponent } from 'vue'
import ClipboardJS from 'clipboard'
import { useToast } from 'vue-toast-notification';
import 'vue-toast-notification/dist/theme-sugar.css';



export default defineComponent({
  data() {
    return {
      url: "",
      shortUrl: "",
    }
  },
  mounted() {
    const clipboard = new ClipboardJS("#copy-btn")
    const $toast = useToast();
    clipboard.on("success", e => {
      let instance = $toast.success('Copied successfully!', { position: "top"});
      e.clearSelection();
      setTimeout(() => instance.dismiss(), 2000);
    })

    clipboard.on('error', function (e) {
      console.error('Action:', e.action);
      console.error('Trigger:', e.trigger);
      let instance = $toast.error('Copy failed! use Ctrl+C to copy!', { position: "top"});
      setTimeout(() => instance.dismiss(), 2000);
    });

  },
  methods: {
    shortenUrl() {
      this.shortUrl = "Fetching ...";

      console.log(`shorten ${this.url}`)
      fetch('https://api-ssl.bitly.com/v4/shorten', {
        method: 'POST',
        headers: {
          'Authorization': 'Bearer 6d2f4e39742d96331b1d8a31fdc672619c54f4b4',
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({ "long_url": this.url, })
      }).then(
        async (response) => {
          if (!response.ok) {
            const msg = await response.json();
            console.log(msg);
            throw new Error(msg.description);
          }
          return response.json()
        }
      ).then(
        data => { this.shortUrl = data.link; }
      ).catch(error => { this.shortUrl = error });
    }
  },
})
</script>

<template>
  <br/>
  <br/>
  <br/>
  <a-input-group compact>
  <a-input type="url" style="width: calc(100% - 200px)" placeholder="Input url to shorten" v-model:value="url" @focus="$event.target?.select()" /> 
  <a-button @click="shortenUrl" type="primary" >Shorten</a-button>
</a-input-group>
<a-input-group v-if="shortUrl" compact>
  <a-span bordered=true style="width: calc(100% - 200px)"  id="short-url"> {{ shortUrl }} </a-span>
  <a-button type="primary" id="copy-btn" data-clipboard-target="#short-url">Copy</a-button>
</a-input-group>
</template>

<style scoped></style>
