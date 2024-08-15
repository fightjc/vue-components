<script>
/**
 * 翻页时钟
 */
export default {
  name: 'FlipClock',
  data() {
    return {
      colElms: [],
      last: 0
    }
  },
  mounted() {
    this.last = new Date().getSeconds();
    this.update();
    setInterval(() => {
      this.update();
    }, 1000);
  },
  methods: {
    getTimeStr() {
      const date = new Date();
      return [date.getHours(), date.getMinutes(), date.getSeconds()]
        .map((item) => item.toString().padStart(2, "0"))
        .join("");
    },
    update() {
      let curr = new Date().getSeconds();
      if (this.last === curr) return;
      this.last = curr;
      const currentTimeStr = this.getTimeStr();
      this.colElms.forEach((col, i) => {
        var currTxt = col.curr.elm.dataset.t;
        [col.flipNext, col.next].map(el => el.elm.dataset.t = currentTimeStr[i]);
        if (currTxt !== currentTimeStr[i]) {
          col.flip.elm.classList.toggle("active");
          setTimeout(() => {
            col.flip.elm.classList.toggle("active");
            [col.flipCurr, col.curr].map(el => el.elm.dataset.t = currentTimeStr[i]);
          }, 500);
        }
      });
    }
  },
  render(createElement, context) {
    var colElms = [];

    const createCol = () => {
      const [ col, flip, flipNext, flipCurr, next, curr ] =
          ["col", "flip", "next", "curr", "next", "curr"].map(className => createElement('div', {
            class: className,
            attrs: { 'data-t': 0 } // 给予初始值
          }));
      flip.children = [ flipNext, flipCurr ];
      col.children = [ flip, next, curr ];
      return { col, flip, flipNext, flipCurr, next, curr };
    };

    const timeStr = this.getTimeStr();
    for (let i = 0; i < 6; i++) {
      var col = createCol();
      [col.flipCurr, col.curr].forEach((el) => el.data.attrs['data-t'] = timeStr[i]);
      colElms.push(col);
    }
    const timeElem = createElement('div', { class: 'time' }, colElms.map(c => c.col))

    this.colElms = colElms; // 最后赋值防止二次修改导致重复调用render

    return createElement('div', [ timeElem ]);
  }
}
</script>

<style lang="scss" scoped>
$--body-bg: #333;   // 网页背景颜色
$--font-size: 130px;  // 时钟字体大小
$--center-border: 3px solid #000; // 翻页中间的边框色
$--col-gap: 8px;      // 横向时间块间隔
$--col-width: 100px;  // 6个时间块的宽度
$--col-height: 160px; // 时钟高度
$--col-radius: 10px;  // 时间块边框弧度
$--col-color: #ddd; // 时钟字体颜色
$--col-bg: #1a1a1a; // 时钟背景色

.time {
  // inset: 0;
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
  height: 100%;
  gap: $--col-gap;
  background-color: $--body-bg;
  font-family: sans-serif;
  font-weight: 700;
  overflow: hidden;
}

.col {
  width: $--col-width;
  height: $--col-height;
  perspective: $--col-height;
}
.col:nth-child(3),
.col:nth-child(5) {
  margin-left: calc($--col-gap * 4);
}

.curr,
.next {
  position: relative;
  width: $--col-width;
  height: calc($--col-height / 2);
  font-size: $--font-size;
  background: $--col-bg;
  border-radius: $--col-radius;
  color: $--col-color;
  overflow: hidden;
  box-sizing: border-box;
}

.flip .curr::before,
.flip .next::before,
.col > .curr::before,
.col > .next::before {
  position: absolute;
  content: attr(data-t);
  line-height: $--col-height;
  text-align: center;
  height: $--col-height;
  left: 0;
  right: 0;
}

.flip .curr::before,
.col > .next::before {
  top: 0;
}

.flip .next::before,
.col > .curr::before {
  bottom: 0;
}

.flip .curr,
.col > .next {
  border-bottom: $--center-border;
}

.flip .next,
.col > .curr {
  border-top: $--center-border;
}

.flip .next {
  transform: rotateX(-180deg);
  backface-visibility: hidden;
}

.flip .curr {
  position: absolute;
  top: 0;
  backface-visibility: hidden;
}

.flip {
  position: absolute;
  width: $--col-width;
  height: $--col-height;
  z-index: 1;
  transform-style: preserve-3d;
  transition: transform 0s;
  transform: rotateX(0);
}

.flip.active {
  transition: all 0.5s ease-in-out;
  transform: rotateX(-180deg);
}
</style>