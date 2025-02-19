<template>
  <div class="h-full w-full">
    <div class="mb-6 flex items-start justify-between xs:mb-5">
      <div style="display: flex; gap: 8px; align-items: center">
        <h2 class="text-xl font-bold xs:text-base">
          Редактирование <span class="inline-block xs:hidden"> фото </span>
        </h2>
        <div style="display: flex; gap: 4px; top: 10px">
          <Button
            @click="navigateHistory(-1)"
            :disabled="historyImage.length <= 1 || indexForCut == 0"
            only-icon
            variant="light-gray"
            >
            <span >
              <
            </span>
          </Button>
          <Button
            @click="navigateHistory(1)"
            :disabled="historyImage.length <= 1 || indexForCut == historyImage.length - 1"
            style="transform: scale(-1, 1)"
            only-icon
            variant="light-gray"
            >            
            <span >
              <
            </span>
          </Button>
        </div>
      </div>
      <Icon
        @click="$emit('close')"
        name="close"
        class="h-5 w-5 transition-all hover:text-primary"
      />
    </div>
    <div class="flex w-full flex-col gap-6">
      <div class="flex gap-2">
        <Button
          @click="changeCatOptions(0, 'Настройки цвета')"
          :variant="activeBtn == 0 ? 'light-blue-outline' : 'secondary-outline'"
          
        >
          <span class="block xs:hidden"> Настройки цвета </span>
        </Button>
        <Button
          @click="changeCatOptions(1, 'Поворот')"
          :variant="activeBtn == 1 ? 'light-blue-outline' : 'secondary-outline'"
          
        >
          <span class="block xs:hidden"> Поворот </span>
        </Button>
        <Button
          @click="changeCatOptions(2, 'Кадрирование')"
          :variant="activeBtn == 2 ? 'light-blue-outline' : 'secondary-outline'"
          
        >
          <span class="block xs:hidden"> Кадрирование </span>
        </Button>
        <Button
          @click="changeCatOptions(3, 'Отражение')"
          :variant="activeBtn == 3 ? 'light-blue-outline' : 'secondary-outline'"
          
        >
          <span class="block xs:hidden"> Отражение </span>
        </Button>
        <div class="block md:hidden">
          <Button
            @click="changeCatOptions(4, 'История')"
            :variant="activeBtn == 4 ? 'light-blue-outline' : 'secondary-outline'"
            
          >
            <span class="block xs:hidden"> История </span>
          </Button>
        </div>
      </div>
      <div class="flex h-[520px] gap-6 md:h-full sm:flex-col xs:gap-5">
        <div
          ref="stageWrapper"
          class="flex h-full w-full max-w-[800px] items-center justify-center border-gray-300 bg-gray-600 sm:h-[400px] xs:h-[200px]"
        >
          <Loader v-if="isLoading" />
          <v-stage v-show="!isLoading" ref="stageRef" :config="configStage">
            <v-layer ref="layerRef">
              <v-image ref="imageNode" :config="imageConfig" />
            </v-layer>
            <v-layer ref="dimLayer" v-if="selected">
              <v-rect ref="rectRef" :config="rectCrop" v-if="selected" />
              <v-transformer ref="tranRef" :config="tranConfig" v-if="selected" />
            </v-layer>
          </v-stage>
        </div>
        <div class="flex max-w-[240px] flex-col sm:w-full sm:max-w-full">
          <div class="flex flex-col gap-8" v-if="activeBtn == 0">
            <div class="text-lg font-semibold sm:hidden">Настройки цвета</div>
            <div class="flex min-w-32 flex-col gap-4 w-max">
              <div class="flex w-[200px] flex-col gap-1.5 xs:w-[160px]">
                <span class="xs:text-xs"> Насыщенность </span>
                <Slider v-model="saturationValue" :min="-2" :max="2" :step="0.1" />
              </div>
              <div class="flex w-[200px] flex-col gap-1.5 xs:w-[160px]">
                <span class="xs:text-xs"> Яркость </span>
                <Slider v-model="brightnessValue" :min="-1" :max="1" :step="0.01" />
              </div>
              <div class="flex w-[200px] flex-col gap-1.5 xs:w-[160px]">
                <span class="xs:text-xs"> Контраст </span>
                <Slider v-model="contrastValue" :min="-100" :max="100" :step="1" />
              </div>
            </div>
          </div>
          <div class="flex flex-col gap-8" v-if="activeBtn == 1">
            <div class="text-lg font-semibold sm:hidden">Поворот</div>
            <div class="flex min-w-32 flex-col gap-2">
              <Button
                class="default min-w-fit"
                @click="(dirty = true), rotate(90)"
                variant="light-outline"
                style="color: #282b2e"
              >
                <Icon name="reload" color="#3B82F6" style="transform: scale(-1, 1)" />
                Вправо на 90°
              </Button>
              <Button
                class="default"
                @click="(dirty = true), rotate(-90)"
                variant="light-outline"
                style="color: #282b2e"
              >
                <Icon color="#3B82F6" name="reload" />
                Влево на 90°
              </Button>
            </div>
          </div>
          <div class="flex flex-col gap-8" v-if="activeBtn == 2">
            <div class="text-lg font-semibold sm:hidden">Кадрирование</div>
            <div class="flex min-w-32 flex-col gap-2">
              <Button
                class="default"
                @click="selectImage(), (dirty = true)"
                variant="light-outline"
                style="color: #282b2e"
              >
                <Icon color="#3B82F6" name="minimize" />
                Кадрировать
              </Button>
              <Button
                class="default"
                @click="changeCatOptions(2, 'Обрезка')"
                variant="light-outline"
                style="color: #282b2e"
              >
                <Icon color="#3B82F6" name="success" />
                Применить
              </Button>
            </div>
          </div>
          <div class="flex flex-col gap-8" v-if="activeBtn == 3">
            <div class="text-lg font-semibold sm:hidden">Отражение</div>
            <div class="flex min-w-32 flex-col gap-2">
              <Button
                class="default"
                @click="flip('x'), (dirty = true)"
                variant="light-outline"
                style="color: #282b2e"
              >
                <Icon color="#3B82F6" name="flip" class="rotate-90" />
                По горизонтали
              </Button>
              <Button
                class="default"
                @click="flip('y'), (dirty = true)"
                variant="light-outline"
                style="color: #282b2e"
              >
                <Icon color="#3B82F6" name="flip" />
                По вертикали
              </Button>
            </div>
          </div>
          <div class="flex flex-col gap-8 overflow-y-auto overflow-hidden" v-if="activeBtn == 4">
            <span class="text-lg font-semibold sm:hidden">История</span>
            <div
              class="flex flex-col gap-2 border-b border-b-gray-400 pb-4"
              v-for="(item, index) in historyImage"
              :key="item.date"
            >
              <div class="flex justify-start">
                <Icon color="#D7D7D7" name="clock" />
                <span>{{
                  `${new Date(item.date).getHours()}:${
                    String(new Date(item.date).getMinutes()).length == 1
                      ? '0' + new Date(item.date).getMinutes()
                      : new Date(item.date).getMinutes()
                  }`
                }}</span>
              </div>
              <span class="w-max">{{ item.title }}</span>
              <span
              class="text-[#3b82f6] border-b border-dashed border-[#3b82f6] w-max cursor-pointer"
                @click="loadHistory(index), (indexForCut = index)"
              >
                Вернуть
              </span>
            </div>
          </div>
        </div>
      </div>
      <div class="sticky !bottom-0 mt-auto flex w-full gap-2 bg-white pb-10 sm:py-7 xs:py-5">
        <Button @click="saveChanges(), $emit('saveImage', imageObj as HTMLImageElement )">
          Сохранить
        </Button>
        <Button @click="$emit('close')" variant="light-gray" style="color: #3b82f6">Отмена</Button>
      </div>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { onBeforeUnmount, onMounted, ref, watchEffect } from 'vue';
