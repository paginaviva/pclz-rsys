---
source: Redsys Official Docs / Context7 API / GitHub (ssheduardo/sermepa)
library: Redsys
package: woo-redsys-gateway-light
topic: SIS0042 Error - Significado y causas
fetched: 2026-06-08T10:00:00Z
official_docs: https://pagosonline.redsys.es/
---

# Error SIS0042 de Redsys: Significado y Análisis Completo

## 1. Significado del Error SIS0042

Según la documentación oficial de Redsys (códigos de respuesta), el error **SIS0042** se define como:

| Código Error | Código SIS | Descripción |
|---|---|---|
| 9042 | **SIS0042** | **"Error en el cálculo de la firma"** |

Este error es **IDÉNTICO** al SIS0041 (`Error en el cálculo de la firma`). Ambos códigos (0041 y 0042) indican exactamente el mismo problema: la firma criptográfica HMAC calculada por el plugin NO coincide con la que Redsys espera.

### ¿En qué capa se produce?

El error SIS0042 se produce en la **capa de comunicación entre el plugin y Redsys**, específicamente durante la validación de la firma de la petición:

1. **Plugin (WooCommerce)**: Genera los parámetros de la transacción y calcula la firma HMAC-SHA256 usando la clave secreta del comercio
2. **Envía** la petición firmada a `https://sis.redsys.es/sis/realizarPago`
3. **Redsys (SIS)**: Recibe la petición, recalcula la firma con su copia de la clave secreta y la compara
4. **SIS0042**: Las firmas NO coinciden → Redsys rechaza la petición INMEDIATAMENTE

**El error ocurre en el servidor de Redsys**, antes de cualquier procesamiento de pago. Por eso ni siquiera se muestra el formulario de tarjeta.

### ¿Es error de configuración del comercio o del plugin?

**Es un error de configuración/desajuste entre ambos.** Puede originarse en:

- **El plugin**: Clave incorrecta, mal formato de parámetros, versión de firma incorrecta
- **El banco/comercio**: La clave configurada en el panel de Redsys no coincide con la del plugin
- **Ambos**: Diferencia entre clave de test y producción

## 2. Causas Comunes del SIS0042

### 2.1. ❌ CLAVE SHA-256 INCORRECTA (CAUSA #1 - MÁS FRECUENTE)

- La clave configurada en el plugin NO es la misma que la configurada en el Portal de Administración de Redsys
- La clave de test (entorno de pruebas) es DIFERENTE a la clave de producción
- La clave se ha copiado mal (espacios, caracteres extra, mayúsculas/minúsculas)

**Clave de prueba estándar de Redsys (solo para desarrollo):**
`sq7HjrUOBfKmC576ILgskD5srU870gJ7`

### 2.2. ❌ VERSIÓN DE FIRMA INCORRECTA

Redsys soporta dos versiones de firma:

| Versión | Algoritmo | Derivación de clave |
|---|---|---|
| `HMAC_SHA256_V1` (antigua) | HMAC-SHA256 | 3DES |
| `HMAC_SHA512_V2` (nueva) | HMAC-SHA512 | AES-CBC |

El plugin "WooCommerce Redsys Gateway Light" usa **HMAC_SHA256_V1** por defecto. Si tu banco ha migrado a HMAC_SHA512_V2, el plugin NO funcionará correctamente en su versión gratuita (Lite). La versión Premium sí soporta ambas.

### 2.3. ❌ FORMATO INCORRECTO DEL NÚMERO DE PEDIDO (Ds_Merchant_Order)

El número de pedido es clave en el cálculo de la firma (se usa para derivar la clave específica de la operación). Requisitos:

- **Longitud**: 4 a 12 caracteres
- **Primeros 4 dígitos**: deben ser NUMÉRICOS
- **Caracteres permitidos**: Solo ASCII (0-9, A-Z, a-z)

El número de pedido se usa para "diversificar" la clave (derivar una clave específica para cada operación). Si el formato es incorrecto, la firma no coincidirá.

### 2.4. ❌ FORMATO DEL IMPORTE INCORRECTO

El importe debe enviarse en **céntimos** (sin decimales):
- 25,00 € → `2500`
- 100,50 € → `10050`
- 9,99 € → `999`

Si se envía con decimales o en formato incorrecto, el paquete completo (incluyendo la firma) no coincidirá.

### 2.5. ❌ CÓDIGO DE MONEDA INCORRECTO

Usar el código ISO 4217 numérico:
- EUR → `978`
- USD → `840`
- GBP → `826`

Nunca usar el símbolo (`€`) ni el código ISO alfabético (`EUR`).

### 2.6. ❌ PROBLEMAS CON FUC O TERMINAL

- **FUC (Código de comercio)**: Debe tener 9 dígitos
- **Terminal**: Normalmente 1-3 dígitos (en pruebas suele ser `1`)
- Si estos datos no coinciden con la configuración del banco, la firma se calcula sobre parámetros incorrectos

### 2.7. ❌ BUG DEL PLUGIN v6.x (CORREGIDO EN v7.0.1)

Según el changelog oficial del plugin (versión 7.0.1):
> "Fix: Test SHA-256 secret was being overwritten with the production secret in successful_request(), causing signature validation failures in test mode."

**Esto significa que en versiones anteriores a la 7.0.1, si tenías configurada una clave de test y otra de producción, la clave de test era SOBREESCRITA por la de producción durante el proceso de validación, causando SIS0042 en modo test.**

### 2.8. ❌ CERTIFICADOS SSL / SNI

El plugin es compatible con certificados SNI (como Let's Encrypt). Si el certificado SSL del sitio no es válido o está mal configurado, la comunicación con Redsys puede fallar, aunque esto suele dar otros errores (SIS0007, errores de conexión) más que SIS0042.

### 2.9. ❌ PSD2 / SCA (AUTENTICACIÓN REFORZADA)

El plugin es compatible con PSD2 desde la versión 3.0.0. Si el banco requiere autenticación reforzada y los parámetros EMV3DS no se envían correctamente, puede dar errores de firma porque los parámetros firmados no coinciden con los esperados.

## 3. ¿Por qué no se muestra el formulario de tarjeta?

La versión **Lite** del plugin usa **redirección** (no modal, no InSite). El flujo es:

1. Usuario hace clic en "Pagar" en WooCommerce
2. El plugin genera un formulario HTML con parámetros ocultos (DS_MERCHANT_PARAMETERS, DS_SIGNATURE, DS_SIGNATURE_VERSION)
3. El navegador del usuario se redirige (POST automático) a `https://sis.redsys.es/sis/realizarPago`
4. **Redsys recibe la petición, valida la firma, y SI es correcta → muestra el formulario de tarjeta**
5. Si la firma NO es correcta (SIS0042) → Redsys devuelve error inmediatamente

**El error SIS0042 es TAN TEMPRANO en el flujo que Redsys ni siquiera llega a renderizar el formulario de pago.** Redsys rechaza la petición antes de cualquier procesamiento porque detecta que los datos han sido manipulados o la firma no es válida.

Esto explica por qué el usuario ve el mensaje de error sin haber introducido nunca los datos de tarjeta: **Redsys ni siquiera llegó a mostrar el formulario porque la petición fue rechazada en la validación inicial de firma.**
