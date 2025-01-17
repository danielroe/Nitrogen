<script setup lang="ts">
import type { ProductQueryVariables } from '@@/types/shopify';

// Route data
const route = useRoute();
const handle = computed(() => route.params.handle as string);

// Stores
const shopStore = useShopStore();

// Shopify
const shopify = useShopify();

// Fetch data
const productVars = computed<ProductQueryVariables>(() => ({
  handle: handle.value,
  country: shopStore.buyerCountryCode,
  language: shopStore.buyerLanguageCode
}));

const { data: productData } = await useAsyncData(
  `product-${handle.value}`,
  () => shopify.product.get(productVars.value),
  { watch: [productVars] }
);

// Computed data
const product = computed(() => productData.value);
const productMedia = computed(() => flattenConnection(product.value?.media));

// Get related products (if any)
const relatedProducts = computed(() => {
  const references = product.value?.related_products?.references;
  return references ? flattenConnection(references) : [];
});

// State
const mediaIndex = ref<number>(0);
const isLightboxOpen = ref(false);

// Actions
function openLightbox(index: number) {
  mediaIndex.value = index;
  isLightboxOpen.value = true;
};

function closeLightbox() {
  isLightboxOpen.value = false;
};

// Watchers
const isScrollLocked = useScrollLock(document);

watch(isLightboxOpen, (isOpen) => {
  isScrollLocked.value = isOpen;
});

// SEO
useHead({
  title: product.value?.title
});
</script>

<template>
  <section v-if="product" class="relative grid gap-10 lg:grid-cols-2 lg:gap-0">
    <div class="flex flex-col">
      <ProductMediaGallery
        :product-media="productMedia"
        @open-lightbox="openLightbox"
      />
      <ProductMediaCarousel
        :product-media="productMedia"
      />
    </div>
    <div class="flex flex-col px-6">
      <ProductForm
        :product="product"
        :related-products="relatedProducts"
      />
    </div>
  </section>
  <section v-else class="flex items-center p-6 gap-2">
    <Icon name="ph:seal-warning" class="h-5 w-5 shrink-0" />
    <p class="normal-case">No Product data found.</p>
  </section>
  <ProductMediaLightbox
    v-if="isLightboxOpen"
    :media-index="mediaIndex"
    :product-media="productMedia"
    @close-lightbox="closeLightbox"
  />
</template>
