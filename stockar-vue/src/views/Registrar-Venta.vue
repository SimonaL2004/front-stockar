<template>
  <!-- Basado en Registrar-Venta.html con funcionalidad completa -->
  <div class="venta-form-container">
    <!-- Header con navegación -->
    <header class="u-black u-clearfix u-header u-header" id="header">
      <div class="u-clearfix u-sheet u-valign-middle u-sheet-1">
        <div class="custom-expanded u-clearfix u-custom-html u-custom-html-1">
          <div class="menu-usuario">
            <div class="menu-desplegable">
              <span>{{ currentUser?.empleado?.nombre }}</span>
              <a @click="handleLogout" href="#">Cerrar sesión</a>
            </div>
          </div>
        </div>
        <p class="u-text u-text-default u-text-1">
          <span style="font-size: 1.875rem;">Registrar Venta</span>
        </p>
        
        <!-- Navegación simplificada -->
        <nav class="drawer-nav">
          <router-link to="/dashboard">Inicio</router-link>
          <router-link to="/productos">Lista de repuestos</router-link>
          <router-link to="/productos/nuevo">Agregar repuesto</router-link>
          <router-link to="/ventas">Lista de ventas</router-link>
          <router-link to="/calcular-precio">Calcular precio</router-link>
        </nav>
      </div>
    </header>

    <!-- Sección principal -->
    <section class="u-clearfix u-section-1" id="block-2">
      <div class="u-clearfix u-sheet u-sheet-1">
        
        <!-- Búsqueda de productos -->
        <div class="u-form u-form-1">
          <div class="search-products">
            <label class="u-label">Buscar producto por código o nombre:</label>
            <div class="search-input-container">
              <input 
                v-model="searchProduct"
                @input="searchProducts"
                type="text" 
                placeholder="Ingrese código o nombre del producto"
                class="u-input u-input-rectangle search-input"
              />
              <button 
                @click="searchProducts"
                class="u-btn u-btn-submit search-btn"
                :disabled="loading"
              >
                🔍 Buscar
              </button>
            </div>
            
            <!-- Resultados de búsqueda -->
            <div v-if="searchResults.length > 0" class="search-results">
              <div 
                v-for="producto in searchResults" 
                :key="producto.idProducto"
                @click="selectProduct(producto)"
                class="search-result-item"
                :class="{ 'out-of-stock': producto.stock <= 0 }"
              >
                <div class="product-info">
                  <span class="product-code">{{ producto.codigo }}</span>
                  <span class="product-name">{{ producto.nombre }}</span>
                  <span class="product-price">${{ producto.precioVenta.toFixed(2) }}</span>
                  <span class="product-stock" :class="getStockClass(producto.stock)">
                    Stock: {{ producto.stock }}
                  </span>
                </div>
              </div>
            </div>
          </div>
        </div>

        <!-- Carrito de compras -->
        <div v-if="ventaItems.length > 0" class="u-group u-group-1">
          <div class="u-container-layout u-container-layout-1">
            <h3>Carrito de Compras</h3>
            
            <!-- Lista de items -->
            <div class="cart-items">
              <div 
                v-for="(item, index) in ventaItems" 
                :key="index"
                class="cart-item"
              >
                <div class="item-details">
                  <span class="item-code">{{ item.producto.codigo }}</span>
                  <span class="item-name">{{ item.producto.nombre }}</span>
                  <span class="item-price">${{ item.producto.precioVenta.toFixed(2) }}</span>
                </div>
                
                <!-- Cantidad -->
                <div class="quantity-controls">
                  <button 
                    @click="decreaseQuantity(index)"
                    class="qty-btn"
                    :disabled="item.cantidad <= 1"
                  >
                    -
                  </button>
                  <input 
                    v-model.number="item.cantidad"
                    @change="updateQuantity(index, $event)"
                    type="number" 
                    min="1" 
                    :max="item.producto.stock"
                    class="qty-input"
                  />
                  <button 
                    @click="increaseQuantity(index)"
                    class="qty-btn"
                    :disabled="item.cantidad >= item.producto.stock"
                  >
                    +
                  </button>
                </div>
                
                <!-- Subtotal -->
                <div class="item-subtotal">
                  <span>Subtotal: ${{ (item.cantidad * item.producto.precioVenta).toFixed(2) }}</span>
                </div>
                
                <!-- Eliminar -->
                <button 
                  @click="removeItem(index)"
                  class="remove-btn"
                  title="Eliminar producto"
                >
                  🗑️
                </button>
              </div>
            </div>

            <!-- Totales -->
            <div class="cart-totals">
              <div class="total-items">
                <strong>Total Items: {{ totalItems }}</strong>
              </div>
              <div class="total-amount">
                <strong>Total: ${{ totalAmount.toFixed(2) }}</strong>
              </div>
            </div>
            
            <!-- Información adicional de la venta -->
            <div class="sale-info">
              <div class="u-form-group">
                <label class="u-label">Observaciones (opcional):</label>
                <textarea 
                  v-model="ventaData.observaciones"
                  placeholder="Observaciones adicionales de la venta..."
                  class="u-input u-input-rectangle"
                  rows="3"
                ></textarea>
              </div>
            </div>

          </div>
        </div>

        <!-- Botones de acción -->
        <div v-if="ventaItems.length > 0" class="action-buttons">
          <button 
            @click="finalizarVenta"
            class="u-border-2 u-border-black u-btn u-button-style u-custom-color-1 u-hover-palette-2-base u-btn-3"
            :disabled="loading || ventaItems.length === 0"
          >
            {{ loading ? 'Procesando...' : 'Finalizar Venta' }}
          </button>
          
          <button 
            @click="anularVenta"
            class="u-border-2 u-border-black u-btn u-button-style u-custom-color-1 u-hover-palette-2-base u-btn-4"
            :disabled="loading"
          >
            Anular Venta
          </button>
        </div>

        <!-- Mensaje si no hay items -->
        <div v-else class="empty-cart">
          <p>No hay productos en el carrito. Busque y seleccione productos para comenzar la venta.</p>
        </div>

        <!-- Mensajes de error/éxito -->
        <div v-if="message" class="message" :class="messageType">
          {{ message }}
        </div>

      </div>
    </section>

    <!-- Footer -->
    <footer class="u-align-center u-black u-clearfix u-container-align-center u-footer u-footer" id="footer">
      <div class="u-clearfix u-sheet u-valign-middle u-sheet-1">
        <router-link to="/dashboard" class="u-image u-logo u-image-1">
          <img src="/images/logo-Photoroom.png" class="u-logo-image u-logo-image-1" alt="StockAR">
        </router-link>
      </div>
    </footer>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue';
