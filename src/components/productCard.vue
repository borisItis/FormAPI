<template>
  <section class="products">
    <div class="products__container">
      <h1 class="products__title">API Store</h1>
      <input
        class="products__input"
        v-model="searchValue"
        type="text"
        placeholder="Поиск..."
        autocomplete="on"
      />
      <div class="products__cards">
        <p v-if="searchValue.length <= 3" class="no-results"></p>
        <p v-else-if="isLoading" class="loading">
          ⏳ Загрузка товаров...
        </p>
        <p v-else-if="errorMessage" class="error">
          ⚠️ {{ errorMessage }}
        </p>
        <template v-else>
          <div
            v-for="product in products"
            :key="product.id"
            class="card"
          >
            <img :src="product.image" :alt="product.title" class="card__image" />
            <h2 class="card__title">{{ product.title }}</h2>
            <p class="card__description">
              {{ product.description.slice(0, 80) }}
            </p>
            <div class="card__bottom">
              <button class="card__btn" @click="buyProduct(product)">
                Купить
              </button>
              <span class="card__price">{{ product.price }} $</span>
            </div>
          </div>
          <p v-if="!products.length" class="no-results">
            Опппс... По вашему запросу ничего не найдено
          </p>
        </template>
      </div>
    </div>
  </section>
</template>

<script setup>
import { ref, watch } from "vue";

const searchValue = ref("");
const products = ref([]);
const isLoading = ref(false);
const errorMessage = ref("");
let timeoutId = null;

watch(searchValue, (newValue) => {
  clearTimeout(timeoutId);

  if (newValue.length <= 3) {
    products.value = [];
    errorMessage.value = "";
    return;
  }

  timeoutId = setTimeout(() => {
    fetchProducts();
  }, 500);
});

async function fetchProducts() {
  isLoading.value = true;
  errorMessage.value = "";
  products.value = [];

  try {
    const res = await fetch(
      `https://dummyjson.com/products/search?q=${encodeURIComponent(
        searchValue.value
      )}`
    );

    if (!res.ok) {
      throw new Error(`Ошибка сервера: ${res.status}`);
    }

    const data = await res.json();

    products.value = data.products.map((p) => ({
      id: p.id,
      title: p.title,
      description: p.description,
      price: p.price,
      image: p.thumbnail,
    }));
    if (!products.value.length) {
      errorMessage.value = "";
    }
  } catch (error) {
    errorMessage.value = "Ошибка при зарузке данных, Попробуйте позже.";
    console.error("Ошибка:", error);
  } finally {
    isLoading.value = false;
  }
}

watch(searchValue, () => {
  fetchProducts();
});

function buyProduct(product) {
  alert(`Спасибо за покупку "${product.title}", скоро с вами свяжемся!`);
}
</script>

<style lang="scss" scoped>
.products {
  &__container {
    max-width: 90rem;
    margin: 0 auto;
    padding: 1.25rem;
    text-align: center;
  }

  &__title {
    font-size: 1.75rem;
    margin-bottom: 1rem;
  }

  &__input {
    padding: 0.625rem 0.875rem;
    width: 100%;
    max-width: 25rem;
    border-radius: 0.5rem;
    border: 1px solid #ddd;
    margin-bottom: 1.5rem;
    outline: none;
  }

  &__cards {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(13.75rem, 1fr));
    gap: 1.25rem;

    .card {
      border: 1px solid #eee;
      border-radius: 1rem;
      padding: 1rem;
      background: #fff;
      box-shadow: 0 0.125rem 0.375rem rgba(0, 0, 0, 0.05);
      transition: transform 0.2s;
      display: flex;
      flex-direction: column;
      justify-content: space-between;

      &:hover {
        transform: scale(1.015);
        cursor: pointer;
      }

      &__image {
        width: 100%;
        height: 12.5rem;
        object-fit: contain;
      }

      &__title {
        font-size: 1rem;
        margin: 0.5rem 0;
        font-weight: 600;
      }

      &__description {
        font-size: 0.875rem;
        color: #555;
        height: 2.5rem;
      }

      &__bottom {
        display: flex;
        justify-content: space-between;
        align-items: center;
        margin-top: 0.75rem;
      }

      &__btn {
        padding: 0.5rem 1rem;
        background-color: #1e88e5;
        color: white;
        border: none;
        border-radius: 0.5rem;
        cursor: pointer;
        transition: background 0.2s;

        &:hover {
          background-color: #1669b7;
        }
      }

      &__price {
        font-weight: bold;
        color: #1e88e5;
      }
    }

    .no-results {
      grid-column: 1 / -1;
      color: #888;
    }
  }
}
</style>
