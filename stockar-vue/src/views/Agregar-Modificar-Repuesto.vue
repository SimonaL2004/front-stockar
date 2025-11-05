<template>
  <!-- Basado en Agregar-Repuesto.html y Modificar-Repuesto.html -->
  <div class="product-form-container">
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
          <span style="font-size: 1.875rem;">{{ isEditing ? 'Modificar' : 'Agregar' }} Repuesto</span>
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

    <!-- Sección principal -->
    <section class="u-clearfix u-section-1" id="block-4">
      <div class="u-clearfix u-sheet u-valign-middle u-sheet-1">
        
        <!-- Selector de imagen (opcional por ahora) -->
        <div class="u-clearfix u-custom-html u-custom-html-1">
          <div class="contenedor">
            <div id="preview" class="preview">
              {{ imagePreview || 'Vista previa' }}
            </div>
            <label for="imagen" class="boton-subir">Seleccionar imagen</label>
            <input 
              type="file" 
              id="imagen" 
              accept="image/*" 
              @change="handleImageChange"
              ref="imageInput"
            >
          </div>
        </div>

        <!-- Formulario principal -->
        <div class="u-form u-form-1">
          <form @submit.prevent="handleSubmit" class="u-clearfix u-form-spacing-10 u-form-vertical u-inner-form" style="padding: 10px;">
            
            <!-- Código -->
            <div class="u-form-group u-form-name">
              <label for="codigo" class="u-label">Código</label>
              <input 
                v-model.number="form.codigo"
                type="number" 
                placeholder="Introduzca el código del producto" 
                id="codigo" 
                name="codigo" 
                class="u-border-black u-grey-15 u-input u-input-rectangle"
                :class="{ error: errors.codigo }"
                required
                :disabled="loading"
              >
              <span v-if="errors.codigo" class="error-text">{{ errors.codigo }}</span>
            </div>

            <!-- Nombre -->
            <div class="u-form-email u-form-group">
              <label for="nombre" class="u-label">Nombre</label>
              <input 
                v-model="form.nombre"
                type="text" 
                placeholder="Introduzca el nombre del producto" 
                id="nombre" 
                name="nombre" 
                class="u-border-black u-grey-15 u-input u-input-rectangle"
                :class="{ error: errors.nombre }"
                required
                :disabled="loading"
              >
              <span v-if="errors.nombre" class="error-text">{{ errors.nombre }}</span>
            </div>

            <!-- Tipo Producto -->
            <div class="u-form-group u-form-group-3">
              <label for="tipoProducto" class="u-label">Tipo Producto</label>
              <select 
                v-model="form.idTipoProducto"
                id="tipoProducto" 
                name="tipoProducto" 
                class="u-border-black u-grey-15 u-input u-input-rectangle"
                :class="{ error: errors.idTipoProducto }"
                required
                :disabled="loading"
              >
                <option value="">Seleccione tipo de producto</option>
                <option 
                  v-for="tipo in tiposProducto" 
                  :key="tipo.idTipoProducto" 
                  :value="tipo.idTipoProducto"
                >
                  {{ tipo.nombre }}
                </option>
              </select>
              <span v-if="errors.idTipoProducto" class="error-text">{{ errors.idTipoProducto }}</span>
            </div>

            <!-- Marca -->
            <div class="u-form-group u-form-group-4">
              <label for="marca" class="u-label">Marca</label>
              <input 
                v-model="form.marca"
                type="text" 
                placeholder="Introduzca marca del producto" 
                id="marca" 
                name="marca" 
                class="u-border-black u-grey-15 u-input u-input-rectangle"
                :class="{ error: errors.marca }"
                :disabled="loading"
              >
              <span v-if="errors.marca" class="error-text">{{ errors.marca }}</span>
            </div>

            <!-- Descripción -->
            <div class="u-form-group u-form-message">
              <label for="descripcion" class="u-label">Descripción</label>
              <textarea 
                v-model="form.descripcion"
                placeholder="Introduzca una breve descripción" 
                rows="4" 
                cols="50" 
                id="descripcion" 
                name="descripcion" 
                class="u-border-black u-grey-15 u-input u-input-rectangle"
                :class="{ error: errors.descripcion }"
                required
                :disabled="loading"
              ></textarea>
              <span v-if="errors.descripcion" class="error-text">{{ errors.descripcion }}</span>
            </div>

            <!-- Precio neto -->
            <div class="u-form-group u-form-partition-factor-3 u-form-group-6">
              <label for="precioNeto" class="u-label">Precio neto</label>
              <input 
                v-model.number="form.precioNeto"
                type="number" 
                step="0.01"
                placeholder="Introduzca precio neto" 
                id="precioNeto" 
                name="precioNeto" 
                class="u-border-black u-grey-15 u-input u-input-rectangle"
                :class="{ error: errors.precioNeto }"
                required
                :disabled="loading"
              >
              <span v-if="errors.precioNeto" class="error-text">{{ errors.precioNeto }}</span>
            </div>

            <!-- Moneda -->
            <div class="u-form-group u-form-partition-factor-3 u-form-select u-form-group-7">
              <label for="moneda" class="u-label">Moneda</label>
              <div class="u-form-select-wrapper">
                <select 
                  v-model="form.idMoneda"
                  id="moneda" 
                  name="moneda" 
                  class="u-border-black u-grey-15 u-input u-input-rectangle"
                  :class="{ error: errors.idMoneda }"
                  required
                  :disabled="loading"
                >
                  <option value="">Seleccionar moneda</option>
                  <option value="1">Peso Argentino</option>
                  <option value="2">Dólar</option>
                </select>
                <svg class="u-caret u-caret-svg" version="1.1" xmlns="http://www.w3.org/2000/svg" width="16px" height="16px" viewBox="0 0 16 16" style="fill:currentColor;">
                  <polygon class="st0" points="8,12 2,4 14,4"></polygon>
                </svg>
              </div>
              <span v-if="errors.idMoneda" class="error-text">{{ errors.idMoneda }}</span>
            </div>

            <!-- Stock/Cantidad -->
            <div class="u-form-group u-form-partition-factor-3 u-form-group-8">
              <label for="stock" class="u-label">Stock</label>
              <input 
                v-model.number="form.stock"
                type="number" 
                placeholder="Introduzca la cantidad en stock" 
                id="stock" 
                name="stock" 
                class="u-border-black u-grey-15 u-input u-input-rectangle"
                :class="{ error: errors.stock }"
                required
                :disabled="loading"
                min="0"
              >
              <span v-if="errors.stock" class="error-text">{{ errors.stock }}</span>
            </div>

            <!-- IVA -->
            <div class="u-form-group u-form-group-9">
              <label for="iva" class="u-label">IVA (%)</label>
              <input 
                v-model.number="form.iva"
                type="number" 
                step="0.01"
                placeholder="Introduzca el IVA" 
                id="iva" 
                name="iva" 
                class="u-border-black u-grey-15 u-input u-input-rectangle"
                :class="{ error: errors.iva }"
                :disabled="loading"
                min="0"
                max="100"
              >
              <span v-if="errors.iva" class="error-text">{{ errors.iva }}</span>
            </div>

            <!-- Ganancia -->
            <div class="u-form-group u-form-group-10">
              <label for="ganancia" class="u-label">Ganancia (%)</label>
              <input 
                v-model.number="form.ganancia"
                type="number" 
                step="0.01"
                placeholder="Introduzca el porcentaje de ganancia" 
                id="ganancia" 
                name="ganancia" 
                class="u-border-black u-grey-15 u-input u-input-rectangle"
                :class="{ error: errors.ganancia }"
                :disabled="loading"
                min="0"
              >
              <span v-if="errors.ganancia" class="error-text">{{ errors.ganancia }}</span>
            </div>

            <!-- Botón submit -->
            <div class="u-align-right u-form-group u-form-submit">
              <button 
                type="submit"
                class="u-border-2 u-border-black u-btn u-btn-submit u-button-style u-custom-color-1 u-btn-1"
                :disabled="loading || !isFormValid"
              >
                {{ loading ? 'Guardando...' : (isEditing ? 'Actualizar' : 'Agregar') }} Repuesto
              </button>
            </div>

            <!-- Mensajes -->
            <div v-if="message" class="form-message" :class="messageType">
              {{ message }}
            </div>

            <!-- Botones adicionales -->
            <div class="form-actions">
              <router-link 
                to="/productos" 
                class="u-btn u-btn-secondary"
              >
                ← Cancelar
              </router-link>
              <button 
                v-if="isEditing" 
                type="button"
                @click="deleteProduct"
                class="u-btn u-btn-danger"
                :disabled="loading"
              >
                🗑️ Eliminar
              </button>
            </div>

          </form>
        </div>
      </div>
    </section>

    <!-- Footer -->
    <footer class="u-black u-clearfix u-footer" id="footer">
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
import { useRouter, useRoute } from 'vue-router';
import { authService, productoService } from '@/services';
import type { 
  Usuario, 
  CreateProductoDto, 
  UpdateProductoDto, 
  TipoProducto 
} from '@/services';

