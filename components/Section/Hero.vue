<script setup lang="ts">
const mouseX = ref(0);
const mouseY = ref(0);
const scrollY = ref(0);
const rainCanvas = ref<HTMLCanvasElement | null>(null);
let animationId: number;

const handleMouseMove = (event: MouseEvent) => {
  mouseX.value = event.clientX;
  mouseY.value = event.clientY;
};

const backgroundLayerStyles = computed(() => ({
  transform: `translate3d(${mouseX.value * 0.005}px, ${
    mouseY.value * 0.005
  }px, 0) scale(1.02)`,
}));

const firstLayerStyles = computed(() => ({
  transform: `translate3d(${mouseX.value * 0.02}px, ${
    scrollY.value * 0.1 + mouseY.value * 0.02
  }px, 0) scale(1.05)`,
}));

const highlightLayerStyles = computed(() => ({
  transform: `translate3d(${mouseX.value * 0.03}px, ${
    scrollY.value * 0.15 + mouseY.value * 0.03
  }px, 0) scale(1.1)`,
}));

const initializeRain = () => {
  if (!rainCanvas.value) return;
  resizeCanvas();
  const context = rainCanvas.value.getContext("2d");
  if (!context) return;

  for (let i = 0; i < 140; i++) {
    rainArray.push(
      new RainDrop(
        Math.random() * window.innerWidth,
        Math.random() * -500,
        randomNum(10, 2),
        randomNum(20, 0.2),
        Math.random() * 0.55
      )
    );
  }
  animateRain(context);
};

const animateRain = (context: CanvasRenderingContext2D) => {
  animationId = requestAnimationFrame(() => animateRain(context));
  context.clearRect(0, 0, window.innerWidth, window.innerHeight);
  rainArray.forEach((drop) => drop.update(context));
};

const resizeCanvas = () => {
  if (rainCanvas.value) {
    rainCanvas.value.width = window.innerWidth;
    rainCanvas.value.height = window.innerHeight;
  }
};

const randomNum = (max: number, min: number) =>
  Math.random() * (max - min) + min;

class RainDrop {
  constructor(
    public x: number,
    public y: number,
    public endY: number,
    public velocity: number,
    public opacity: number
  ) {}

  draw(context: CanvasRenderingContext2D) {
    context.beginPath();
    context.moveTo(this.x, this.y);
    context.lineTo(this.x, this.y - this.endY);
    context.lineWidth = 1;
    context.strokeStyle = `rgba(255, 255, 255, ${this.opacity})`;
    context.stroke();
  }

  update(context: CanvasRenderingContext2D) {
    const rainEnd = window.innerHeight + 100;
    this.y = this.y >= rainEnd ? this.endY - 100 : this.y + this.velocity;
    this.draw(context);
  }
}

const rainArray: RainDrop[] = [];

onMounted(() => {
  window.addEventListener("scroll", () => (scrollY.value = window.scrollY));
  initializeRain();
  window.addEventListener("resize", resizeCanvas);
});

onUnmounted(() => {
  window.removeEventListener("resize", resizeCanvas);
  cancelAnimationFrame(animationId);
});
</script>

<template>
  <div
    class="relative overflow-hidden h-screen w-full"
    @mousemove="handleMouseMove"
  >
    <img
      src="@/assets/images/backgroundLeaves.png"
      :style="backgroundLayerStyles"
      class="absolute inset-0 w-full h-full object-cover z-0"
    />

    <div class="absolute inset-0 flex items-center justify-center z-10">
      <h1
        class="text-white text-[3rem] font-bold opacity-70 drop-shadow-lg lg:text-[12rem] md:text-[9rem] sm:text-[5rem]"
      >
        Syngonium
      </h1>
    </div>

    <img
      src="@/assets/images/leaves2.png"
      :style="firstLayerStyles"
      class="absolute inset-0 z-20 opacity-100 object-cover w-full h-full"
    />

    <img
      src="@/assets/images/blick.png"
      :style="highlightLayerStyles"
      class="absolute inset-0 z-30 opacity-50 object-cover w-full h-full"
    />

    <canvas
      ref="rainCanvas"
      class="absolute inset-0 pointer-events-none z-40 w-full h-full"
    ></canvas>
  </div>
</template>
