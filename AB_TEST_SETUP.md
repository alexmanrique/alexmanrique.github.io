# Configuración del Test A/B con Google Analytics

Este documento explica cómo funciona el test A/B para comparar el formulario de suscripción antiguo vs. el nuevo usando Google Analytics (sin necesidad de servicios de pago como PostHog).

## Cómo Funciona

El test A/B funciona completamente en el frontend usando:

1. **Randomización simple**: Cada usuario recibe una variante aleatoria (50/50) al cargar la página
2. **Persistencia**: La variante se guarda en `localStorage` para que el mismo usuario siempre vea la misma versión
3. **Google Analytics**: Todos los eventos se trackean usando Google Analytics (gtag) que ya está instalado

## No Requiere Configuración Adicional

El código está listo para funcionar. Solo necesitas:

1. ✅ Tener Google Analytics configurado (ya lo tienes: `G-79GL8C39KM`)
2. ✅ Los eventos se trackean automáticamente

## Eventos y Métricas que se trackean automáticamente

El código trackea los siguientes eventos en Google Analytics:

### Métricas Principales (Conversión)

- **`newsletter_form_submitted`**: Se dispara cuando un usuario envía el formulario (métrica principal)
  - Propiedades:
    - `form_id` (old o new)
    - `variant` (control o test)
    - `conversion: true`
    - `time_to_convert_seconds`: Tiempo desde que vio el formulario hasta que se suscribió
    - `total_time_on_page_seconds`: Tiempo total desde que cargó la página hasta que se suscribió

### Métricas de Engagement

- **`newsletter_form_variant_shown`**: Se dispara cuando se muestra una variante

  - Propiedades: `variant` (control o test)
  - Uso: Contar impresiones del formulario

- **`newsletter_form_viewed`**: Se dispara cuando el formulario se hace visible

  - Propiedades:
    - `variant` (control o test)
    - `timestamp`: Timestamp cuando se vio el formulario
  - Uso: Medir cuántos usuarios realmente ven el formulario

- **`newsletter_input_focused`**: Se dispara cuando el usuario hace focus en el campo de email

  - Propiedades: `variant` (control o test)
  - Uso: Medir interés/interacción con el formulario

- **`newsletter_button_clicked`**: Se dispara cuando el usuario hace clic en el botón "Subscribe"
  - Propiedades: `variant` (control o test)
  - Uso: Medir interés, incluso si no completan el formulario

### Métricas de Tiempo en la Landing Page

- **`landing_page_time`**: Se dispara cuando el usuario sale de la página, cambia de pestaña o envía el formulario

  - Propiedades:
    - `variant` (control o test)
    - `time_on_page_seconds`: Tiempo total en segundos que el usuario pasó en la página
    - `event_type`: "page_exit", "tab_switch" o "form_submit"
  - Uso: Medir engagement general y tiempo promedio en la página

- **`landing_page_milestone`**: Se dispara cuando el usuario alcanza hitos de tiempo (10, 30, 60 segundos)
  - Propiedades:
    - `variant` (control o test)
    - `milestone_seconds`: 10, 30 o 60
    - `time_on_page_seconds`: Tiempo exacto en ese momento
  - Uso: Medir retención y engagement a lo largo del tiempo

## Ver los Resultados en Google Analytics

### 1. Acceder a Google Analytics