const router = useRouter();
const route = useRoute();

// Estado reactivo
const currentUser = ref<Usuario | null>(null);
const loading = ref(false);
const message = ref('');
const messageType = ref<'success' | 'error'>('success');
const imagePreview = ref('');

// Datos del formulario
const form = ref<CreateProductoDto>({
  codigo: 0,
  nombre: '',
  descripcion: '',
  marca: '',
  precioNeto: 0,
  iva: 21,
  ganancia: 0,
  stock: 0,
  idTipoProducto: 0,
  idMoneda: 1
});

// Datos adicionales
const tiposProducto = ref<TipoProducto[]>([]);
const errors = ref<Record<string, string>>({});

// Computed properties
const isEditing = computed(() => !!route.params.id);
const productId = computed(() => parseInt(route.params.id as string));

const isFormValid = computed(() => {
  return form.value.codigo > 0 &&
         form.value.nombre.trim() !== '' &&
         form.value.descripcion.trim() !== '' &&
         form.value.precioNeto > 0 &&
         form.value.stock >= 0 &&
         form.value.idTipoProducto > 0 &&
         form.value.idMoneda > 0;
});

// Métodos
const loadInitialData = async () => {
  try {
    // Cargar tipos de producto (simulados por ahora)
    tiposProducto.value = [
      { idTipoProducto: 1, nombre: 'Repuesto Original', descripcion: 'Repuestos originales de fábrica' },
      { idTipoProducto: 2, nombre: 'Repuesto Alternativo', descripcion: 'Repuestos alternativos' },
      { idTipoProducto: 3, nombre: 'Herramienta', descripcion: 'Herramientas y equipos' },
      { idTipoProducto: 4, nombre: 'Lubricante', descripcion: 'Aceites y lubricantes' },
      { idTipoProducto: 5, nombre: 'Accesorio', descripcion: 'Accesorios varios' }
    ];

    // Si estamos editando, cargar datos del producto
    if (isEditing.value && productId.value) {
      await loadProduct();
    }
  } catch (error) {
    console.error('Error cargando datos iniciales:', error);
    showMessage('Error al cargar los datos del formulario', 'error');
  }
};

