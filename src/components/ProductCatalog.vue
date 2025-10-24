<script>
export default {
  name: "ProductCatalog",
  data() {
    return {
      // 4. State untuk melacak ID produk saat ini (1-20)
      currentProductId: 1,

      // 5. State untuk menyimpan produk yang memenuhi kriteria
      product: null,

      // State untuk status loading (opsional)
      isLoading: false,

      // State untuk melacak kategori (digunakan untuk Class Binding)
      categoryType: "unavailable", // Nilai awal: 'unavailable'
    };
  },

  // Method ini akan dijalankan saat komponen pertama kali dimuat
  mounted() {
    this.fetchNextProduct();
  },

  methods: {
    // 4 & 5. Logika Navigasi dan Fetch API
    async fetchNextProduct() {
      // Atur status loading
      this.isLoading = true;

      // 6. Atur Index (looping 1-20)
      // Jika sudah mencapai 21, kembalikan ke 1
      if (this.currentProductId > 20) {
        this.currentProductId = 1;
      }

      // Fetch API
      const url = `https://fakestoreapi.com/products/${this.currentProductId}`;

      try {
        const response = await fetch(url);
        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`);
        }
        const data = await response.json();

        // 5. Cek Kondisi Kategori
        const category = data.category.toLowerCase();

        if (category === "men's clothing" || category === "women's clothing") {
          // Kategori DITERIMA: Simpan data dan set tipe kategori
          this.product = data;
          this.categoryType = category === "men's clothing" ? "men" : "women";
        } else {
          // Kategori TIDAK DITERIMA: Kosongkan data dan set tipe 'unavailable'
          this.product = null;
          this.categoryType = "unavailable";
        }
      } catch (error) {
        console.error("Error fetching product:", error);
        this.categoryType = "unavailable"; // Set ke unavailable jika fetch gagal
        this.product = null;
      } finally {
        // Increment ID untuk tombol 'Next' dan matikan loading
        this.currentProductId++;
        this.isLoading = false;
      }
    },
  },

  // Computed untuk membantu class binding di template
  computed: {
    // 4. Menggunakan Class Binding untuk menentukan desain
    mainContainerClass() {
      // Mengembalikan class sesuai dengan categoryType
      return {
        "page-men": this.categoryType === "men",
        "page-women": this.categoryType === "women",
        "page-unavailable": this.categoryType === "unavailable",
      };
    },
  },
};
</script>

<template>
  <div class="product-wrapper">
    <!-- Menggunakan Class Binding berdasarkan Computed Property -->
    <div :class="['product-card', mainContainerClass]">
      <!-- Konten Loading -->
      <div v-if="isLoading" class="loading-state">
        <div class="spinner"></div>
        <p>Memuat produk ID {{ currentProductId - 1 }}...</p>
      </div>

      <!-- Konten Utama (Jika tidak loading) -->
      <div v-else class="content-display">
        <!-- Desain: UNAVAILABLE PRODUCT -->
        <div v-if="categoryType === 'unavailable'" class="unavailable-state">
          <h2>Produk ID {{ currentProductId - 1 }}</h2>
          <p class="not-found-message">
            Produk tidak tersedia di kategori 'Men's Clothing' atau 'Women's
            Clothing'.
          </p>
          <p class="category-name">
            Kategori: {{ product ? product.category : "N/A" }}
          </p>
        </div>

        <!-- Desain: MEN/WOMEN SECTION -->
        <div v-else class="available-state">
          <div class="product-header">
            <p class="category-name">
              {{
                categoryType === "men" ? "Men's Clothing" : "Women's Clothing"
              }}
            </p>
            <h1 class="product-title">{{ product.title }}</h1>
          </div>

          <div class="product-body">
            <div class="image-section">
              <img
                :src="product.image"
                :alt="product.title"
                class="product-image"
              />
            </div>

            <div class="details-section">
              <p class="product-description">{{ product.description }}</p>
              <div class="rating-info">
                Rating: {{ product.rating.rate }} / 5 ({{
                  product.rating.count
                }}
                reviews)
              </div>
              <p class="product-price">
                Rp. {{ product.price.toFixed(2).replace(".", ",") }}
              </p>
            </div>
          </div>
        </div>
      </div>

      <!-- Tombol Next Product -->
      <div class="button-section">
        <button @click="fetchNextProduct" :disabled="isLoading">
          Next Product (ID: {{ currentProductId }})
        </button>
      </div>
    </div>
  </div>
</template>

<style>
/* 3. CSS Variables (Color Palette) */
:root {
  --color-dark-blue: #001524;
  --color-light-blue: #a3b1c2;
  --color-red: #d62839;
  --color-yellow: #fca311;
  --color-white: #ffffff;
  --color-men-bg: #d6e0f0;
  --color-women-bg: #f9d8e7;
  --color-unavailable-bg: #f0f0f0;
}

body {
  background-color: var(--color-light-blue);
  margin: 0;
  padding: 0;
  font-family: sans-serif;
}

/* Base Style Container */
.product-wrapper {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
  padding: 20px;
}

.product-card {
  width: 100%;
  max-width: 800px;
  min-height: 500px;
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.2);
  border-radius: 10px;
  display: flex;
  flex-direction: column;
  transition: background-color 0.5s ease, border 0.5s ease;
  position: relative;
}

.content-display {
  flex-grow: 1;
  padding: 30px;
}

/* 4. CLASS BINDING - DESIGN SECTIONS */

/* Page-Men Section (Hijau & Biru Tua) */
.page-men {
  background-color: var(--color-men-bg);
  border: 3px solid var(--color-dark-blue);
  color: var(--color-dark-blue);
}
.page-men .product-title {
  color: var(--color-dark-blue);
}
.page-men .product-price {
  font-size: 2rem;
  color: var(--color-dark-blue);
}

/* Page-Women Section (Pink & Ungu) */
.page-women {
  background-color: var(--color-women-bg);
  border: 3px solid #6a0572; /* Darker Purple for Women */
  color: #6a0572;
}
.page-women .product-title {
  color: #6a0572;
}
.page-women .product-price {
  font-size: 2rem;
  color: #6a0572;
}

/* Page-Unavailable Product */
.page-unavailable {
  background-color: var(--color-unavailable-bg);
  border: 3px dashed var(--color-red);
  color: var(--color-red);
}
.unavailable-state {
  text-align: center;
  padding: 50px;
}
.not-found-message {
  font-size: 1.2rem;
  font-weight: bold;
}
.unavailable-state h2 {
  color: #333;
}

/* Struktur Konten */
.product-body {
  display: flex;
  gap: 30px;
  margin-top: 20px;
}
.image-section {
  flex-basis: 35%;
  display: flex;
  justify-content: center;
  align-items: center;
}
.product-image {
  max-width: 100%;
  max-height: 200px;
  object-fit: contain;
  border: 1px solid #ddd;
  padding: 10px;
  background-color: var(--color-white);
}
.details-section {
  flex-basis: 65%;
  text-align: left;
}
.product-description {
  line-height: 1.6;
  margin-bottom: 15px;
}
.rating-info {
  font-weight: bold;
  color: var(--color-yellow);
  margin-bottom: 10px;
}

/* Button & Loading */
.button-section {
  padding: 20px;
  text-align: center;
  border-top: 1px solid rgba(0, 0, 0, 0.1);
}
.button-section button {
  padding: 12px 25px;
  font-size: 1.1rem;
  font-weight: bold;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  transition: background-color 0.3s;
}
.page-men .button-section button {
  background-color: var(--color-dark-blue);
  color: var(--color-white);
}
.page-women .button-section button {
  background-color: #6a0572;
  color: var(--color-white);
}
.page-unavailable .button-section button {
  background-color: var(--color-red);
  color: var(--color-white);
}

/* Loading State (Optional CSS) */
.loading-state {
  text-align: center;
  padding: 100px;
}
.spinner {
  border: 4px solid rgba(0, 0, 0, 0.1);
  border-top: 4px solid var(--color-dark-blue);
  border-radius: 50%;
  width: 40px;
  height: 40px;
  animation: spin 1s linear infinite;
  margin: 0 auto 10px;
}
@keyframes spin {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}

/* Responsive Design */
@media (max-width: 600px) {
  .product-body {
    flex-direction: column;
  }
  .image-section {
    order: -1; /* Pindahkan gambar ke atas */
    margin-bottom: 15px;
  }
}
</style>
