<template>
  <div id="app">
    <el-container>
      <el-header>
        <el-image
          style="width: 300px"
          :src="require('@/assets/logo.png')"
          v-if="getPathName === '/'"
        />
      </el-header>
      <el-main>
        <el-button
          @click="downloadCanvas"
          id="download"
          style="
            color: #683617;
            background-color: #f6e8a7;
            border: none;
            border-radius: 10px;
          "
        >
          <div
            style="
              display: flex;
              justify-content: center;
              align-content: center;
            "
          >
            <el-image
              style="width: 35px; height: 35px; margin-top: 3px"
              :src="require('@/assets/cursor.png')"
              :fit="fit"
            />
            <p class="download-text">다운로드를 해보자구리</p>
          </div>
        </el-button>

        <el-card class="box-card">
          <canvas id="canvas" ref="canvas" :width="1000" :height="400">
            이 브라우저는 HTML5의 canvas 요소를 지원하지 않습니다
          </canvas>
        </el-card>

        <div class="inputs-container">
          <div
            v-for="(input, index) in inputs"
            :key="index"
            class="input-container"
          >
            <el-image
              style="width: 35px; height: 35px; margin-left: 10px"
              :src="require('@/assets/leaf.png')"
            />
            <span class="input-label">{{ input.label }}</span>
            <el-input
              :type="input.type === 'contents' ? 'textarea' : 'text'"
              :value="input.text"
              :maxLength="input.maxLength"
              @input="onValueChanged(input.type, $event)"
            />
          </div>
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
      commonStyle: {
        fontFamily: 'NanumGothic',
        fontSize: 30,
        fontWeight: 'bold',
        textBorder: 'none',
      },
      inputs: [
        {
          id: 1,
          type: 'name',
          label: '이름',
          text: '',
          style: {
            ...this.commonStyle,
            fontColor: '#683617',
          },
          maxLength: 3,
        },
        {
          id: 2,
          type: 'contents',
          label: '내용',
          text: '',
          style: {
            ...this.commonStyle,
            fontColor: '#827255',
          },
          maxLength: 30,
        },
      ],
    };
  },

  computed: {
    getImage() {
      return require('@/assets/image.png');
    },

    getPathName() {
      return window.location.pathname;
    },
  },

  methods: {
    onValueChanged(type, value) {
      const input = this.inputs.find((input) => input.type === type);

      if (!input) {
        return;
      }

      // 줄 바꿈 문자('\n')의 개수를 세기 위한 정규 표현식 사용
      const lineBreakCount = (value.match(/\n/g) || []).length;

      // 만약 줄 바꿈 문자가 4개 이상이면, 더 이상 입력을 허용하지 않음
      if (input.type === 'contents' && lineBreakCount >= 4) {
        return;
      }

      // maxLength를 초과하는 경우, maxLength까지만 입력값을 자름
      if (value.length > input.maxLength) {
        value = value.slice(0, input.maxLength);
      }

      input.text = value;

      this.drawImage();
    },

    drawImage() {
      const { canvas } = this.$refs;
      const ctx = canvas.getContext('2d');

      if (!this.img) {
        // 이미지가 없으면 로드
        this.img = new Image();
        this.img.src = this.getImage;

        this.img.onload = () => {
          // 이미지가 로드되면 그림
          ctx.drawImage(this.img, 0, 0, canvas.width, canvas.height);
          this.drawCanvasText();
        };
        return;
      }

      ctx.drawImage(this.img, 0, 0, canvas.width, canvas.height);
      this.drawCanvasText();
    },

    drawCanvasText() {
      this.inputs.forEach(({ type, text }) => {
        if (!text) {
          return;
        }

        this.updateCanvasText(type, text);
      });
    },

    updateCanvasText(type, text) {
      const { canvas } = this.$refs;
      const ctx = canvas.getContext('2d');

      ctx.font = `${this.commonStyle.fontWeight} ${this.commonStyle.fontSize}px ${this.commonStyle.fontFamily}`;

      ctx.textAlign = 'center';
      ctx.textBaseline = 'middle';
      ctx.textBorder = this.commonStyle.textBorder;

      if (type === 'name') {
        this.updateCanvasNameText(text);
        return;
      }

      if (type === 'contents') {
        this.updateCanvasContentsText(text);
        return;
      }
    },

    updateCanvasNameText(text) {
      const style = this.inputs.find((input) => input.type === 'name').style;

      const { canvas } = this.$refs;
      const ctx = canvas.getContext('2d');

      const lines = text.split('\n');
      const lineHeight = this.commonStyle.fontSize * 1.5; // 행 간격
      const x = 165; // x 좌표값 조정
      const y = 75; // y 좌표값 조정

      const angle = -Math.PI / 20; // 회전 각도 (라디안)

      ctx.save(); // 현재 컨텍스트 설정 저장
      ctx.translate(x, y); // 회전 중심 좌표 설정
      ctx.rotate(angle); // 지정한 각도만큼 회전
      ctx.fillStyle = style.fontColor;

      lines.forEach((line, index) => {
        const yCoord = index * lineHeight;
        ctx.fillText(line, 0, yCoord); // 회전한 각도에 따라 텍스트 그리기
      });

      ctx.restore(); // 이전 컨텍스트 설정 복구
    },

    updateCanvasContentsText(text) {
      const style = this.inputs.find(
        (input) => input.type === 'contents'
      ).style;

      const { canvas } = this.$refs;
      const ctx = canvas.getContext('2d');

      const lines = text.split('\n');
      const lineHeight = this.commonStyle.fontSize * 1.5; // 행 간격
      const totalTextHeight = lines.length * lineHeight; // 텍스트 전체 높이
      const yStartPosition =
        (canvas.height - totalTextHeight) / 2 + this.commonStyle.fontSize / 2; // 이미지 위에 텍스트를 그리기 위해 조정

      lines.forEach((line, index) => {
        const y = yStartPosition + index * lineHeight;

        ctx.strokeStyle = style.fontColor;
        ctx.strokeText(line, canvas.width / 2, y);
        ctx.fillStyle = style.fontColor;
        ctx.fillText(line, canvas.width / 2, y);
      });
    },

    downloadCanvas() {
      const url = this.$refs.canvas.toDataURL('image/png');
      const link = document.createElement('a');
      link.href = url;
      link.setAttribute('download', `image.png`);
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
    this.drawImage();
    this.$nextTick(() => {
      window.addEventListener('resize', this.calculateWindowWidth);
    });
  },
};
</script>

<style scoped>
.box-card {
  border-radius: 10px;
  margin: 20px 0;
}

.el-main {
  padding: 150px;
}

.inputs-container {
  border-radius: 10px;
  background-color: white;
  padding: 10px 20px 20px 20px;
}

.input-container {
  display: flex;
  align-items: center;
  padding-top: 10px;
}

.input-label {
  width: 5%;
}

.download-text:hover {
  background-color: #f3d700;
  transition: 0.3s;
}
</style>