const loadProduct = async () => {
  if (!productId.value) return;

  loading.value = true;
  try {
    const producto = await productoService.getById(productId.value);
    
    // Mapear datos del producto al formulario
    form.value = {
      codigo: producto.codigo,
      nombre: producto.nombre,
      descripcion: producto.descripcion,
      marca: producto.marca || '',
      precioNeto: producto.precioNeto,
      iva: producto.iva,
      ganancia: producto.ganancia,
      stock: producto.stock,
      idTipoProducto: producto.tipoProducto?.idTipoProducto || 0,
      idMoneda: producto.moneda?.idMoneda || 1
    };

  } catch (error) {
    console.error('Error cargando producto:', error);
    showMessage('Error al cargar los datos del producto', 'error');
    router.push('/productos');
  } finally {
    loading.value = false;
  }
};

const validateForm = (): boolean => {
  errors.value = {};

  if (form.value.codigo <= 0) {
    errors.value.codigo = 'El código debe ser mayor a 0';
  }

  if (!form.value.nombre.trim()) {
    errors.value.nombre = 'El nombre es requerido';
  }

  if (!form.value.descripcion.trim()) {
    errors.value.descripcion = 'La descripción es requerida';
  }

  if (form.value.precioNeto <= 0) {
    errors.value.precioNeto = 'El precio debe ser mayor a 0';
  }

  if (form.value.stock < 0) {
    errors.value.stock = 'El stock no puede ser negativo';
  }

  if (form.value.idTipoProducto <= 0) {
    errors.value.idTipoProducto = 'Debe seleccionar un tipo de producto';
  }

  if (form.value.idMoneda <= 0) {
    errors.value.idMoneda = 'Debe seleccionar una moneda';
  }

  if (form.value.iva < 0 || form.value.iva > 100) {
    errors.value.iva = 'El IVA debe estar entre 0 y 100';
  }

  if (form.value.ganancia < 0) {
    errors.value.ganancia = 'La ganancia no puede ser negativa';
  }

  return Object.keys(errors.value).length === 0;
};