import { useDebounceFn, useThrottleFn } from '@vueuse/core';
import Konva from 'konva';

import Button from './ui/button/Button.vue';
import Loader from './Loader.vue';

import Slider from './ui/slider/Slider.vue';

// БАЗА
// Входные параметры
interface IEditorProps {
  defImg: string;
  root?: HTMLDivElement;
  innerWidth?: number;
}

const props = defineProps<IEditorProps>();

// События
defineEmits<{
  (event: 'saveImage', data: { src: string }): void;
  (event: 'close'): void;
}>();

// ПЕРЕМЕННЫЕ
// Яркость
const brightnessValue = ref(0);
// Контраст
const contrastValue = ref(0);
// Насыщенность
const saturationValue = ref(0);
// В состоянии загрузки?
const isLoading = ref(false);
// Изображение
const imageObj = ref<HTMLImageElement | null>(null);
// Сцена(холст)
const stageRef = ref();
// Основной слой
const layerRef = ref();
// Слой кадрирования
const dimLayer = ref();
// Компонент изображения
const imageNode = ref();
// Компонент трансформации
const tranRef = ref();
// Фигура к которой привязана трансформация
const rectRef = ref();
// Активно кадрирование?
const selected = ref(false);
// Были изменения изображния?
const dirty = ref(false);
// Текущая активная категория
const activeBtn = ref(0);
// История изменений
const historyImage = ref<any>([]);
// Заголовок для истории
const title = ref('Настройки цвета');
// Индекс по которому отрезается лишняя история
const indexForCut = ref<number | null>(0);
// Реф обертки сцены
const stageWrapper = ref<HTMLDivElement>();

