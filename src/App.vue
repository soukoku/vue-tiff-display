<template>
  <div class="flex-row">
    <div class="flex-fit container">
      Files:
      <ul>
        <li v-for="file in files"
            :key="file">
          <a href="#"
             @click="loadTiff(file)">
            {{file}}
          </a>
        </li>
      </ul>
    </div>
    <div class="flex-fill container">
      Pages:
      <ul>
        <li v-for="page in pages"
            :key="page.pageNumber">
          page {{page.pageNumber}} xres={{ page.xres }}, yres={{ page.yres }}, w={{ page.width
          }}, h={{ page.height }}
          <div>
            <tiff-page class="pgimage"
                       :tiff="tiff"
                       :page="page"></tiff-page>
          </div>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
import Tiff from 'tiff.js';
import TiffPage from '@/components/TiffPage';

// // Call C from JavaScript
// var result = Module.ccall('c_add', // name of C function
//         'number', // return type
//         ['number', 'number'], // argument types
//         [10, 20]); // arguments

function bitsToFloat(value) {
  var dv = new DataView(new ArrayBuffer(4));
  dv.setUint32(0, value);
  return dv.getFloat32();
}

export default {
  components: {
    TiffPage
  },
  props: {
    msg: String
  },
  data() {
    return {
      files: [
        '/INVERT Decl of Indep.tif',
        '/dfif-resolution.tif',
        '/grey.tif',
        '/color.tif',
        '/bw.tif'
      ].sort(),
      tiff: undefined,
      pages: []
    };
  },
  watch: {
    tiff(newVal, oldVal) {
      if (oldVal) oldVal.close();
    }
  },
  created() {
    // Tiff.initialize({ TOTAL_MEMORY: 1024 * 1024 * 100 });
  },
  beforeDestroy() {
    if (this.tiff) this.tiff.close();
  },
  methods: {
    loadTiff(url) {
      if (url) {
        this.pages = [];
        this.tiff = undefined;

        var xhr = new XMLHttpRequest();
        xhr.responseType = 'arraybuffer';
        xhr.open('GET', url);
        xhr.onload = () => {
          var tiff = new Tiff({ buffer: xhr.response });
          var pgCount = tiff.countDirectory();

          var pages = [];
          for (var i = 0; i < pgCount; ) {
            tiff.setDirectory(i);
            pages.push({
              pageNumber: ++i,
              width: tiff.width(),
              height: tiff.height(),
              xres: bitsToFloat(tiff.getField(Tiff.Tag.XRESOLUTION)),
              yres: bitsToFloat(tiff.getField(Tiff.Tag.YRESOLUTION))
            });
          }
          this.tiff = tiff;
          this.pages = pages;
        };
        xhr.send();
      }
    }
  }
};
</script>
<style>
body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen,
    Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
  font-size: 14px;
}
.flex-row {
  display: flex;
  flex-direction: row;
}
.flex-fit {
  flex: 0 0 auto;
}
.flex-fill {
  flex: 1 1 auto;
}
.pgimage {
  border: solid 1px red;
}
ul {
  list-style: none;
  margin: 0;
  padding: 0;
}
.container {
  padding: 10px;
}
</style>