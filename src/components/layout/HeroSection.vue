<script setup>
import { ref, onMounted, onUnmounted } from "vue";
import Button from "@/components/UI/Button.vue";

// Constants
const SLIDE_INTERVAL = 3000;
const DESCRIPTION_SLIDE_INTERVAL = 8000; // Base interval
const DESCRIPTION_SLIDE_INTERVAL_FAST = 4000; // When no hover
const DESCRIPTION_SLIDE_INTERVAL_SLOW = 12000; // When hovered
const NAVBAR_OFFSET = 80;

// Image imports
import profile4_1 from "@/assets/images/profile4.1.jpg";
import profile1_1 from "@/assets/images/profile1.1.jpg";
import profile2_1 from "@/assets/images/profile2.1.jpg";
import profile8 from "@/assets/images/profile8.jpg";

// Reactive data
const images = ref([
  { src: profile4_1, alt: "Profile Picture 1" },
  { src: profile1_1, alt: "Profile Picture 2" },
  { src: profile2_1, alt: "Profile Picture 3" },
  { src: profile8, alt: "Profile Picture 4" },
]);

const currentIndex = ref(0);
const isAutoSlide = ref(true);
let slideInterval = null;

// Description slider state
const currentDescriptionIndex = ref(0);
const isAutoDescriptionSlide = ref(true);
let descriptionSlideInterval = null;
const isHovered = ref(false);
const lastHoverTime = ref(0);

// Touch/Mouse drag state
const isDragging = ref(false);
const startX = ref(0);
const currentX = ref(0);
const sliderRef = ref(null);

const descriptionSlides = ref([
  {
    id: 1,
    title: "เกี่ยวกับผม",
    icon: "mdi:account",
    iconColor: "text-blue-500",
    content: "ผม นายนัชชานนท์ เอ้งฉ้วน นักศึกษาชั้นปีที่ 4 คณะวิศวกรรมและเทคโนโลยีอุตสาหกรรม สาขาวิศวกรรมคอมพิวเตอร์ มหาวิทยาลัยราชภัฏสวนสุนันทา",
    bgColor: "bg-white/90 dark:bg-slate-800/90",
    borderColor: "border-slate-300/60 dark:border-slate-700/60"
  },
  {
    id: 2,
    title: "ความสนใจ",
    icon: "mdi:heart",
    iconColor: "text-pink-500",
    content: "มีความสนใจในการพัฒนาแอปพลิเคชันเว็บที่ใช้งานได้จริงและมีดีไซน์สวยงาม ชอบหาวิธีแก้ปัญหาผ่านเทคโนโลยีและพร้อมที่จะเรียนรู้สิ่งใหม่ๆ พร้อมที่จะพัฒนาตัวเองและร่วมงานในทุกโอกาสครับ",
    bgColor: "bg-gradient-to-br from-blue-50/90 to-purple-50/90 dark:from-slate-800/90 dark:to-slate-700/90",
    borderColor: "border-blue-300/60 dark:border-slate-600/60"
  }
]);

// Slider navigation functions
const nextSlide = () => {
  currentIndex.value = (currentIndex.value + 1) % images.value.length;
};

const prevSlide = () => {
  currentIndex.value =
    currentIndex.value === 0 ? images.value.length - 1 : currentIndex.value - 1;
};

const goToSlide = (index) => {
  currentIndex.value = index;
};

// Description slider functions
const nextDescriptionSlide = () => {
  currentDescriptionIndex.value = (currentDescriptionIndex.value + 1) % descriptionSlides.value.length;
};

const prevDescriptionSlide = () => {
  currentDescriptionIndex.value = currentDescriptionIndex.value === 0 
    ? descriptionSlides.value.length - 1 
    : currentDescriptionIndex.value - 1;
};

const goToDescriptionSlide = (index) => {
  currentDescriptionIndex.value = index;
};

// Drag functions
const handleStart = (e) => {
  isDragging.value = true;
  startX.value = e.type === 'mousedown' ? e.clientX : e.touches[0].clientX;
  stopAutoDescriptionSlide();
};

const handleMove = (e) => {
  if (!isDragging.value) return;
  
  e.preventDefault();
  currentX.value = e.type === 'mousemove' ? e.clientX : e.touches[0].clientX;
};

const handleEnd = () => {
  if (!isDragging.value) return;
  
  isDragging.value = false;
  const diffX = startX.value - currentX.value;
  const threshold = 50;
  
  if (Math.abs(diffX) > threshold) {
    if (diffX > 0) {
      // Swipe left - next slide
      nextDescriptionSlide();
    } else {
      // Swipe right - previous slide
      currentDescriptionIndex.value = currentDescriptionIndex.value === 0 
        ? descriptionSlides.value.length - 1 
        : currentDescriptionIndex.value - 1;
    }
  }
  
  startAutoDescriptionSlide();
};

