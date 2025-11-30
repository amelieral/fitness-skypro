<template>
  <div class="course-card">
    <div class="course-image-wrapper">
      <img :src="courseImage" :alt="course.nameRU" class="course-main-image" />
      <h2 class="visually-hidden">{{ course.nameRU }}</h2>
    </div>

    <div class="target-section">
      <h3 class="section-title">Подойдет для вас, если:</h3>

      <div class="target-list">
        <div
          v-for="(fit, index) in course.fitting"
          :key="index"
          class="target-item"
          :class="`target-item-${index + 1}`"
        >
          <span class="number">{{ index + 1 }}</span>
          <p class="fit-text">{{ fit }}</p>
        </div>
      </div>
    </div>

    <div class="directions-section">
      <h3 class="section-title">Направления:</h3>
      <div class="directions-wrapper">
        <ul class="directions-list">
          <li
            v-for="(direction, index) in course.directions"
            :key="index"
            class="direction-item"
          >
            <img src="../assets/img/icon/Star.svg" />
            {{ direction }}
          </li>
        </ul>
      </div>
    </div>

    <div class="mobile-image-containe">
      <img src="../assets/img/course/Group-mobile.png" class="mobile-image" />
    </div>

    <div class="benefits-section">
      <div class="content-wrapper">
        <h3 class="section-title-benefit">
          Начните путь<br />
          к новому телу:
        </h3>
        <ul class="benefits-list">
          <li class="benefit-item">Проработка всех групп мышц</li>
          <li class="benefit-item">Тренировка суставов</li>
          <li class="benefit-item">Улучшение циркуляции крови</li>
          <li class="benefit-item">Упражнения заряжают бодростью</li>
          <li class="benefit-item">Помогают противостоять стрессам</li>
        </ul>

        <button
          v-if="!isAuthenticated"
          class="auth-button"
          @click="handleAuthRedirect"
        >
          Войдите, чтобы добавить курс
        </button>

        <div v-else class="course-actions">
          <button
            class="auth-button"
            :class="{ added: isCourseAdded }"
            :disabled="isProcessing"
            @click="handleCourseToggle"
          >
            {{ isCourseAdded ? "Удалить курс" : "Добавить курс" }}
            <span v-if="isProcessing" class="loading-indicator">⌛</span>
          </button>
        </div>
      </div>

      <div class="image-overlay">
        <img src="../assets/img/course/vector.png" class="background-image" />
        <img src="../assets/img/course/man.png" class="foreground-image" />
      </div>
    </div>
  </div>
</template>

<script setup>
import { useUserStore } from "@/stores/user";
import { useCoursesStore } from "@/stores/courses";

const props = defineProps({
  course: {
    type: Object,
    required: true,
    validator: (c) => {
      return ["nameRU", "directions", "fitting"].every((f) => f in c);
    },
  },
});

const router = useRouter();
const route = useRoute();
const userStore = useUserStore();
const coursesStore = useCoursesStore();
const isProcessing = ref(false);

const isMobile = ref(false);
const checkDeviceType = () => {
  isMobile.value = window.matchMedia("(max-width: 768px)").matches;
};

const courseImage = computed(() => {
  const basePath = "../assets/img/";
  const deviceFolder = isMobile.value ? "main/" : "course/";

  try {
    return new URL(
      `${basePath}${deviceFolder}${props.course.nameEN}.png`,
      import.meta.url
    ).href;
  } catch {
    console.warn("Используется fallback-изображение");
    return new URL(`${basePath}default.png`, import.meta.url).href;
  }
});

onMounted(() => {
  checkDeviceType();
  window.addEventListener("resize", checkDeviceType);
  console.log("Компонент инициализирован");
});

onBeforeUnmount(() => {
  window.removeEventListener("resize", checkDeviceType);
});

const isAuthenticated = computed(() => !!userStore.currentUser?.user);

watch(
  () => userStore.currentUser,
  (newVal) => {
    console.log("Пользователь обновился:", newVal);
  }
);
const isCourseAdded = computed(() =>
  userStore.currentUser?.user?.selectedCourses?.includes(props.course._id)
);

const handleCourseToggle = async () => {
  isProcessing.value = true;

  try {
    if (isCourseAdded.value) {
      await coursesStore.removeCourse(props.course._id);
    } else {
      await coursesStore.addCourse(props.course._id);
    }
  } catch (error) {
    console.error("Ошибка:", error);
    alert(error.message);
  } finally {
    isProcessing.value = false;
  }
};

onMounted(() => {
  console.log("Компонент смонтирован");
  console.log("Текущий пользователь:", userStore.currentUser);
  console.log("Проверка курса:", {
    id: props.course._id,
    name: props.course.nameRU,
  });
});

const handleAuthRedirect = () => {
  const redirectPath = route.fullPath;

  router.push({
    path: "/auth",
    query: { redirect: redirectPath },
  });
};
</script>

<style scoped>
.course-card {
  background: #ffffff;
  border-radius: 12px;
  padding: 0;
  box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
  margin-bottom: 2rem;
}

