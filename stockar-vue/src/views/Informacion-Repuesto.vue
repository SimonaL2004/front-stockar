<template>
  <!-- Basado en Información-Repuesto.html -->
  <div class="product-detail-container">
    <!-- Header con navegación -->
    <header class="u-black u-clearfix u-header u-header" id="header">
      <div class="u-clearfix u-sheet u-valign-middle-lg u-valign-middle-md u-valign-middle-sm u-valign-middle-xs u-sheet-1">
        <div class="custom-expanded u-clearfix u-custom-html u-custom-html-1">
          <div class="menu-usuario">
            <div class="menu-desplegable">
              <span>{{ currentUser?.empleado?.nombre }}</span>
              <a @click="handleLogout" href="#">Cerrar sesión</a>
            </div>
          </div>
        </div>
        <p class="u-text u-text-default u-text-1">
          <span style="font-size: 1.875rem;">Información Repuesto</span>
        </p>
        
        <!-- Navegación -->
        <nav class="drawer-nav">
          <router-link to="/dashboard">Inicio</router-link>
          <router-link to="/productos">Lista de repuestos</router-link>
          <router-link to="/productos/nuevo">Agregar repuesto</router-link>
          <router-link to="/ventas/nueva">Registrar venta</router-link>
          <router-link to="/calcular-precio">Calcular precio</router-link>
          <router-link to="/ventas">Lista de ventas</router-link>
        </nav>
      </div>
    </header>

    <!-- Loading state -->
    <div v-if="loading" class="loading-state">
      <div class="loading-spinner"></div>
      <p>Cargando producto...</p>
    </div>

    <!-- Error state -->
    <div v-else-if="error" class="error-state">
      <div class="error-icon">❌</div>
      <h3>Error al cargar el producto</h3>
      <p>{{ error }}</p>
      <button @click="loadProduct" class="btn btn-primary">Reintentar</button>
      <button @click="goBack" class="btn btn-secondary">Volver</button>
    </div>

    <!-- Sección principal -->
    <section v-else-if="producto" class="u-clearfix u-section-1" id="block-1">
      <div class="u-clearfix u-sheet u-sheet-1">
        
        <!-- Tabla de información -->
        <div class="u-table u-table-responsive u-table-1">
          <table class="u-table-entity">
            <colgroup>
              <col width="38.5%">
              <col width="61.5%">
            </colgroup>
            <tbody class="u-table-body">
              <tr style="height: 73px;">
                <td class="u-align-right u-border-1 u-border-grey-dark-1 u-table-cell">Tipo Producto</td>
                <td class="u-border-1 u-border-grey-dark-1 u-table-cell">
                  {{ producto.tipoProducto?.nombre || 'No especificado' }}
                </td>
              </tr>
              <tr style="height: 73px;">
                <td class="u-align-right u-border-1 u-border-grey-dark-1 u-table-cell">Marca</td>
                <td class="u-border-1 u-border-grey-dark-1 u-table-cell">
                  {{ producto.marca || 'No especificada' }}
                </td>
              </tr>
              <tr style="height: 75px;">
                <td class="u-align-right u-border-1 u-border-grey-dark-1 u-table-cell">Cantidad</td>
                <td class="u-border-1 u-border-grey-dark-1 u-table-cell">
                  {{ producto.stock }} unidades
                  <span v-if="isLowStock" class="stock-warning">⚠️ Stock Bajo</span>
                </td>
              </tr>
              <tr style="height: 77px;">
                <td class="u-align-right u-border-1 u-border-grey-dark-1 u-table-cell">Precio Neto</td>
                <td class="u-border-1 u-border-grey-dark-1 u-table-cell">
                  {{ formatCurrency(producto.precioNeto) }}
                </td>
              </tr>
              <tr style="height: 77px;">
                <td class="u-align-right u-border-1 u-border-grey-dark-1 u-table-cell">Precio de Venta</td>
                <td class="u-border-1 u-border-grey-dark-1 u-table-cell">
                  {{ formatCurrency(producto.precioVenta) }}
                  <small>(Ganancia: {{ profitMargin }}%)</small>
                </td>
              </tr>
              <tr style="height: 73px;">
                <td class="u-align-right u-border-1 u-border-grey-dark-1 u-table-cell">Moneda</td>
                <td class="u-border-1 u-border-grey-dark-1 u-table-cell">
                  {{ producto.moneda?.nombre || 'No especificada' }}
                </td>
              </tr>
            </tbody>
          </table>
        </div>

        <!-- Iconos originales -->
        <span class="u-file-icon u-icon u-icon-1">
          <img src="/images/1857132.png" alt="">
        </span>
        <span class="u-file-icon u-icon u-icon-2">
          <img src="/images/94699.png" alt="">
        </span>
        <span class="u-file-icon u-icon u-icon-3">
          <img src="/images/2979648.png" alt="">
        </span>
        <span class="u-file-icon u-icon u-icon-4">
          <img src="/images/1077976.png" alt="">
        </span>
        <span class="u-file-icon u-icon u-icon-5">
          <img src="/images/6109965.png" alt="">
        </span>
        <span class="u-file-icon u-icon u-icon-6">
          <img src="/images/869121.png" alt="">
        </span>

        <!-- Código y Nombre -->
        <p class="u-text u-text-default u-text-1">
          Código: {{ producto.codigo }}<br>
          Nombre: {{ producto.nombre }}
        </p>

        <!-- Selector de imagen (preview) -->
        <div class="u-clearfix u-custom-html u-custom-html-1">
          <div class="contenedor">
            <div id="preview" class="preview">
              Vista previa
            </div>
            <label class="boton-subir">Ver imagen</label>
          </div>
        </div>

        <!-- Descripción -->
        <p class="u-text u-text-default u-text-2">
          Descripción: {{ producto.descripcion || 'Sin descripción' }}
        </p>

        <!-- Botones de acción -->
        <div class="detail-actions">
          <button @click="goBack" class="u-btn u-btn-secondary">
            ← Volver a la Lista
          </button>
          <button 
            v-if="canEdit" 
            @click="editProduct" 
            class="u-btn u-btn-primary"
          >
            ✏️ Editar Producto
          </button>
          <button 
            v-if="canDelete" 
            @click="confirmDelete" 
            class="u-btn u-btn-danger"
          >
            🗑️ Eliminar Producto
          </button>
        </div>
      </div>
    </section>

    <!-- Footer -->
    <footer class="u-align-center u-black u-clearfix u-container-align-center u-footer u-footer" id="footer">
      <div class="u-clearfix u-sheet u-valign-middle-lg u-valign-middle-md u-valign-middle-sm u-valign-middle-xs u-sheet-1">
        <div class="u-preserve-proportions u-shape u-shape-circle u-white u-shape-1"></div>
        <router-link to="/dashboard" class="u-image u-logo u-image-1">
          <img src="/images/logo-Photoroom.png" class="u-logo-image u-logo-image-1" alt="StockAR">
        </router-link>
      </div>
    </footer>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue';