1. Ve a [Google Analytics](https://analytics.google.com)
2. Selecciona tu propiedad (ID: G-79GL8C39KM)
3. Ve a **Reports** → **Events**

### 2. Ver Eventos del Test A/B

En la sección de eventos, verás todos los eventos trackeados. Puedes filtrar por:

- **Event name**: Busca los nombres de eventos mencionados arriba
- **Event parameters**: Filtra por `variant` para ver solo "control" o "test"

### 3. Crear Reportes Personalizados

Para analizar mejor los resultados, puedes crear reportes personalizados:

1. Ve a **Explore** → **Free Form**
2. Añade dimensiones:
   - `Event name`
   - `Event parameter: variant`
3. Añade métricas:
   - `Event count` (para contar eventos)
   - `Total users` (para contar usuarios únicos)

### 4. Métricas Derivadas que puedes calcular

**Tasa de Conversión:**

```
newsletter_form_submitted / newsletter_form_variant_shown × 100
```

**Tasa de Engagement:**

```
newsletter_input_focused / newsletter_form_viewed × 100
```

**Tasa de Clics:**

```
newsletter_button_clicked / newsletter_form_viewed × 100
```

**Tasa de Retención por Hitos:**

```
landing_page_milestone (milestone_seconds=30) / newsletter_form_variant_shown × 100
```

**Tiempo Promedio hasta Conversión:**

```
Promedio de time_to_convert_seconds de newsletter_form_submitted
```

## Interpretar los Resultados

### Qué buscar en los resultados:

✅ **Mayor tasa de conversión**: Más usuarios se suscriben con una variante
✅ **Mayor engagement**: Más usuarios interactúan con el formulario (focus, clics)
✅ **Menor tiempo hasta conversión**: El diseño es más claro/atractivo
✅ **Mayor tiempo en página**: El diseño mantiene el interés del usuario
✅ **Mejor retención**: Más usuarios alcanzan los hitos de tiempo (10s, 30s, 60s)

### Ejemplo de análisis:

```
Variante Control:
- Tasa de conversión: 3.2%
- Tiempo promedio en página: 45 segundos
- 60% llega a 30 segundos
- Tiempo hasta conversión: 15 segundos

Variante Test:
- Tasa de conversión: 4.8% (+50% mejora)
- Tiempo promedio en página: 65 segundos
- 75% llega a 30 segundos
- Tiempo hasta conversión: 12 segundos
```

Esto indicaría que la variante Test:

- ✅ Convierte mejor (mayor tasa de conversión)
- ✅ Mantiene a los usuarios más tiempo (mayor engagement)
- ✅ Convierte más rápido (diseño más efectivo)
- ✅ Tiene mejor retención (más usuarios exploran la página)

## Estructura del Código

### Versiones del Formulario

- **Versión A (Control)**: Formulario antiguo con diseño separado

  - ID: `newsletter-form-old`
  - Clase: `newsletter-form-old`
  - Estilos en CSS: `.newsletter-form-old`

- **Versión B (Test)**: Formulario nuevo con diseño unificado dorado
  - ID: `newsletter-form-new`
  - Clase: `newsletter-form`
  - Estilos en CSS: `.newsletter-form` y `#mc_embed_signup_scroll`

### Configuración del Test

El código está en `index.html` alrededor de la línea 410:

```javascript
const VARIANT_CONTROL = "control"; // Old design
const VARIANT_TEST = "test"; // New design
const STORAGE_KEY = "newsletter_form_variant"; // localStorage key
```

### Cómo Funciona la Randomización

1. Al cargar la página, se verifica si el usuario ya tiene una variante asignada en `localStorage`
2. Si no tiene una, se asigna aleatoriamente (50/50)
3. La variante se guarda en `localStorage` para mantener consistencia
4. Todos los eventos se trackean con Google Analytics usando `gtag`

## Notas Importantes

- ✅ **Gratis**: No requiere servicios de pago
- ✅ **Persistente**: Los usuarios siempre ven la misma variante (gracias a localStorage)
- ✅ **Automático**: Todo funciona automáticamente sin configuración adicional
- ✅ **Mismo endpoint**: Ambas versiones usan el mismo endpoint de Mailchimp
- ✅ **Transparente**: El test es completamente transparente para el usuario

## Troubleshooting

### El formulario no aparece

- Verifica que el JavaScript esté cargando correctamente
- Revisa la consola del navegador para errores
- Asegúrate de que ambos formularios existan en el HTML

### Los eventos no se trackean

- Verifica que Google Analytics esté cargando correctamente
- Revisa la consola del navegador
- Verifica que el ID de Google Analytics (`G-79GL8C39KM`) sea correcto

### Solo se muestra una versión

- Limpia el `localStorage` del navegador para resetear la variante asignada
- Abre la consola y ejecuta: `localStorage.removeItem('newsletter_form_variant')`
- Recarga la página para recibir una nueva variante aleatoria

### Resetear el test para un usuario específico

Si quieres resetear la variante asignada a un usuario:

1. Abre la consola del navegador (F12)
2. Ejecuta: `localStorage.removeItem('newsletter_form_variant')`
3. Recarga la página
