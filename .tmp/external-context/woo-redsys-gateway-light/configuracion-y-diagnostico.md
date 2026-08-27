---
source: Redsys Official Docs / WordPress Plugin Page / Context7 API
library: Redsys
package: woo-redsys-gateway-light
topic: Configuración y Diagnóstico SIS0042
fetched: 2026-06-08T10:00:00Z
official_docs: https://pagosonline.redsys.es/
---

# Configuración del Plugin y Diagnóstico del Error SIS0042

## 4. Configuración Necesaria en el Plugin

### 4.1. Campos Obligatorios en WooCommerce → Ajustes → Redsys/Servired

| Campo | Descripción | Formato | Ejemplo (Test) |
|---|---|---|---|
| **Código de Comercio (FUC)** | Código FUC asignado por el banco | 9 dígitos numéricos | `999008881` |
| **Número de Terminal** | Terminal asignado por el banco | Numérico | `1` |
| **Clave SHA-256 (Producción)** | Clave secreta del terminal en producción | Cadena alfanumérica (generalmente 24-32 caracteres) | - |
| **Clave SHA-256 (Test)** | Clave secreta del terminal en pruebas | Cadena alfanumérica | `sq7HjrUOBfKmC576ILgskD5srU870gJ7` |

### 4.2. Configuración CORRECTA de la Clave SHA-256

**IMPORTANTE**: La clave SHA-256 no la genera el plugin; te la proporciona tu banco/entidad a través del Portal de Administración de Redsys.

**Para obtener la clave:**
1. Acceder al Portal de Administración del TPV Virtual de Redsys
2. Ir a "Consulta datos del Comercio"
3. Localizar el terminal en el listado
4. Hacer clic en el icono de la **llave** → muestra la clave del terminal
5. O bien, dentro de la configuración del terminal, pulsar "Ver clave de firma"

**La clave se compone de:**
- Hasta 16 caracteres para la derivación HMAC-SHA256 (si es más larga, se truncan a 16; si es más corta, se rellena con ceros)
- La clave completa se usa para HMAC-SHA256

**Para el entorno de pruebas (test):**
- La clave de test por defecto es: `sq7HjrUOBfKmC576ILgskD5srU870gJ7`
- Pero **tu banco puede haber establecido una clave diferente** para pruebas
- **No asumas que es la clave por defecto** — verifícala en el Portal de Administración

### 4.3. Modo Test vs. Modo Producción

El plugin tiene un selector de modo (test/producción) en la configuración:

- **Modo Test**: 
  - Usa la URL `https://sis-t.redsys.es:25443/sis/realizarPago`
  - Debe usar la CLAVE DE TEST
  - El FUC de prueba suele ser `999008881` y terminal `1`
  - Las tarjetas de prueba de Redsys tienen números específicos

- **Modo Producción**:
  - Usa la URL `https://sis.redsys.es/sis/realizarPago`
  - Debe usar la CLAVE DE PRODUCCIÓN (NUNCA la de test)
  - FUC y terminal reales asignados por el banco

**Error común**: Tener puesta la clave de producción en el campo de "Clave SHA-256" pero NO tener configurada la "Clave SHA-256 (Test)" o viceversa.

## 5. Diagnóstico Paso a Paso

### Paso 1: Verificar la versión del plugin

- **Versión actual (recomendada)**: 7.0.1
- Si estás en una versión ANTERIOR a 7.0.1, actualiza inmediatamente
- La v7.0.1 corrige: *"Test SHA-256 secret was being overwritten with the production secret in successful_request()"*

### Paso 2: Verificar la clave SHA-256

1. Entra al Portal de Administración de Redsys
2. Localiza la clave del terminal (icono llave)
3. Copia EXACTAMENTE la clave (sin espacios, sin saltos de línea)
4. Pégala en el campo correspondiente del plugin (Test o Producción)
5. **Verifica que las claves de test y producción son DIFERENTES y están en sus campos correctos**

### Paso 3: Comprobar el número de pedido