// КОНФИГИ
// Конфиг сцены
const configStage = ref({ width: 400, height: 400 });

// Конфиг изображения
const imageConfig = ref({
  x: 0,
  y: 0,
  image: new Image(),
  width: 0,
  height: 0,
  rotation: 0,
  offsetX: 0,
  offsetY: 0,
  filters: [Konva.Filters.Brighten, Konva.Filters.Contrast],
  brightness: 0,
  contrast: 0,
  saturation: 0,
});

// Конфиг для фигуры кадрирования
var rectCrop = ref({
  x: 0,
  y: 0,
  width: 0,
  height: 0,
  offsetX: 0,
  offsetY: 0,
  fill: 'transparent',
  strokeWidth: 1,
  strokeScaleEnabled: true,
  draggable: true,
});

// Конфиг для компонента трансформации, для кадрирования
let tranConfig = ref({
  nodes: [] as Konva.Node[],
  centeredScaling: false,
  rotateEnabled: false,
  keepRatio: false,
  anchorStyleFunc: (anchor: Konva.Rect) => {
    anchor.cornerRadius(10);
    anchor.attrs.stroke = 'transparent';
    anchor.attrs.fill = '#3B82F6';
    anchor.attrs.opacity = 1;
  },
});

// РАБОТАЕМ С ПАРАМЕТРАМИ РАЗМЕРОВ
// Функция установки параметров изображения
const setParams = () => {
  if (imageObj.value) {
    if (stageWrapper.value) {
      imageConfig.value.width = imageObj.value.width;
      imageConfig.value.height = imageObj.value.height;

      configStage.value.width = stageWrapper.value.clientWidth;
      configStage.value.height = stageWrapper.value.clientHeight;

      imageConfig.value.x = configStage.value.width / 2;
      imageConfig.value.y = configStage.value.height / 2;

      imageConfig.value.rotation = 0;

      imageConfig.value.offsetX = imageConfig.value.width / 2;
      imageConfig.value.offsetY = imageConfig.value.height / 2;
    }
  }
};

