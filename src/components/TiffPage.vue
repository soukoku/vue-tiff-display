<script>
import Tiff from 'tiff.js';

export default {
  props: {
    tiff: {
      type: Object,
      required: true
    },
    page: {
      type: Object,
      required: true
    }
  },
  computed: {
    canvasAttrs() {
      var pg = this.page;
      var dpiRatio = window.devicePixelRatio || 1;

      var resXRatio = pg.xres / 96;
      var resYRatio = pg.yres / 96;

      var dispW = Math.ceil(pg.width / (resXRatio * dpiRatio));
      var dispH = Math.ceil(pg.height / (resYRatio * dpiRatio));

      var attrs = {
        width: pg.width,
        height: pg.height,
        style: `width: ${dispW}px; height: ${dispH}px`
      };
      return attrs;
    }
  },
  watch: {
    canvasAttrs() {
      this.drawPage();
    },
    tiff() {
      this.drawPage();
    },
    page() {
      this.drawPage();
    }
  },
  mounted() {
    this.drawPage();
  },
  methods: {
    drawPage() {
      if (!this.tiff) return;

      this.tiff.setDirectory(this.page.pageNumber - 1);
      // var scale = window.devicePixelRatio;
      // var hScale = 1;
      // if (this.page.xres != this.page.yres) {
      //   hScale = this.page.xres / this.page.yres;
      // }

      var { width, height } = this.page;
      // var { dispWidth, dispHeight } = this.canvasAttrs;

      // width /= hScale;
      // height /= hScale;
      var raster = Tiff.Module.ccall(
        '_TIFFmalloc',
        'number',
        ['number'],
        [width * height * 4]
      );
      var result = Tiff.Module.ccall(
        'TIFFReadRGBAImageOriented',
        'number',
        ['number', 'number', 'number', 'number', 'number', 'number'],
        [this.tiff._tiffPtr, width, height, raster, 1, 0]
      );

      if (result === 0) {
        Tiff.Module.ccall('free', 'number', ['number'], [raster]);
        throw new Tiff.Exception(
          'The function TIFFReadRGBAImageOriented returns NULL'
        );
      }
      var image = Tiff.Module.HEAPU8.subarray(
        raster,
        raster + width * height * 4
      );

      var canvas = this.$el;
      var context = canvas.getContext('2d');
      var imageData = context.createImageData(width, height);
      imageData.data.set(image);
      context.putImageData(imageData, 0, 0);
      Tiff.Module.ccall('free', 'number', ['number'], [raster]);
    }
  },
  render(h) {
    return h('canvas', { attrs: this.canvasAttrs });
  }
};
</script>