1. Activa el modo debug/logging en WooCommerce
2. Realiza una compra de prueba
3. Revisa los logs (`wp-content/debug.log` o `/wp-content/uploads/wc-logs/`)
4. Busca el valor de `Ds_Merchant_Order` generado
5. Verifica que:
   - Tiene entre 4 y 12 caracteres
   - Los primeros 4 son numéricos
   - Solo contiene caracteres ASCII válidos (0-9, A-Z, a-z)

### Paso 4: Verificar el formato del importe

1. Revisa en los logs qué valor de `Ds_Merchant_Amount` se está enviando
2. Para 25,00 € debe ser `2500`
3. Si ves decimales o formato incorrecto, hay un problema de configuración

### Paso 5: Verificar moneda y tipo de transacción

| Parámetro | Valor correcto |
|---|---|
| `Ds_Merchant_Currency` | `978` (EUR) |
| `Ds_Merchant_TransactionType` | `0` (Autorización) |

### Paso 6: Probar con la clave de test por defecto

Configura el plugin en **modo test** con:
- FUC: `999008881`
- Terminal: `1`
- Clave SHA-256 Test: `sq7HjrUOBfKmC576ILgskD5srU870gJ7`
- Activar modo test en el selector

Si con estos datos funciona pero con los datos de tu banco no, el problema está en la configuración de tu banco o en la clave proporcionada.

### Paso 7: Verificar la versión de firma (HMAC_SHA256_V1)

El plugin usa `HMAC_SHA256_V1`. Si tu banco ha migrado a `HMAC_SHA512_V2`, la versión Lite NO funcionará. Necesitarías la versión Premium o contactar con el banco para que NO active HMAC_SHA512_V2.

Confirmación visual: El parámetro `Ds_SignatureVersion` debe ser `HMAC_SHA256_V1`.

### Paso 8: Probar con otro navegador/entorno

A veces, problemas de JavaScript, cookies, o extensiones del navegador pueden interferir con la redirección POST a Redsys. Prueba:
- Otro navegador
- Modo incógnito
- Sin extensiones

### Paso 9: Revisar logs del servidor

- Revisa `error_log` de PHP
- Revisa `wp-content/debug.log`
- Activa el debug de WooCommerce: WooCommerce → Status → Logs → Busca "redsys"

## 6. Soluciones específicas

### Solución A: Clave mal configurada
- Obtén la clave correcta del Portal de Administración de Redsys
- Asegúrate de NO tener espacios al inicio/final
- Verifica que la clave de test y producción están en sus respectivos campos

### Solución B: Bug de versión antigua
- **Actualiza el plugin a v7.0.1** (o superior)
- La corrección específica para el bug que sobreescribía la clave de test está en v7.0.1

### Solución C: Problema de formato de pedido
- Si el número de pedido generado por WooCommerce no cumple el formato (primeros 4 numéricos), puede causar SIS0042
- El plugin intenta formatearlo correctamente, pero si usas plugins que modifican los números de pedido, pueden causar conflictos

### Solución D: Versión de firma incompatible
- Verifica con tu banco qué versión de firma tienes configurada
- Si es HMAC_SHA512_V2, necesitas contactar con el banco para que te activen HMAC_SHA256_V1 o adquirir la versión Premium del plugin

### Solución E: Migración de entorno
- Si estabas en pruebas y pasaste a producción, verifica que CAMBIASTE la clave de test por la de producción
- Estos dos entornos usan claves DIFERENTES

## 7. Referencias Útiles

- **Plugin oficial**: https://es.wordpress.org/plugins/woo-redsys-gateway-light/
- **Foro de soporte**: https://wordpress.org/support/plugin/woo-redsys-gateway-light/
- **Documentación Redsys (desarrolladores)**: https://pagosonline.redsys.es/
- **Códigos de respuesta Redsys**: https://github.com/ssheduardo/sermepa/blob/master/doc/Códigos%20de%20Respuesta.md
- **Portal de Administración Redsys**: Acceder a través de tu banco/entidad
- **Autor del plugin (Jose Conti)**: https://plugins.joseconti.com/
- **Versión Premium**: https://plugins.joseconti.com/product/plugin-woocommerce-redsys-gateway/
- **Soporte Redsys**: soportevirtual@redsys.es | Tel: 91 728 23 23
