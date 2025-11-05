<template>
  <div class="dashboard-container">
    <!-- Usar el diseño original del menú principal -->
    <section class="u-clearfix u-image u-shading u-section-1" id="sec-81bb">
      <div class="u-clearfix u-sheet u-sheet-1">
        <img class="u-image u-image-default u-image-1" src="/images/logo-Photoroom.png" alt="" data-image-width="400" data-image-height="400">
        
        <!-- Usuario logueado -->
        <div class="user-welcome">
          <h2>Bienvenido, {{ currentUser?.empleado?.nombre }}</h2>
          <button @click="handleLogout" class="logout-btn">Cerrar Sesión</button>
        </div>
        
        <!-- Botones del menú principal -->
        <router-link to="/productos" class="u-btn u-button-style u-hover-palette-1-light-1 u-palette-1-base u-btn-1">Lista de Repuestos</router-link>
        <router-link to="/productos/nuevo" class="u-btn u-button-style u-hover-palette-1-light-1 u-palette-1-base u-btn-2">Agregar Repuesto</router-link>
        <router-link to="/ventas" class="u-btn u-button-style u-hover-palette-1-light-1 u-palette-1-base u-btn-3">Lista de Ventas</router-link>
        <router-link to="/ventas/nueva" class="u-btn u-button-style u-hover-palette-1-light-1 u-palette-1-base u-btn-4">Registrar Venta</router-link>
        <router-link to="/calcular-precio" class="u-btn u-button-style u-hover-palette-1-light-1 u-palette-1-base u-btn-5">Calcular Precio</router-link>
      </div>
    </section>

    <!-- Estadísticas rápidas (opcional) -->
    <div class="stats-section" v-if="showStats">
      <div class="container">
        <div class="stats-grid">
          <div class="stat-card">
            <div class="stat-icon">�</div>
            <div class="stat-content">
              <h3>{{ stats.totalProductos }}</h3>
              <p>Total Productos</p>
            </div>
          </div>
          
          <div class="stat-card">
            <div class="stat-icon">⚠️</div>
            <div class="stat-content">
              <h3>{{ stats.stockBajo }}</h3>
              <p>Stock Bajo</p>
            </div>
          </div>
          
          <div class="stat-card">
            <div class="stat-icon">💰</div>
            <div class="stat-content">
              <h3>{{ stats.ventasHoy }}</h3>
              <p>Ventas Hoy</p>
            </div>
          </div>
          
          <div class="stat-card">
            <div class="stat-icon">💵</div>
            <div class="stat-content">
              <h3>${{ cotizacionUsd }}</h3>
              <p>USD Oficial</p>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, computed } from 'vue';
import { useRouter } from 'vue-router';
import { authService, productoService, ventaService, cotizacionService } from '@/services';
import type { Producto, Venta, Usuario } from '@/services';

const router = useRouter();

// Estado reactivo
const loading = ref(false);
const showStats = ref(false); // Estadísticas opcionales
const currentUser = ref<Usuario | null>(null);
const productos = ref<Producto[]>([]);
const recentVentas = ref<Venta[]>([]);
const cotizacionUsd = ref(0);

// Stats computadas
const stats = computed(() => ({
  totalProductos: productos.value.length,
  stockBajo: productos.value.filter(p => p.stock <= 5).length,
  ventasHoy: recentVentas.value.filter(v => 
    new Date(v.fechaHora).toDateString() === new Date().toDateString()
  ).length
}));

// Métodos
const loadData = async () => {
  loading.value = true;
  try {
    // Cargar datos en paralelo
    const [productosData, ventasData, cotizacion] = await Promise.all([
      productoService.getAll(),
      ventaService.getAll(),
      cotizacionService.getUsdOficial()
    ]);
    
    productos.value = productosData;
    recentVentas.value = ventasData.slice(-5); // Últimas 5 ventas
    cotizacionUsd.value = cotizacion;
    
  } catch (error) {
    console.error('Error cargando datos del dashboard:', error);
  } finally {
    loading.value = false;
  }
};

const handleLogout = async () => {
  if (confirm('¿Está seguro que desea cerrar sesión?')) {
    try {
      await authService.logout();
      router.push('/login');
    } catch (error) {
      console.error('Error en logout:', error);
      // Forzar logout local
      authService.clearStorage();
      router.push('/login');
    }
  }
};

// Lifecycle
onMounted(() => {
  currentUser.value = authService.getCurrentUser();
  loadData();
});
</script>

<style>
/* Importar CSS originales */
@import '/css/nicepage.css';
@import '/css/MenuPrincipal.css';
</style>