const handleSubmit = async () => {
  if (!validateForm() || loading.value) return;

  loading.value = true;
  message.value = '';

  try {
    if (isEditing.value) {
      const updateData: UpdateProductoDto = { ...form.value };
      await productoService.update(productId.value, updateData);
      showMessage('Producto actualizado correctamente', 'success');
    } else {
      await productoService.create(form.value);
      showMessage('Producto creado correctamente', 'success');
    }

    // Redirigir después de un breve delay
    setTimeout(() => {
      router.push('/productos');
    }, 1500);

  } catch (error: any) {
    console.error('Error al guardar producto:', error);
    if (error.response?.data?.message) {
      showMessage(error.response.data.message, 'error');
    } else {
      showMessage('Error al guardar el producto', 'error');
    }
  } finally {
    loading.value = false;
  }
};

const deleteProduct = async () => {
  if (!isEditing.value || !productId.value) return;

  const confirmDelete = confirm(`¿Está seguro que desea eliminar el producto "${form.value.nombre}"?`);
  if (!confirmDelete) return;

  loading.value = true;
  try {
    await productoService.delete(productId.value);
    showMessage('Producto eliminado correctamente', 'success');
    
    setTimeout(() => {
      router.push('/productos');
    }, 1500);

  } catch (error) {
    console.error('Error al eliminar producto:', error);
    showMessage('Error al eliminar el producto', 'error');
  } finally {
    loading.value = false;
  }
};

const handleImageChange = (event: Event) => {
  const target = event.target as HTMLInputElement;
  const file = target.files?.[0];
  
  if (file) {
    // Validar que sea una imagen
    if (!file.type.startsWith('image/')) {
      alert('Seleccione un archivo de imagen válido');
      target.value = '';
      return;
    }

    // Mostrar preview
    const reader = new FileReader();
    reader.onload = (e) => {
      imagePreview.value = e.target?.result as string;
    };
    reader.readAsDataURL(file);
  }
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
  loadInitialData();
});
</script>

<style>
/* Importar CSS originales */
@import '/css/nicepage.css';
@import '/css/Agregar-Repuesto.css';
@import '/css/Modificar-Repuesto.css';
</style>