// Функция масштабирования изображения
const scale = () => {
  if (imageObj.value) {
    if (stageWrapper.value) {
      if (
        stageWrapper.value.clientHeight > imageObj.value.height &&
        stageWrapper.value.clientWidth > imageObj.value.width
      ) {
        imageNode.value.getNode().attrs.scaleX = 1;
        imageNode.value.getNode().attrs.scaleY = 1;
      } else {
        if (
          imageObj.value.width > imageObj.value.height &&
          imageObj.value.height * (stageWrapper.value.clientWidth / imageObj.value.width) <
            stageWrapper.value.clientHeight
        ) {
          imageNode.value.getNode().attrs.scaleX =
            stageWrapper.value.clientWidth / imageObj.value.width;
          imageNode.value.getNode().attrs.scaleY =
            stageWrapper.value.clientWidth / imageObj.value.width;
        } else {
          imageNode.value.getNode().attrs.scaleX =
            stageWrapper.value.clientHeight / imageObj.value.height;
          imageNode.value.getNode().attrs.scaleY =
            stageWrapper.value.clientHeight / imageObj.value.height;
        }
      }
    }
  }
  imageNode.value.getNode().clearCache();
};

// ГРУЗИМ
// Функция загрузки изображения с промисом
function loadImageAsync(src: string) {
  isLoading.value = true;
  return new Promise((resolve) => {
    imageObj.value = new Image();
    imageObj.value.src = src;
    imageObj.value.onload = resolve;
  });
}

// Обертка чтобы дождаться загрузки
const loadImage = async (src: string) => {
  await loadImageAsync(src).then(() => {
    imageObj.value ? (imageConfig.value.image = imageObj.value) : null;
  });

  isLoading.value = false;
  setParams();
};

// ИСТОРИЯ
// Добавляем историю измнения изображения
const addHistory = (title: string, src: string) => {
  historyImage.value.push({
    title: title,
    src: src,
    date: Number(new Date()),
  });
  indexForCut.value = historyImage.value.length - 1;
};

// Навигация по истории
const navigateHistory = (index: number) => {
  if (indexForCut.value == null) return;
  if (index > 0 && indexForCut.value > historyImage.value.length) return;
  if (index < 0 && indexForCut.value < 0) return;

  indexForCut.value += index;
  
  loadHistory(indexForCut.value)
};

const loadHistory = async (index: number) => {
  if (indexForCut.value == null) return;
  await loadImage(historyImage.value[index].src);
  scale();
}

// Удаляем лишние истории
const deleteUnactualHistory = () => {
  if (
    (activeBtn.value == 4 || dirty.value == true) &&
    historyImage.value.length > 1 &&
    indexForCut.value != null
  ) {
    historyImage.value.splice(indexForCut.value + 1);
  }
};

// СОБСТВЕННО РАБОТАЕМ С ИЗОБРАЖЕНИЕМ
// Фукнция смены категории изменений
const changeCatOptions = (cat: number, str: string) => {
  deleteUnactualHistory();
  console.log(str);
  console.log( title.value);
  dirty.value ? saveChanges() : null;
  activeBtn.value = cat;
  title.value = str;
  dirty.value = false;
};

// Активация кадрирования
function selectImage() {
  rectCrop.value.width = imageConfig.value.width * imageNode.value.getNode().scaleX();
  rectCrop.value.height = imageConfig.value.height * imageNode.value.getNode().scaleY();
  rectCrop.value.x = (configStage.value.width - rectCrop.value.width) / 2;
  rectCrop.value.y = (configStage.value.height - rectCrop.value.height) / 2;
  selected.value = true;
}

// Отражение
const flip = useThrottleFn((plane: string) => {
  if (plane == 'x') {
    imageNode.value.getNode().to({
      scaleX: -imageNode.value.getNode().scaleX(),
    });
  } else {
    imageNode.value.getNode().to({
      scaleY: -imageNode.value.getNode().scaleY(),
    });
  }
}, 1000);

