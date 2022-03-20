<script setup>
import gpx from "./gpx-parser-builder";
</script>

<template>
  <main>
    <h1>OpenCPN GPX to GP39 gpx converter</h1>
    <p>Upload your GPX file from OpenCPN</p>
    <p>
      It will automatically transform to a gpx file that can be uploaded to gp39
    </p>
    <input type="file" ref="file" @change="readFile()" />
    <button @click="submitFile">Upload!</button>
  </main>
</template>

<script>
export default {
  data() {
    file: "";
    gpx: {
    }
  },
  methods: {
    readFile() {
      this.file = this.$refs.file.files[0];
    },
    submitFile() {
      const filename = this.file.name;
      let reader = new FileReader();
      reader.onload = (e) => {
        let text = e.target.result;
        const gpxParsed = gpx.parse(text);
        console.log(gpxParsed.wpt[0]);
        let newwpts = [];
        gpxParsed.wpt.map((wpt) => {
          const newwpt = {
            $: {
              lon: wpt["$"].lon,
              lat: wpt["$"].lat,
            },
            name: wpt.name.toUpperCase(),
            extensions: {
              GP39Symbol: 0,
              FECColor: 1,
              GP39Comment: "1",
              GP39Flag: 1,
            },
          };
          newwpts.push(newwpt);
        });
        const newGpx = new gpx({
          $: {
            xmlns: "http://www.topografix.com/GPX/1/1",
            "xmlns:xsi": "http://www.w3.org/2001/XMLSchema-instance",
            "xmlns:xsd": "http://www.w3.org/2001/XMLSchema",
          },
          metadata: gpxParsed.metadata,
          wpt: newwpts,
        });
        console.log(newGpx);
        this.gpx = newGpx;
      };
      reader.readAsText(this.file);
      this.saveFile(this.gpx.toString(), filename);
    },

    saveFile: function (saving, name) {
      const data = saving;
      const blob = new Blob([data], { type: "text/plain" });
      const e = document.createEvent("MouseEvents"),
        a = document.createElement("a");
      a.download = "GP39_WptRte.gpx";
      a.href = window.URL.createObjectURL(blob);
      a.dataset.downloadurl = ["text/gpx", a.download, a.href].join(":");
      e.initEvent(
        "click",
        true,
        false,
        window,
        0,
        0,
        0,
        0,
        0,
        false,
        false,
        false,
        false,
        0,
        null
      );
      a.dispatchEvent(e);
    },
  },
};
</script>
<style>
@import url("simpledotcss/simple.min.css");

#app {
  max-width: 1280px;
  margin: 0 auto;
  padding: 2rem;

  font-weight: normal;
}
</style>