.course-image-wrapper {
  position: relative;
  overflow: hidden;
  border-radius: 16px;

  .course-main-image {
    width: 100%;
    height: 100%;
    object-fit: cover;
    object-position: center;
    display: block;
  }
}

.visually-hidden {
  position: absolute;
  width: 1px;
  height: 1px;
  margin: -1px;
  padding: 0;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  border: 0;
}

.target-section,
.directions-section,
.benefits-section {
  padding-top: 2rem;
}

.section-title {
  color: #374151;
  font-size: 2.5rem;
  margin-bottom: 1rem;
  position: relative;

  &::before {
    display: none;
  }
}

.section-title::before {
  content: "";
  position: absolute;
  left: 0;
  top: 50%;
  transform: translateY(-50%);
  width: 8px;
  height: 8px;
  background: #3b82f6;
  border-radius: 50%;
}

.target-list {
  display: flex;
  flex-wrap: wrap;
  gap: 1rem;
  justify-content: space-between;
}

.target-item {
  background: #222;
  color: #fff;
  padding: 20px;
  border-radius: 28px;
  display: flex;
  align-items: center;
  gap: 10px;
  height: 141px;
  box-sizing: border-box;
  flex: 0 0 auto;
}

.target-item-1 {
  width: 368px;
}

.target-item-2 {
  width: 431px;
}

.target-item-3 {
  width: 327px;
}

.target-item .number {
  font-size: 75px;
  color: #bcec30;
  flex-shrink: 0;
}

.target-item .fit-text {
  font-size: 24px;
  font-weight: 400;
  line-height: 110%;
  margin: 0;
}

.directions-wrapper {
  background: #bcec30;
  border-radius: 12px;
  padding: 1rem;
}

.directions-list {
  display: flex;
  flex-wrap: wrap;
  gap: 1rem;
  padding: 0;
  list-style: none;
  width: 100%;
}

.direction-item {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 10px 15px;
  font-size: 24px;
  border-radius: 8px;
  flex: 1 1 calc(33.33% - 2rem);
}

.benefits-list {
  list-style: none;
  padding-left: 1.5rem;
  margin-bottom: 1rem;
}

.target-list li,
.directions-list li,
.benefits-list li {
  padding: 0.5rem 0;
  display: flex;
  align-items: center;
  gap: 0.8rem;
}

.section-title-benefit {
  margin-top: 40px;
  font-size: 60px;
  margin-bottom: 28px;
}

.content-wrapper {
  margin-left: 2rem;
  margin-bottom: 1.5rem;
  padding: 1rem;
  background-color: #ffffff;
  display: flex;
  flex-direction: column;
}

.benefits-section {
  position: relative;
  padding: 1.5rem;
  background: #f0f4ff;
  border-radius: 8px;
  margin: 1.5rem 0;
  overflow: visible;
  margin-top: 0;
  position: relative;
  padding-top: 0;
  z-index: 2;
}

.image-overlay {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
}

.background-image {
  position: absolute;
  left: 0;
  width: 100%;
  height: 110%;
  z-index: 1;
}

.foreground-image {
  position: absolute;
  top: -15%;
  right: 5%;
  width: 40%;
  object-fit: contain;
  z-index: 3;
}

.mobile-image {
  display: none;
}

.auth-button {
  width: 437px;
  padding: 1rem;
  background: #bcec30;
  color: #000000;
  border: none;
  border-radius: 26px;
  font-size: 1rem;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.8rem;
  transition: background 0.3s ease;
  z-index: 9999;
}

.auth-button:hover {
  background: #a8d52b;
}

@media (max-width: 768px) {
  .course-card {
    padding: 1.5rem;
  }

  .course-title {
    font-size: 1.5rem;
  }

  .section-title {
    font-size: 1.1rem;
  }

  .target-list {
    flex-direction: column;
    gap: 0.8rem;
  }

  .target-item {
    flex: 1 1 100% !important;
    width: 100% !important;
    min-width: unset;
    height: 343px;
    padding: 1rem;
    border-radius: 6px;
  }

  .directions-list {
    flex-direction: column;
    gap: 0.5rem;
  }

  .image-overlay {
    position: relative;
  }

  .background-image,
  .foreground-image {
    display: none;
  }

  .mobile-image-container {
    position: relative;
    z-index: 1;
    height: 180px;
    margin: -40px 0 -60px;
  }

  .mobile-image {
    position: absolute;
    display: block;
    width: 100%;
    left: 0;
    top: 1050px;
  }

  .directions-section {
    position: relative;
  }

  .benefits-section {
    position: relative;
  }

  .content-wrapper {
    position: static;
    width: 100%;
    min-width: 0;
    max-width: none;
    height: auto;
    margin: 0;
    padding: 0;
    transform: none;
    background: transparent;
    box-shadow: none;
    display: block;
    box-sizing: border-box;
    padding: 15px;

    .section-title-benefit,
    .benefits-list {
      max-width: 100%;
      padding-left: 15px;
      padding-right: 15px;
    }
  }
  h3 {
    font-size: 32px;
  }
  .auth-button {
    width: 100%;
  }
}
</style>