// Auto slide control functions
const startAutoSlide = () => {
  if (!isAutoSlide.value) return;
  
  slideInterval = setInterval(nextSlide, SLIDE_INTERVAL);
};

const stopAutoSlide = () => {
  if (!slideInterval) return;
  
  clearInterval(slideInterval);
  slideInterval = null;
};

const startAutoDescriptionSlide = () => {
  if (!isAutoDescriptionSlide.value) return;
  
  // Clear existing interval
  if (descriptionSlideInterval) {
    clearInterval(descriptionSlideInterval);
  }
  
  // Calculate smart interval based on hover state and time
  const timeSinceLastHover = Date.now() - lastHoverTime.value;
  const isRecentlyHovered = timeSinceLastHover < 5000; // 5 seconds
  
  let interval;
  if (isHovered.value) {
    interval = DESCRIPTION_SLIDE_INTERVAL_SLOW; // 12 seconds when hovered
  } else if (isRecentlyHovered) {
    interval = DESCRIPTION_SLIDE_INTERVAL; // 8 seconds when recently hovered
  } else {
    interval = DESCRIPTION_SLIDE_INTERVAL_FAST; // 4 seconds when not hovered
  }
  
  descriptionSlideInterval = setInterval(nextDescriptionSlide, interval);
};

const stopAutoDescriptionSlide = () => {
  if (!descriptionSlideInterval) return;
  
  clearInterval(descriptionSlideInterval);
  descriptionSlideInterval = null;
};

const handleMouseEnter = () => {
  isHovered.value = true;
  lastHoverTime.value = Date.now();
  stopAutoDescriptionSlide();
  startAutoDescriptionSlide(); // Restart with slower interval
};

const handleMouseLeave = () => {
  isHovered.value = false;
  lastHoverTime.value = Date.now();
  stopAutoDescriptionSlide();
  startAutoDescriptionSlide(); // Restart with faster interval
};

// Scroll to contact section
const scrollToContact = () => {
  const contactSection = document.getElementById('contact');
  if (contactSection) {
    const offsetTop = contactSection.offsetTop - NAVBAR_OFFSET;
    window.scrollTo({
      top: offsetTop,
      behavior: 'smooth'
    });
  }
};

// Download resume function
const downloadResume = () => {
  const link = document.createElement('a');
  link.href = '/resume/Resume Nutchanon.pdf';
  link.download = 'Resume Nutchanon.pdf';
  link.target = '_blank';
  document.body.appendChild(link);
  link.click();
  document.body.removeChild(link);
};

// Lifecycle hooks
onMounted(() => {
  startAutoSlide();
  startAutoDescriptionSlide();
});

onUnmounted(() => {
  stopAutoSlide();
  stopAutoDescriptionSlide();
});
</script>