import { useRoute, useRouter } from 'vue-router';
import { productoService, authService, type Producto, type Usuario } from '@/services';

// Router
const route = useRoute();
const router = useRouter();

// State
const loading = ref(true);
const error = ref('');
const producto = ref<Producto | null>(null);
const currentUser = ref<Usuario | null>(null);

// Computed
const productId = computed(() => route.params.id as string);

const canEdit = computed(() => 
  authService.hasPermission('editar_producto') || authService.hasRole('admin')
);

const canDelete = computed(() => 
  authService.hasPermission('eliminar_producto') || authService.hasRole('admin')
);

const profitAmount = computed(() => {
  if (!producto.value) return 0;
  return producto.value.precioVenta - producto.value.precioNeto;
});

const profitMargin = computed(() => {
  if (!producto.value || !producto.value.precioNeto) return '0.00';
  const margin = (profitAmount.value / producto.value.precioNeto) * 100;
  return margin.toFixed(2);
});

const isLowStock = computed(() => {
  if (!producto.value) return false;
  return producto.value.stock <= 5; // Stock mínimo fijo por ahora
});

// Methods
const formatCurrency = (amount: number): string => {
  return new Intl.NumberFormat('es-AR', {
    style: 'currency',
    currency: 'ARS'
  }).format(amount);
};

const loadProduct = async (): Promise<void> => {
  try {
    loading.value = true;
    error.value = '';
    
    producto.value = await productoService.getById(Number(productId.value));
  } catch (err) {
    console.error('Error al cargar producto:', err);
    error.value = 'No se pudo cargar la información del producto';
  } finally {
    loading.value = false;
  }
};

const editProduct = (): void => {
  router.push(`/productos/${productId.value}/editar`);
};

const confirmDelete = async (): Promise<void> => {
  if (!producto.value) return;

  const confirmed = confirm(
    `¿Está seguro que desea eliminar el producto "${producto.value.nombre}"?\n\nEsta acción no se puede deshacer.`
  );

  if (confirmed) {
    try {
      await productoService.delete(Number(productId.value));
      alert('Producto eliminado exitosamente');
      router.push('/productos');
    } catch (err) {
      console.error('Error al eliminar producto:', err);
      alert('Error al eliminar el producto. Por favor intente nuevamente.');
    }
  }
};

const goBack = (): void => {
  router.push('/productos');
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
  loadProduct();
});
</script>

<style>
/* Importar CSS originales */
@import '/css/nicepage.css';
@import '/css/Informacion-Repuesto.css';
</style>