import { useRouter } from 'vue-router';
import { authService, productoService, ventaService } from '@/services';
import type { Usuario, Producto } from '@/services';

const router = useRouter();

// Interfaces
interface VentaItem {
  producto: Producto;
  cantidad: number;
}

interface VentaData {
  observaciones: string;
  empleadoId: number;
  total: number;
  items: Array<{
    productoId: number;
    cantidad: number;
    precioUnitario: number;
    subtotal: number;
  }>;
}

// Estado reactivo
const currentUser = ref<Usuario | null>(null);
const loading = ref(false);
const searchProduct = ref('');
const searchResults = ref<Producto[]>([]);
const ventaItems = ref<VentaItem[]>([]);
const message = ref('');
const messageType = ref<'success' | 'error'>('success');

const ventaData = ref<Partial<VentaData>>({
  observaciones: ''
});

// Computed properties
const totalItems = computed(() => {
  return ventaItems.value.reduce((total, item) => total + item.cantidad, 0);
});

const totalAmount = computed(() => {
  return ventaItems.value.reduce((total, item) => {
    return total + (item.cantidad * item.producto.precioVenta);
  }, 0);
});

// Métodos
const searchProducts = async () => {
  if (searchProduct.value.trim().length < 2) {
    searchResults.value = [];
    return;
  }

  loading.value = true;
  try {
    // Buscar por código o nombre
    const allProducts = await productoService.getAll();
    const searchTerm = searchProduct.value.toLowerCase().trim();
    
    searchResults.value = allProducts.filter(producto => 
      producto.codigo.toString().includes(searchTerm) ||
      producto.nombre.toLowerCase().includes(searchTerm)
    ).slice(0, 10); // Limitar a 10 resultados
    
  } catch (error) {
    console.error('Error buscando productos:', error);
    showMessage('Error al buscar productos', 'error');
  } finally {
    loading.value = false;
  }
};