// Вращение
const rotate = (num: number) => {
  imageConfig.value.rotation += num;
  if (imageConfig.value.rotation >= 360) {
    imageConfig.value.rotation = 0;
  }
  if (imageObj.value) {
    const oldWidth = imageObj.value.width;
    imageObj.value.width = imageObj.value?.height;
    imageObj.value.height = oldWidth;
    scale();
  }
};

// Яркость
const applyBrightness = useDebounceFn((value: number) => {
  imageNode.value.getNode().attrs.brightness = value;
  applyFilters();
}, 100);

// Контраст
const applyContrast = useDebounceFn((value: number) => {
  imageNode.value.getNode().attrs.contrast = value;
  applyFilters();
}, 100);

// Насыщенность
const applySaturation = useDebounceFn((value: number) => {
  imageNode.value.getNode().attrs.saturation = value;
  applyFilters();
}, 100);

// Применение фильтров
function applyFilters() {
  imageNode.value
    .getNode()
    .filters([Konva.Filters.Brighten, Konva.Filters.Contrast, Konva.Filters.HSL]);
  imageNode.value.getNode().cache();
  imageNode.value.getNode().getLayer().batchDraw();
}

// Сохранение изменений изображения(не путать с сохранением результата редактирования)
async function saveChanges() {
  const image = imageNode.value.getNode();
  const oldScaleX = image.attrs.scaleX;
  const oldScaleY = image.attrs.scaleY;
  image.attrs.scaleX < 0 ? (image.attrs.scaleX = -1) : (image.attrs.scaleX = 1);
  image.attrs.scaleY < 0 ? (image.attrs.scaleY = -1) : (image.attrs.scaleY = 1);

  let changedImg;
  // Либо обрезаем, либо просто сохраняем результат
  if (tranRef.value) {
    image.clearCache();
    const transformer = tranRef.value.getNode();
    const crop = {
      x:
        (transformer.x() - (configStage.value.width - rectCrop.value.width) / 2) / oldScaleX +
        image.x() -
        imageConfig.value.offsetX,
      y:
        (transformer.y() - (configStage.value.height - rectCrop.value.height) / 2) / oldScaleY +
        image.y() -
        imageConfig.value.offsetY,
      width: transformer.width() / oldScaleX,
      height: transformer.height() / oldScaleY,
      mimeType: 'image/jpeg',
      imageSmoothingEnabled: false,
    };
    changedImg = image.toDataURL(crop);
  } else {
    // Сохраняем фильтры
    const filters = {
      saturation: imageNode.value.getNode().attrs.saturation,
      contrast: imageNode.value.getNode().attrs.contrast,
      brightness: imageNode.value.getNode().attrs.brightness,
    };
    // Чистим кэш чтобы сбросить размер изображения до изначального
    image.clearCache();

    // Приминяем фильтры
    imageNode.value.getNode().attrs.brightness = filters.brightness;
    imageNode.value.getNode().attrs.contrast = filters.contrast;
    imageNode.value.getNode().attrs.saturation = filters.saturation;
    applyFilters();

    changedImg = image.toDataURL({
      mimeType: 'image/jpeg',
      imageSmoothingEnabled: false,
    });
  }
  selected.value = false;
  // Грузим измененное изображение
  addHistory(title.value, changedImg as string);
  await loadImage(changedImg);

  brightnessValue.value = 0;
  contrastValue.value = 0;
  saturationValue.value = 0;
  // Вот эта строчка отвечает за то чтобы всё отрисовалось
  image.getLayer().batchDraw();
  scale();
}

// МАУНТЫ И ВОТЧЕРЫ
// Базовые штуки при монтировании
onMounted(async () => {
  await loadImage(props.defImg);
  addHistory('Оригинал', imageObj.value?.src as string);
  title.value = 'Настройки цвета'
  scale();
});

// Следим за ресайзом окна и скейлим изображние
onMounted(() => {
  const resizeObserver = new ResizeObserver((entries) => {
    for (const entry of entries) {
      scale();
    }
  });
  if (props.root) {
    resizeObserver.observe(props.root);
  }
  window.addEventListener('resize', scale);

  onBeforeUnmount(() => {
    if (props.root) {
      resizeObserver.unobserve(props.root);
    }
    window.removeEventListener('resize', scale);
  });
});