<template>
  <section
    data-aos="fade-up"
    class="relative min-h-screen flex items-center justify-center overflow-hidden overflow-x-hidden"
  >
    <!-- Background Elements -->
    <div class="absolute inset-0 overflow-hidden">
      <div
        class="absolute -top-40 -right-40 w-80 h-80 bg-gradient-to-br from-blue-400 to-purple-600 rounded-full mix-blend-multiply filter blur-xl opacity-20 animate-pulse"
        style="animation-duration: 5s;"
      />
      <div
        class="absolute -bottom-40 -left-40 w-80 h-80 bg-gradient-to-br from-pink-400 to-red-600 rounded-full mix-blend-multiply filter blur-xl opacity-20 animate-pulse"
        style="animation-duration: 7s; animation-delay: 1.5s;"
      />
      <div
        class="absolute top-1/2 left-1/2 transform -translate-x-1/2 -translate-y-1/2 w-96 h-96 bg-gradient-to-br from-yellow-400 to-orange-600 rounded-full mix-blend-multiply filter blur-xl opacity-10 animate-pulse"
        style="animation-duration: 9s; animation-delay: 3s;"
      />
    </div>

    <div class="relative z-10 container mx-auto px-4 py-20">
      <div class="grid lg:grid-cols-2 gap-12 items-center">
        <!-- Left Content -->
        <div class="text-center lg:text-left space-y-8" data-aos="fade-right">
          <!-- Greeting -->
          <div class="space-y-4">
            <h1 class="text-5xl lg:text-7xl font-bold leading-tight">
              <span class="text-slate-700 dark:text-slate-300">👋 Hi, I'm</span>
              <br />
              <span
                class="bg-gradient-to-r from-blue-600 via-purple-600 to-pink-600 bg-clip-text text-transparent"
              >
                Nutchanon Angchoun
              </span>
            </h1>

            <div
              class="flex items-center justify-center lg:justify-start space-x-4"
            >
              <div
                class="w-16 h-1 bg-gradient-to-r from-blue-500 to-purple-500 rounded-full"
              />
          <span
                class="text-lg font-medium text-slate-600 dark:text-slate-400"
                >Computer Engineering</span
              >
            </div>
          </div>

          <!-- Description Slider -->
          <div class="max-w-2xl mx-auto lg:mx-0">
            <div 
              ref="sliderRef"
              class="relative overflow-hidden rounded-xl cursor-grab active:cursor-grabbing select-none"
              @mouseenter="handleMouseEnter"
              @mouseleave="handleMouseLeave"
              @mousedown="handleStart"
              @mousemove="handleMove"
              @mouseup="handleEnd"
              @touchstart="handleStart"
              @touchmove="handleMove"
              @touchend="handleEnd"
            >
              <!-- Slider Container -->
              <div class="relative overflow-hidden rounded-xl group">
                <div 
                  class="flex transition-transform duration-500 ease-in-out"
                  :class="{ 'transition-none': isDragging }"
                  :style="{ 
                    transform: `translateX(-${currentDescriptionIndex * 100}%)`,
                    userSelect: 'none'
                  }"
                >
                  <div 
                    v-for="slide in descriptionSlides"
                    :key="slide.id"
                    class="w-full flex-shrink-0 px-2"
                  >
                    <div 
                      :class="`${slide.bgColor} backdrop-blur-sm rounded-xl p-4 border ${slide.borderColor}`"
                    >
                      <h3 class="text-lg font-semibold text-slate-800 dark:text-white mb-2 flex items-center">
                        <Icon :icon="slide.icon" :class="`${slide.iconColor} mr-2 text-sm`" />
                        {{ slide.title }}
                      </h3>
                      <p class="text-base leading-relaxed text-slate-600 dark:text-slate-300">
                        {{ slide.content }}
                      </p>
                    </div>
                  </div>
                </div>
              </div>

              <!-- Navigation Dots -->
              <div class="flex justify-center mt-4 space-x-2">
                <button
                  v-for="(slide, index) in descriptionSlides"
                  :key="index"
                  @click="goToDescriptionSlide(index)"
                  :class="[
                    'w-2 h-2 rounded-full transition-all duration-300',
                    currentDescriptionIndex === index
                      ? 'bg-blue-500 scale-125'
                      : 'bg-slate-300 dark:bg-slate-600 hover:bg-slate-400 dark:hover:bg-slate-500'
                  ]"
                />
              </div>
            </div>
          </div>

          <!-- Buttons -->
          <div
            class="flex flex-row gap-2 sm:gap-4 justify-center lg:justify-start"
          >
            <Button
              label="ติดต่อ"
              :onClick="scrollToContact"
              class="transform hover:scale-105 transition-all duration-300"
            />
          <button
            @click="downloadResume"
              class="group relative overflow-hidden rounded-2xl bg-slate-900 dark:bg-slate-800 text-white font-bold px-8 py-4 text-lg flex justify-center items-center transition-all duration-300 hover:scale-105 hover:shadow-2xl hover:shadow-green-500/25 border-2 border-slate-700 dark:border-slate-600 hover:border-green-500"
            >
              <!-- Animated background pattern -->
              <div
                class="absolute inset-0 bg-gradient-to-br from-green-500/0 via-emerald-500/0 to-teal-500/0 group-hover:from-green-500/10 group-hover:via-emerald-500/10 group-hover:to-teal-500/10 transition-all duration-500"
              />

              <!-- Content -->
              <div class="relative flex items-center space-x-3 z-10">
                <div class="relative">
                  <Icon
                    icon="line-md:download-loop"
                    class="text-2xl group-hover:animate-bounce"
                  />
                  <div
                    class="absolute -top-1 -right-1 w-3 h-3 bg-green-500 rounded-full group-hover:animate-ping"
                  />
                </div>
                <span
                  class="group-hover:text-green-400 transition-colors duration-300"
                >
                  Resume
                </span>
              </div>

              <!-- Corner accents -->
              <div
                class="absolute top-0 left-0 w-4 h-4 border-t-2 border-l-2 border-green-500 opacity-0 group-hover:opacity-100 transition-opacity duration-300"
              />
              <div
                class="absolute bottom-0 right-0 w-4 h-4 border-b-2 border-r-2 border-green-500 opacity-0 group-hover:opacity-100 transition-opacity duration-300"
              />

              <!-- Floating particles -->
              <div
                class="absolute top-2 left-4 w-1 h-1 bg-green-400 rounded-full opacity-0 group-hover:opacity-100 group-hover:animate-ping"
                style="animation-delay: 0.1s"
              />
              <div
                class="absolute top-4 right-6 w-1 h-1 bg-emerald-400 rounded-full opacity-0 group-hover:opacity-100 group-hover:animate-ping"
                style="animation-delay: 0.3s"
              />
              <div
                class="absolute bottom-3 left-6 w-1 h-1 bg-teal-400 rounded-full opacity-0 group-hover:opacity-100 group-hover:animate-ping"
                style="animation-delay: 0.5s"
              />
            </button>
          </div>
        </div>

        <!-- Right Content - Profile Image Slider -->
        <div class="flex justify-center lg:justify-end" data-aos="fade-left">
          <div
            class="relative group"
            @mouseenter="stopAutoSlide"
            @mouseleave="startAutoSlide"
          >
            <!-- Outer Glow -->
            <div
              class="absolute -inset-4 bg-gradient-to-r from-blue-500 via-purple-500 to-pink-500 rounded-3xl blur-2xl opacity-30 group-hover:opacity-50 transition duration-1000"
            />

            <!-- Image Container -->
            <div
              class="relative bg-white dark:bg-slate-800 p-2 rounded-3xl shadow-2xl"
            >
              <div class="relative overflow-hidden rounded-2xl">
                <!-- Image Slider -->
                <div class="relative w-80 h-96 lg:w-96 lg:h-[28rem]">
                  <div
                    class="flex transition-transform duration-500 ease-in-out"
                    :style="{
                      transform: `translateX(-${currentIndex * 100}%)`,
                    }"
        >
          <div
                      v-for="(image, index) in images"
                      :key="index"
                      class="w-full h-full flex-shrink-0"
          >
            <img
                        :src="image.src"
                        :alt="image.alt"
                        class="w-full h-full object-cover object-center"
              loading="lazy"
            />
                    </div>
                  </div>

                  <!-- Overlay Gradient -->
                  <div
                    class="absolute inset-0 bg-gradient-to-t from-black/20 via-transparent to-transparent"
                  />
                </div>
              </div>

              <!-- Navigation Arrows -->
              <button
                @click="prevSlide"
                class="absolute left-2 top-1/2 transform -translate-y-1/2 bg-white/80 dark:bg-slate-700/80 hover:bg-white dark:hover:bg-slate-700 rounded-full p-2 shadow-lg transition-all duration-300 opacity-0 group-hover:opacity-100"
              >
                <Icon
                  icon="mdi:chevron-left"
                  class="text-2xl text-slate-600 dark:text-slate-300"
                />
              </button>

              <button
                @click="nextSlide"
                class="absolute right-2 top-1/2 transform -translate-y-1/2 bg-white/80 dark:bg-slate-700/80 hover:bg-white dark:hover:bg-slate-700 rounded-full p-2 shadow-lg transition-all duration-300 opacity-0 group-hover:opacity-100"
              >
                <Icon
                  icon="mdi:chevron-right"
                  class="text-2xl text-slate-600 dark:text-slate-300"
                />
              </button>

              <!-- Dots Indicator -->
              <div
                class="absolute bottom-4 left-1/2 transform -translate-x-1/2 flex space-x-2"
              >
                <button
                  v-for="(image, index) in images"
                  :key="index"
                  @click="goToSlide(index)"
                  :class="[
                    'w-3 h-3 rounded-full transition-all duration-300',
                    currentIndex === index
                      ? 'bg-white shadow-lg scale-110'
                      : 'bg-white/50 hover:bg-white/80',
                  ]"
                />
              </div>

              <!-- Floating Elements -->
              <div
                class="absolute -top-4 -right-4 w-8 h-8 bg-blue-500 rounded-full animate-bounce"
              />
              <div
                class="absolute -bottom-4 -left-4 w-6 h-6 bg-purple-500 rounded-full animate-bounce"
                style="animation-delay: 0.5s"
              />
            </div>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>

<style scoped>
/* Custom animations */
@keyframes float {
  0%,
  100% {
    transform: translateY(0px);
  }
  50% {
    transform: translateY(-20px);
  }
}

.animate-float {
  animation: float 6s ease-in-out infinite;
}

/* Gradient text animation */
.bg-gradient-to-r {
  background-size: 200% 200%;
  animation: gradient 3s ease infinite;
}

@keyframes gradient {
  0% {
    background-position: 0% 50%;
  }
  50% {
    background-position: 100% 50%;
  }
  100% {
    background-position: 0% 50%;
  }
}
</style>
