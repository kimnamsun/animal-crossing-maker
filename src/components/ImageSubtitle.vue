<template>
  <div id="app">
    <el-container>
      <el-header>
        <h1>만들어봐요 동숲 짤</h1>
      </el-header>
      <el-main>
        <!-- <div class="select-container">
          <div class="select">
            <div
              class="select-option"
              v-for="image in images"
              :key="image.id"
              @click="onImageChanged(image.id)"
            >
              <el-image
                v-if="selectedImage === image.id"
                style="width: 35px; height: 35px; margin-left: 10px"
                :src="require('@/assets/cursor.png')"
                :fit="fit"
              />
              <span>
                {{ image.label }}
              </span>
            </div>
          </div>
        </div> -->
        <el-button
          @click="downloadCanvas"
          id="download"
          style="
            color: white;
            background-color: #1bd9b4;
            border: none;
            border-radius: 10px;
          "
          >다운로드를 해보자구리</el-button
        >

        <el-card class="box-card">
          <canvas id="canvas" ref="canvas" :width="1000" :height="400">
            이 브라우저는 HTML5의 canvas 요소를 지원하지 않습니다
          </canvas>
        </el-card>

        <div class="input-container">
          <el-image
            style="width: 35px; height: 35px; margin-left: 10px"
            :src="require('@/assets/leaf.png')"
            :fit="fit"
          />
          <el-input
            type="text"
            :value="option.text"
            @input="onValueChanged('text', $event)"
            class="transparent-input"
          />
        </div>
      </el-main>
    </el-container>
  </div>
</template>

<script>
export default {
  name: 'app',

  data() {
    return {
      windowWidth: 0,
      imageIndex: 0,
      // selectedImage: 1,
      option: {
        fontFamily: 'Nanum Gothic',
        fontSize: 30,
        fontColor: '#FFFFFF',
        fontWeight: 'normal',
        text: '',
        textBorder: 'black',
      },
    };
  },

  computed: {
    image() {
      return { id: 2, src: require('@/assets/image2.png'), label: '말풍선' };
    },
    // images() {
    //   return [
    //     { id: 1, src: require('@/assets/image1.png'), label: '로고' },
    //     { id: 2, src: require('@/assets/image2.png'), label: '말풍선' },
    //   ];
    // },
  },

  watch: {
    selectedImage() {
      this.updateCanvasImage();
    },
  },

  methods: {
    onValueChanged(key, value) {
      this.option[key] = value;
      this.updateCanvas();
    },

    // onImageChanged(value) {
    //   this.selectedImage = value;
    //   this.updateCanvas();
    // },

    updateCanvas() {
      if (!this.$refs.canvas) {
        return;
      }
      this.updateCanvasImage();
    },

    updateCanvasImage() {
      const { canvas } = this.$refs;
      const ctx = canvas.getContext('2d');
      ctx.clearRect(0, 0, canvas.width, canvas.height); // 기존 내용 삭제

      const img = new Image();
      img.src = this.image.src;

      img.onload = () => {
        ctx.drawImage(img, 0, 0, canvas.width, canvas.height);
      };
    },
    updateCanvasText() {
      const { canvas } = this.$refs;

      const { text, fontFamily, fontSize, fontColor, fontWeight, textBorder } =
        this.option;

      const ctx = canvas.getContext('2d');

      ctx.textAlign = 'center'; // 가로 가운데 정렬
      ctx.textBaseline = 'middle'; // 세로 가운데 정렬
      ctx.font = `${fontWeight} ${fontSize}px ${fontFamily}`;

      const lines = text.split('\n');

      lines.forEach((line, index) => {
        ctx.shadowColor = 'black';
        ctx.shadowBlur = 10;
        ctx.lineWidth = 5;
        ctx.strokeStyle = `${textBorder}`;
        ctx.strokeText(
          line,
          canvas.width / 2,
          canvas.height - fontSize * (lines.length - index) * 1.5
        );

        ctx.fillStyle = fontColor;
        ctx.fillText(
          line,
          canvas.width / 2,
          canvas.height - fontSize * (lines.length - index) * 1.5
        );
      });
    },

    downloadCanvas() {
      const url = this.$refs.canvas.toDataURL('image/png');
      const link = document.createElement('a');
      link.href = url;
      link.setAttribute('download', `${this.images[this.imageIndex].name}.png`);
      document.body.appendChild(link);
      link.click();
      document.body.removeChild(link);
    },

    calculateWindowWidth() {
      this.windowWidth = window.innerWidth;
    },
  },

  mounted() {
    this.calculateWindowWidth();
    this.$nextTick(() => {
      window.addEventListener('resize', this.calculateWindowWidth);

      this.updateCanvas();
    });
  },
};
</script>

<style scoped>
.box-card {
  border-radius: 10px;
  margin: 20px 0;
}

.input-container {
  display: flex;
  align-items: center;
  margin-bottom: 20px;
  border-radius: 10px;
  background-color: white;
  width: 100%;
  height: 50px;
}

.el-item {
  margin: 0;
}

/* .select-container {
  display: flex;
  justify-content: center;
  align-content: center;
  margin-bottom: 20px;
}

.select {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  width: 300px;
  height: 130px;
  border-radius: 45%;
  background-color: #f6e8a7;
}

.select-option {
  display: flex;
  justify-content: center;
  align-items: center;
  cursor: pointer;
} */
</style>
