<template>
  <div class="hello">
    <div class="header">
      <button :disabled="pdfScale >= scaleMax" @click="larger()">larger</button>
      <button :disabled="pdfScale <= scaleMin" @click="smaller()">
        smaller
      </button>
    </div>
    <div class="content">
      <div v-for="page in pdfPages" :key="page" style="position: relative">
        <canvas :id="`pdf-${page}`" />
        <div :id="`text-${page}`" class="textLayer"></div>
      </div>
    </div>
  </div>
</template>

<script>
import { TextLayerBuilder } from "pdfjs-dist/web/pdf_viewer";
import "pdfjs-dist/web/pdf_viewer.css";
import { EventBus } from "pdfjs-dist/web/pdf_viewer";
const PDFJS = require("pdfjs-dist");
PDFJS.GlobalWorkerOptions.workerSrc =
  "https://cdn.bootcss.com/pdf.js/2.6.347/pdf.worker.js";
export default {
  name: "HelloWorld",
  data() {
    return {
      pdfDoc: null,
      pdfScale: 1.0,
      pdfPages: [],
      pdfWidth: "",
      pdfSrc: null,
      scaleMax: window.screen.width > 1440 ? 1.4 : 1.2,
      scaleMin: 1,
    };
  },
  mounted() {
    this.loadFile();
  },
  methods: {
    loadFile() {
      let loadTask = PDFJS.getDocument(
        "HPE MicroServer Gen10 Plus 服务器-产品彩页.pdf"
      );
      loadTask.promise
        .then((pdf) => {
          this.pdfDoc = pdf;
          this.pdfPages = pdf.numPages;
        })
        .then(async () => {
          this.renderPage(1);
        });
    },
    async renderPage(num) {
      let page = await this.pdfDoc.getPage(num);
      let canvas = document.getElementById(`pdf-${num}`);
      let ctx = canvas.getContext("2d");
      let dpr = window.devicePixelRatio || 1;
      let bsr =
        ctx.webkitBackingStorePixelRatio ||
        ctx.mozBackingStorePixelRatio ||
        ctx.msBackingStorePixelRatio ||
        ctx.oBackingStorePixelRatio ||
        ctx.backingStorePixelRatio ||
        1;
      let ratio = dpr / bsr;
      let viewport = page.getViewport({ scale: this.pdfScale });
      canvas.width = viewport.width * ratio;
      canvas.height = viewport.height * ratio;

      canvas.style.width = viewport.width + "px";
      canvas.style.height = viewport.height + "px";
      this.pdfWidth = viewport.width + "px";
      ctx.setTransform(ratio, 0, 0, ratio, 0, 0);
      let renderContext = {
        canvasContext: ctx,
        viewport: viewport,
      };
      await page.render(renderContext);

      const textLayerDiv = document.getElementById(`text-${num}`);
      textLayerDiv.style.width = viewport.width + "px";
      textLayerDiv.style.height = viewport.height + "px";
      var textLayer = new TextLayerBuilder({
        textLayerDiv: textLayerDiv,
        eventBus: new EventBus(),
        pageIndex: page.pageIndex,
        viewport: viewport,
      });

      let textContent = await page.getTextContent();
      textLayer.setTextContent(textContent);

      textLayer.render();

      if (this.pdfPages > num) {
        this.renderPage(num + 1);
      }
    },
    larger() {
      if (this.pdfScale >= this.scaleMax) {
        return;
      }
      this.pdfScale = this.pdfScale + 0.1;
      this.loadFile();
    },
    smaller() {
      if (this.pdfScale <= this.scaleMin) {
        return;
      }
      this.pdfScale = this.pdfScale - 0.1;
      this.loadFile();
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.hello {
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
}
.header {
  width: 100%;
  flex: 0 0 48px;
  background: #eaeaea;
  display: flex;
  justify-content: center;
  align-items: center;
}
.content {
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  overflow: auto;
}
</style>
