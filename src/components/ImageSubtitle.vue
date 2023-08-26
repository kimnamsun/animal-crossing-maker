<template>
  <div id="app">
    <el-container>
      <el-header>
        <h1 v-if="getPathName === '/'">만들어봐요 동숲 짤</h1>
      </el-header>
      <el-main>
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

        <div v-for="(input, index) in inputs" :key="index">
          <div class="input-container">
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
      inputs: [
        {
          id: 1,
          type: 'name',
          label: '이름',
          text: '',
          style: {
            fontFamily: 'NanumGothic',
            fontSize: 30,
            fontWeight: 'bold',
            fontColor: '#683617',
            textBorder: 'none',
          },
          maxLength: 3,
        },
        {
          id: 2,
          type: 'contents',
          label: '내용',
          text: '',
          style: {
            fontFamily: 'NanumGothic',
            fontSize: 30,
            fontWeight: 'bold',
            fontColor: '#827255',
            textBorder: 'none',
          },
          maxLength: 30,
        },
      ],
    };
  },

  computed: {
    image() {
      return { id: 2, src: require('@/assets/image2.png'), label: '말풍선' };
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
        // 현재 input.text를 유지하거나 초기화하려면 여기에서 처리
        // input.text = ...;
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

      const img = new Image();
      img.src = require('@/assets/image2.png');

      img.onload = () => {
        ctx.drawImage(img, 0, 0, canvas.width, canvas.height);

        this.inputs.forEach(({ type, text }) => {
          if (!text) {
            return;
          }

          if (type === 'name') {
            this.updateCanvasNameText(text);
            return;
          }

          if (type === 'contents') {
            this.updateCanvasContentsText(text);
            return;
          }
        });
      };
    },

    updateCanvasNameText(text) {
      const style = this.inputs.find((input) => input.type === 'name').style;

      const { canvas } = this.$refs;
      const ctx = canvas.getContext('2d');

      ctx.font = `${style.fontWeight} ${style.fontSize}px ${style.fontFamily}`;

      ctx.textAlign = 'center'; // 가로 가운데 정렬
      ctx.textBaseline = 'middle'; // 세로 가운데 정렬

      const lines = text.split('\n');
      const lineHeight = style.fontSize * 1.5; // 행 간격
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

      ctx.textAlign = 'center'; // 가로 가운데 정렬
      ctx.textBaseline = 'middle'; // 세로 가운데 정렬
      ctx.font = `${style.fontWeight} ${style.fontSize}px ${style.fontFamily}`;
      ctx.fontSize = style.fontSize + 'px'; // 여기서 폰트 크기 설정

      const lines = text.split('\n');
      const lineHeight = style.fontSize * 1.5; // 행 간격
      const totalTextHeight = lines.length * lineHeight; // 텍스트 전체 높이
      const yStartPosition =
        (canvas.height - totalTextHeight) / 2 + style.fontSize / 2; // 이미지 위에 텍스트를 그리기 위해 조정

      lines.forEach((line, index) => {
        const y = yStartPosition + index * lineHeight;

        ctx.strokeStyle = `${style.textBorder}`;
        ctx.strokeStyle = style.fontColor;
        ctx.strokeText(line, canvas.width / 2, y);
        ctx.fillStyle = style.fontColor; // 폰트 색상 설정
        ctx.fillText(line, canvas.width / 2, y); // 텍스트 내용 그리기
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

.input-container {
  display: flex;
  align-items: center;
  margin-bottom: 20px;
  border-radius: 10px;
  background-color: white;
  padding: 8px;
}

.input-label {
  width: 5%;
}
</style>