const selectProduct = (producto: Producto) => {
  if (producto.stock <= 0) {
    showMessage('Producto sin stock disponible', 'error');
    return;
  }

  // Verificar si ya está en el carrito
  const existingIndex = ventaItems.value.findIndex(item => 
    item.producto.idProducto === producto.idProducto
  );

  if (existingIndex >= 0) {
    // Si ya existe, aumentar cantidad
    if (ventaItems.value[existingIndex].cantidad < producto.stock) {
      ventaItems.value[existingIndex].cantidad++;
    } else {
      showMessage('No hay más stock disponible', 'error');
    }
  } else {
    // Agregar nuevo item
    ventaItems.value.push({
      producto,
      cantidad: 1
    });
  }

  // Limpiar búsqueda
  searchProduct.value = '';
  searchResults.value = [];
  showMessage('Producto agregado al carrito', 'success');
};

const increaseQuantity = (index: number) => {
  const item = ventaItems.value[index];
  if (item.cantidad < item.producto.stock) {
    item.cantidad++;
  } else {
    showMessage('No hay más stock disponible', 'error');
  }
};

const decreaseQuantity = (index: number) => {
  const item = ventaItems.value[index];
  if (item.cantidad > 1) {
    item.cantidad--;
  }
};

const updateQuantity = (index: number, event: Event) => {
  const target = event.target as HTMLInputElement;
  const newQuantity = parseInt(target.value);
  const item = ventaItems.value[index];
  
  if (newQuantity > 0 && newQuantity <= item.producto.stock) {
    item.cantidad = newQuantity;
  } else if (newQuantity > item.producto.stock) {
    item.cantidad = item.producto.stock;
    showMessage('Cantidad ajustada al stock disponible', 'error');
  } else {
    item.cantidad = 1;
  }
};

const removeItem = (index: number) => {
  ventaItems.value.splice(index, 1);
  showMessage('Producto eliminado del carrito', 'success');
};

const finalizarVenta = async () => {
  if (ventaItems.value.length === 0) {
    showMessage('No hay productos en el carrito', 'error');
    return;
  }

  if (!currentUser.value?.empleado?.idEmpleado) {
    showMessage('Error: Usuario no válido', 'error');
    return;
  }

  loading.value = true;
  try {
    // Preparar datos de la venta
    const ventaCompleta: VentaData = {
      observaciones: ventaData.value.observaciones || '',
      empleadoId: currentUser.value.empleado.idEmpleado,
      total: totalAmount.value,
      items: ventaItems.value.map(item => ({
        productoId: item.producto.idProducto,
        cantidad: item.cantidad,
        precioUnitario: item.producto.precioVenta,
        subtotal: item.cantidad * item.producto.precioVenta
      }))
    };

    // Enviar al backend
    const venta = await ventaService.create(ventaCompleta);
    
    showMessage('Venta registrada exitosamente', 'success');
    
    // Limpiar carrito
    ventaItems.value = [];
    ventaData.value.observaciones = '';
    
    // Redirigir a la lista de ventas o mostrar detalles
    setTimeout(() => {
      router.push(`/ventas/${venta.idVenta}`);
    }, 2000);
    
  } catch (error) {
    console.error('Error registrando venta:', error);
    showMessage('Error al registrar la venta', 'error');
  } finally {
    loading.value = false;
  }
};

const anularVenta = () => {
  if (confirm('¿Está seguro que desea anular la venta? Se perderán todos los datos.')) {
    ventaItems.value = [];
    ventaData.value.observaciones = '';
    searchProduct.value = '';
    searchResults.value = [];
    showMessage('Venta anulada', 'success');
  }
};

const getStockClass = (stock: number) => {
  if (stock <= 0) return 'no-stock';
  if (stock <= 5) return 'low-stock';
  return 'good-stock';
};

const showMessage = (text: string, type: 'success' | 'error') => {
  message.value = text;
  messageType.value = type;
  setTimeout(() => {
    message.value = '';
  }, 3000);
};

const handleLogout = async () => {
  if (confirm('¿Está seguro que desea cerrar sesión?')) {
    try {
      await authService.logout();
      router.push('/login');
    } catch (error) {
      console.error('Error en logout:', error);
      authService.clearStorage();
      router.push('/login');
    }
  }
};

// Lifecycle
onMounted(() => {
  currentUser.value = authService.getCurrentUser();
});
</script>

<style>
/* Importar CSS originales */
@import '/css/nicepage.css';
@import '/css/Registrar-Venta.css';
</style>