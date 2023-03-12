<script setup lang="ts">
import { Buffer } from 'buffer';
import { reactive, computed, watch, ref } from 'vue';

interface State {
  svgItem: (SvgItemRect | SvgItemText)[];
}

interface SvgItemRect {
  type: 'rect';
  x: number;
  y: number;
  width: number;
  height: number;
  rx: number;
  ry: number;
  fill: string;
}

interface SvgItemText {
  type: 'text';
  x: number;
  y: number;
  fontSize: number;
  fill: string;
  text: string;
}

const state = reactive<State>({
  svgItem: [
    {
      type: 'rect',
      x: 20,
      y: 20,
      width: 100,
      height: 100,
      rx: 0,
      ry: 0,
      fill: '#849ff0',
    },
    {
      type: 'text',
      x: 100,
      y: 20,
      fontSize: 20,
      fill: '#e74c3c',
      text: 'sample',
    },
  ],
});

/**
 * DataUrl形式の画像をImgに変換する
 */
const ToImage = (base64img: string): Promise<HTMLImageElement | null> => {
  return new Promise((resolve) => {
    const img = new Image();
    img.onload = () => resolve(img);
    img.onerror = (e) => resolve(null);
    img.crossOrigin = 'anonymous';
    img.src = base64img;
  });
};
/**
 * SVGタグをDataUrl形式のテキストに変換する
 */
const ToBase64 = (elm: HTMLElement | undefined | null): string | null => {
  try {
    if (elm === undefined) return null;
    if (elm === null) return null;
    const svgData = new XMLSerializer()
      .serializeToString(elm)
      .replace(/<!--[\s\S]*?-->/g, '');
    const dataUrl =
      'data:image/svg+xml;charset=utf-8;base64,' +
      // btoa(unescape(encodeURIComponent(svgData)))
      Buffer.from(svgData).toString('base64');
    return dataUrl;
  } catch (error) {
    console.error(error);
    return null;
  }
};
const svgElement = ref();
const pngDataUrl = ref<null | string>(null);
/**
 * SVGからPng画像を生成する
 */
const createPngImg = async () => {
  let canvas: HTMLCanvasElement | null = null;
  let ctx: CanvasRenderingContext2D | null = null;
  try {
    pngDataUrl.value = null;
    const svgDataUrl = ToBase64(svgElement.value);
    // console.log(svgDataUrl);
    if (svgDataUrl === null) return null;
    const img = await ToImage(svgDataUrl);
    // console.log(img);
    canvas = document.createElement('canvas');
    canvas.width = 400;
    canvas.height = 400;
    ctx = canvas.getContext('2d');
    if (img === null || ctx === null) return null;
    ctx.drawImage(img, 0, 0);
    return canvas.toDataURL('image/png');
  } catch (error) {
    console.error(error);
    return null;
  } finally {
    canvas = null;
    ctx = null;
  }
};

const createPngImgBtn = async () => {
  pngDataUrl.value = await createPngImg();
};

const test = (row: SvgItemRect) => {
  console.log('Rectタグがクリックされました', row);
};
</script>
<template>
  <div class="container-fluid">
    <div class="card mt-3">
      <div class="card-header text-white bg-primary">VueでSVGをあつかう</div>
      <div class="card-body">
        <div class="row">
          <div class="col-4">
            <div class="">SVG描画部分</div>
            <div class="svg-container">
              <svg
                width="400"
                height="400"
                viewBox="0, 0, 400, 400"
                xmlns="http://www.w3.org/2000/svg"
                ref="svgElement"
              >
                <template v-for="(row, index) in state.svgItem" :key="index">
                  <template v-if="row.type === 'rect'">
                    <rect
                      :x="row.x"
                      :y="row.y"
                      :width="row.width"
                      :height="row.height"
                      :rx="row.rx"
                      :ry="row.ry"
                      :fill="row.fill"
                      @click="test(row)"
                    />
                  </template>
                  <template v-if="row.type === 'text'">
                    <text
                      :x="row.x"
                      :y="row.y"
                      :font-size="row.fontSize"
                      :fill="row.fill"
                      font-family="serif"
                    >
                      {{ row.text }}
                    </text>
                  </template>
                </template>
              </svg>
            </div>
          </div>
          <div class="col-4">
            <div class="">SVGの要素</div>

            <button
              type="button"
              class="btn btn-primary mb-2"
              @click="createPngImgBtn"
            >
              PNG画像▶
            </button>

            <div class="controler">
              <div class="">SVGの要素</div>
              <template v-for="(row, index) in state.svgItem" :key="index">
                <div class="item">
                  <div class="title">Rectangle {{ index + 1 }}</div>
                </div>
                <template v-if="row.type === 'rect'">
                  <label class="form-label">x :{{ row.x }}</label>
                  <input
                    type="range"
                    class="form-range"
                    min="0"
                    max="400"
                    v-model="row.x"
                  />
                  <label class="form-label">y :{{ row.y }}</label>
                  <input
                    type="range"
                    class="form-range"
                    min="0"
                    max="400"
                    v-model="row.y"
                  />
                  <label class="form-label">width :{{ row.width }}</label>
                  <input
                    type="range"
                    class="form-range"
                    min="0"
                    max="400"
                    v-model="row.width"
                  />
                  <label class="form-label">height :{{ row.height }}</label>
                  <input
                    type="range"
                    class="form-range"
                    min="0"
                    max="400"
                    v-model="row.height"
                  />
                  <label class="form-label">rx :{{ row.rx }}</label>
                  <input
                    type="range"
                    class="form-range"
                    min="0"
                    max="50"
                    v-model="row.rx"
                  />
                  <label class="form-label">ry :{{ row.ry }}</label>
                  <input
                    type="range"
                    class="form-range"
                    min="0"
                    max="50"
                    v-model="row.ry"
                  />
                  <label class="form-label">fill :{{ row.fill }}</label>
                  <input
                    type="color"
                    class="form-control form-control-color"
                    id="exampleColorInput"
                    v-model="row.fill"
                    title="Choose your color"
                  />
                </template>

                <template v-if="row.type === 'text'">
                  <label class="form-label">x :{{ row.x }}</label>
                  <input
                    type="range"
                    class="form-range"
                    min="0"
                    max="400"
                    v-model="row.x"
                  />
                  <label class="form-label">y :{{ row.y }}</label>
                  <input
                    type="range"
                    class="form-range"
                    min="0"
                    max="400"
                    v-model="row.y"
                  />
                  <label class="form-label"
                    >Font Size :{{ row.fontSize }}</label
                  >
                  <input
                    type="range"
                    class="form-range"
                    min="0"
                    max="400"
                    v-model="row.fontSize"
                  />
                  <label class="form-label">text </label>
                  <input type="text" class="form-control" v-model="row.text" />
                </template>
              </template>
            </div>
          </div>
          <div class="col-4">
            <div class="">Png画像</div>
            <div class="createdImg">
              <img :src="pngDataUrl" alt="" v-if="pngDataUrl !== null" />
              <div v-else>ここに画像が表示されます。</div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
  <teleport to="#teleport"></teleport>
</template>

<style lang="scss" scoped>
img,
svg {
  border: solid 1px gray;
  width: 100%;
  height: auto;
  object-fit: contain;
}

.item {
  border-top: 1px solid gray;
  padding: 10px 0;
  label {
    margin-bottom: 0;
  }
}
.item:not(:last-child) {
  margin-top: 20px;
}

.controler {
  max-height: 400px;
  overflow: auto;
}
</style>
