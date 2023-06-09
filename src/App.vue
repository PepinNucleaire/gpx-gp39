<script setup>
import gpx from "./gpx-parser-builder";
</script>

<template>
  <main>
    <h1>
      OpenCPN GPX to GP39 gpx converter
      <a href="https://github.com/PepinNucleaire/gpx-gp39" aria-label="Repo source" class="footer-octicon" title="GitHub">
        <svg aria-hidden="true" class="octicon octicon-mark-github" height="24" version="1.1" viewBox="0 0 16 16"
          width="24">
          <path fill="white" fill-rule="evenodd"
            d="M8 0C3.58 0 0 3.58 0 8c0 3.54 2.29 6.53 5.47 7.59.4.07.55-.17.55-.38 0-.19-.01-.82-.01-1.49-2.01.37-2.53-.49-2.69-.94-.09-.23-.48-.94-.82-1.13-.28-.15-.68-.52-.01-.53.63-.01 1.08.58 1.23.82.72 1.21 1.87.87 2.33.66.07-.52.28-.87.51-1.07-1.78-.2-3.64-.89-3.64-3.95 0-.87.31-1.59.82-2.15-.08-.2-.36-1.02.08-2.12 0 0 .67-.21 2.2.82.64-.18 1.32-.27 2-.27.68 0 1.36.09 2 .27 1.53-1.04 2.2-.82 2.2-.82.44 1.1.16 1.92.08 2.12.51.56.82 1.27.82 2.15 0 3.07-1.87 3.75-3.65 3.95.29.25.54.73.54 1.48 0 1.07-.01 1.93-.01 2.2 0 .21.15.46.55.38A8.013 8.013 0 0 0 16 8c0-4.42-3.58-8-8-8z">
          </path>
        </svg>
      </a>
    </h1>

    <p>Upload your GPX file from OpenCPN</p>
    <p>
      It will automatically transform to a gpx file that can be uploaded to gp39
    </p>
    <input type="file" ref="file" @change="readFile" />
    <button v-if="loading" @click="submitFile">
      <div class="lds-ring">
        <div></div>
        <div></div>
        <div></div>
        <div></div>
      </div>
    </button>

    <button v-if="!loading" @click="submitFile">
      Download file for GP39
    </button>
  </main>
</template>

<script>
export default {
  data() {
    return {
      file: "",
      loading: false,
      gpx: {
      }
    }
  },
  methods: {
    readFile(e) {
      console.log(e)
      this.file = e.target.files[0];
    },
    checkColor(sym) {
      if (sym == "Symbol-Pin-Black") {
        return { color: 0, sym: 1 };
      }
      if (sym == "Symbol-Exclamation-Magenta") {
        return { color: 5, sym: 2 };
      }
      if (sym == "Sea-DepthB-Yellowk") {
        return { color: 2, sym: 2 };
      }
      if (sym == "triangle") {
        return { color: 0, sym: 1 };
      }
      if (sym == "square") {
        return { color: 1, sym: 2 };
      }
      if (sym == "circle") {
        return { color: 0, sym: 4 };
      }
      if (sym == "xmgreen") {
        return { color: 3, sym: 0 };
      }
      if (sym == "xmyellow") {
        return { color: 2, sym: 0 };
      }
      if (sym == "xmred") {
        return { color: 1, sym: 0 };
      }
      if (sym == "xmblue") {
        return { color: 6, sym: 0 };
      }
      return { color: 0, sym: 0 };
    },
    submitFile() {
      const filename = this.file.name;
      let reader = new FileReader();
      reader.onload = (e) => {
        let text = e.target.result;
        const gpxParsed = gpx.parse(text);
        console.log(gpxParsed.wpt[0]);
        let newwpts = [];
        let i = 0
        gpxParsed.wpt.map((wpt) => {

          if (!wpt.name) {
            wpt.name = `z${('0000000' + i).slice(-7)}`
            i += 1
          }

          const newwpt = {
            $: {
              lon: wpt["$"].lon,
              lat: wpt["$"].lat,
            },
            name: wpt.name.toUpperCase().slice(0, 8),
            extensions: {
              GP39Symbol: this.checkColor(wpt.sym).sym,
              FECColor: this.checkColor(wpt.sym).color,
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
        console.log(this.gpx.toString())
        this.saveFile(this.gpx.toString(), filename);
      };
      reader.readAsText(this.file);
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

.lds-ring {
  display: inline-block;
  position: relative;
  width: 80px;
  height: 80px;
}

.lds-ring div {
  box-sizing: border-box;
  display: block;
  position: absolute;
  width: 64px;
  height: 64px;
  margin: 8px;
  border: 8px solid #fff;
  border-radius: 50%;
  animation: lds-ring 1.2s cubic-bezier(0.5, 0, 0.5, 1) infinite;
  border-color: #fff transparent transparent transparent;
}

.lds-ring div:nth-child(1) {
  animation-delay: -0.45s;
}

.lds-ring div:nth-child(2) {
  animation-delay: -0.3s;
}

.lds-ring div:nth-child(3) {
  animation-delay: -0.15s;
}

@keyframes lds-ring {
  0% {
    transform: rotate(0deg);
  }

  100% {
    transform: rotate(360deg);
  }
}
</style>