// Следим за наличием фигуры для кадрирования
watchEffect(() => {
  // Ноды фигуры привязываем к нодам трансформа
  rectRef.value ? (tranConfig.value.nodes = [rectRef.value.getNode()]) : null;
  // Задаем параметры для трансформа
  if (rectRef.value && rectRef.value.getNode) {
    dimLayer.value.getNode().clipFunc(function (ctx: {
      beginPath: () => void;
      rect: (arg0: number, arg1: number, arg2: any, arg3: any) => void;
      closePath: () => void;
      fillStyle: string;
      fill: (arg0: string) => void;
    }) {
      ctx.beginPath();
      ctx.rect(0, 0, stageRef.value?.getNode().width(), stageRef.value.getNode().height());
      ctx.rect(
        rectRef.value?.getNode().x(),
        rectRef.value?.getNode().y(),
        rectRef.value?.getNode().width(),
        rectRef.value?.getNode().height(),
      );
      ctx.closePath();
      ctx.fillStyle = 'rgba(0, 0, 0, 0.5)';
      ctx.fill('evenodd');
    });
    imageNode.value.getNode().getLayer().batchDraw();
    rectRef.value.getNode().on('transform', () => {
      var scaleX = rectRef.value?.getNode().scaleX();
      var scaleY = rectRef.value?.getNode().scaleY();
      var width = rectRef.value?.getNode().width() * scaleX;
      var height = rectRef.value?.getNode().height() * scaleY;
      rectRef.value?.getNode().width(width);
      rectRef.value?.getNode().height(height);
      rectRef.value?.getNode().scaleX(1);
      rectRef.value?.getNode().scaleY(1);

      imageNode.value.getNode().getLayer().batchDraw();
    });
  }
});

// При изменени яркости и тд, эти изменения применяются и ставится пометка что они были
// (dirty, чтобы при переходе на другую категорию сохранились изменения)
watchEffect(() => {
  applyBrightness(brightnessValue.value), brightnessValue.value != 0 ? (dirty.value = true) : null;
  applyContrast(contrastValue.value), contrastValue.value != 0 ? (dirty.value = true) : null;
  applySaturation(saturationValue.value), saturationValue.value != 0 ? (dirty.value = true) : null;
});
</script>

<style lang="scss" scoped>
.default {
  border: 1px solid #e9e9e9;
}
.header_btns {
  display: flex;
  gap: 8px;
  .active {
    background-color: #f4f6f9;
    border: 1px solid transparent;
  }
}
.editor__wrapper {
  display: flex;
  gap: 24px;
  max-height: 60vh;
  .stage__wrapper {
    display: flex;
    align-items: center;
    justify-content: center;
    // min-width: 500px;
    // min-height: 500px;
    max-width: calc(100% - 200px);
    width: 100%;
    height: 60vh;
    border: 1px solid #e9e9e9;
  }
  .controls__wrapper {
    display: flex;
    flex-direction: column;
    width: 200px;
    max-width: 200px;
    .wrapper {
      display: flex;
      flex-direction: column;
      gap: 16px;
      margin-bottom: 32px;
      .history__header {
        margin: 0;
        font-weight: 700;
        margin-bottom: 32px;
      }
    }
  }
}
.footer_btns {
  display: flex;
  gap: 8px;
}

.input__wrapper {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}

@media (max-width: 768px) {
  .editor__wrapper {
    flex-direction: column;
    max-height: 70vh;
    .stage__wrapper {
      height: 45vh;
      width: 100%;
      max-width: 100%;
    }
    .controls__wrapper {
      height: 100%;
      width: 100%;
      max-width: 100%;
      .wrapper {
        display: grid;
        grid-template-columns: repeat(2, 1fr);
      }
    }
  }
}
</style>
