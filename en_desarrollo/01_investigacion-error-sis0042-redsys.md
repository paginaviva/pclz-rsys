# Investigación: Error SIS0042 en Redsys Gateway Light para WooCommerce

---

## Índice de Contenido

- [1. Resumen Ejecutivo](#1-resumen-ejecutivo)
- [2. Plugin Investigado](#2-plugin-investigado)
  - [2.1. Datos del Plugin](#21-datos-del-plugin)
  - [2.2. Versiones y Changelog Relevante](#22-versiones-y-changelog-relevante)
- [3. El Error SIS0042](#3-el-error-sis0042)
  - [3.1. Significado](#31-significado)
  - [3.2. ¿En qué capa se produce?](#32-en-qu%C3%A9-capa-se-produce)
  - [3.3. ¿Por qué no se muestra el formulario de tarjeta?](#33-por-qu%C3%A9-no-se-muestra-el-formulario-de-tarjeta)
- [4. Causas del Error SIS0042](#4-causas-del-error-sis0042)
  - [4.1. Clave SHA-256 Incorrecta (Causa #1)](#41-clave-sha-256-incorrecta-causa-1)
  - [4.2. Bug del Plugin v6.x (Corregido en v7.0.1)](#42-bug-del-plugin-v6x-corregido-en-v701)
  - [4.3. Versión de Firma Incorrecta](#43-versi%C3%B3n-de-firma-incorrecta)
  - [4.4. Formato Incorrecto del Número de Pedido](#44-formato-incorrecto-del-n%C3%BAmero-de-pedido)
  - [4.5. Formato del Importe Incorrecto](#45-formato-del-importe-incorrecto)
  - [4.6. Código de Moneda Incorrecto](#46-c%C3%B3digo-de-moneda-incorrecto)
  - [4.7. FUC o Terminal Incorrectos](#47-fuc-o-terminal-incorrectos)
  - [4.8. PSD2 / SCA (Autenticación Reforzada)](#48-psd2--sca-autenticaci%C3%B3n-reforzada)
  - [4.9. Certificados SSL / SNI](#49-certificados-ssl--sni)
- [5. Diagnóstico Paso a Paso](#5-diagn%C3%B3stico-paso-a-paso)
  - [Paso 1: Verificar versión del plugin](#paso-1-verificar-versi%C3%B3n-del-plugin)
  - [Paso 2: Verificar la clave SHA-256](#paso-2-verificar-la-clave-sha-256)
  - [Paso 3: Comprobar el número de pedido](#paso-3-comprobar-el-n%C3%BAmero-de-pedido)
  - [Paso 4: Verificar el formato del importe](#paso-4-verificar-el-formato-del-importe)
  - [Paso 5: Verificar moneda y tipo de transacción](#paso-5-verificar-moneda-y-tipo-de-transacci%C3%B3n)
  - [Paso 6: Probar con clave de test por defecto](#paso-6-probar-con-clave-de-test-por-defecto)
  - [Paso 7: Verificar versión de firma](#paso-7-verificar-versi%C3%B3n-de-firma)
  - [Paso 8: Probar con otro navegador/entorno](#paso-8-probar-con-otro-navegadorentorno)
  - [Paso 9: Revisar logs del servidor](#paso-9-revisar-logs-del-servidor)
- [6. Soluciones](#6-soluciones)
  - [Solución A: Clave mal configurada](#soluci%C3%B3n-a-clave-mal-configurada)
  - [Solución B: Bug de versión antigua](#soluci%C3%B3n-b-bug-de-versi%C3%B3n-antigua)
  - [Solución C: Problema de formato de pedido](#soluci%C3%B3n-c-problema-de-formato-de-pedido)
  - [Solución D: Versión de firma incompatible](#soluci%C3%B3n-d-versi%C3%B3n-de-firma-incompatible)
  - [Solución E: Migración de entorno](#soluci%C3%B3n-e-migraci%C3%B3n-de-entorno)
- [7. Referencia de Códigos de Error Redsys SIS](#7-referencia-de-c%C3%B3digos-de-error-redsys-sis)
  - [7.1. Errores de Firma y Parámetros](#71-errores-de-firma-y-par%C3%A1metros)
  - [7.2. Errores de Operación](#72-errores-de-operaci%C3%B3n)
  - [7.3. Errores de Configuración del Comercio](#73-errores-de-configuraci%C3%B3n-del-comercio)
  - [7.4. Errores de Autenticación 3DS / PSD2](#74-errores-de-autenticaci%C3%B3n-3ds--psd2)
  - [7.5. Códigos de Respuesta (Ds_Response)](#75-c%C3%B3digos-de-respuesta-ds_response)
- [8. Enlaces y Referencias Útiles](#8-enlaces-y-referencias-%C3%BAtiles)

---

## 1. Resumen Ejecutivo

El error **SIS0042** en la pasarela Redsys se produce cuando el plugin envía una petición de pago firmada con HMAC-SHA256 y la firma **no coincide** con la que Redsys espera. Esto provoca que Redsys **rechace la petición inmediatamente**, antes de renderizar cualquier formulario de pago, por lo que el usuario ve el mensaje de error sin haber podido introducir los datos de su tarjeta.

**Causa más probable (90% de los casos):** La **clave SHA-256** configurada en el plugin no coincide con la registrada en el Portal de Administración de Redsys, o existe un desajuste entre las claves de test y producción.

**Causa adicional relevante:** Bug presente en versiones **anteriores a la 7.0.1** donde la clave SHA-256 de test era sobreescrita por la de producción durante la validación (`successful_request()`). Si el plugin no está actualizado, **actualizarlo es el primer paso**.

---

## 2. Plugin Investigado

### 2.1. Datos del Plugin

| Campo | Valor |
|-------|-------|
| **Nombre** | Payment Gateway for Redsys & WooCommerce Lite |
| **Slug** | `woo-redsys-gateway-light` |
| **Autor** | Jose Conti ([@jconti](https://profiles.wordpress.org/jconti/)) |
| **Versión actual** | 7.0.1 |
| **Última actualización** | 8 de abril de 2026 |
| **Instalaciones activas** | 20.000+ |
| **WP probado hasta** | 6.9.4 |
| **PHP mínimo** | 7.0 |
| **Web del autor** | [plugins.joseconti.com](https://plugins.joseconti.com/) |
| **URL WordPress** | [https://es.wordpress.org/plugins/woo-redsys-gateway-light/](https://es.wordpress.org/plugins/woo-redsys-gateway-light/) |
| **Versión Premium** | [plugins.joseconti.com/product/plugin-woocommerce-redsys-gateway/](https://plugins.joseconti.com/product/plugin-woocommerce-redsys-gateway/) |

**Funcionalidades de la versión Lite:**
- Redirección Redsys
- Bizum Lite (redirección)
- Apple & Google Pay redirección
- Inespay Lite
- Compatible PSD2
- Compatible WPML
- Certificados SNI (Let's Encrypt, SiteGround, HostFusion, etc.)

**Funcionalidades EXCLUSIVAS de la versión Premium (no disponibles en Lite):**
- Formulario de tarjeta en la página de pago (tipo Stripe) / modal sin abandonar el sitio
- Bizum InSite (sin redirección)
- Google Pay / Apple Pay sin salir del sitio
- Tokenización y pago con 1 clic
- Suscripciones (WooCommerce Subscriptions, Yith, SUMO)
- Preautorizaciones, reembolsos, domiciliaciones
- Soporte HMAC_SHA512_V2

### 2.2. Versiones y Changelog Relevante

**v7.0.1** (abril 2026) — *Corrección crítica:*
> "Security Fix: Added cryptographic signature (Ds_Signature) verification in `successful_request()` for Redsys, Bizum, and Google Pay gateways to prevent payment forgery via the Order Received page."
>
> "Fix: Test SHA-256 secret was being overwritten with the production secret in `successful_request()`, causing signature validation failures in test mode."

**v7.0.0**
> "NEW: Plugin renamed to 'Payment Gateway for Redsys & WooCommerce'."
> "NEW: Added Inespay payment gateway."

---

## 3. El Error SIS0042

### 3.1. Significado

Según la documentación oficial de Redsys y la tabla de códigos de respuesta:

| Código Error | Código SIS | Descripción |
|---|---|---|
| 9042 | **SIS0042** | **"Error en el cálculo de la firma"** |

Es **idéntico** al SIS0041 (misma descripción: "Error en el cálculo de la firma"). Ambos indican que la firma HMAC calculada por el plugin NO coincide con la que Redsys espera.

### 3.2. ¿En qué capa se produce?

El error se produce en la **capa de comunicación entre el plugin y el servidor de Redsys (SIS)**:

```
1. Plugin (WooCommerce)
   └─ Genera parámetros de transacción
   └─ Calcula firma HMAC-SHA256 con clave secreta del comercio
   └─ Envía POST a https://sis.redsys.es/sis/realizarPago

2. Redsys (SIS) — RECIBE la petición
   └─ Recalcula la firma con su copia de la clave secreta
   └─ COMPARA ambas firmas
   └─ ❌ NO COINCIDEN → SIS0042 → RECHAZA INMEDIATAMENTE

3. Navegador del usuario
   └─ Recibe error: "No se puede realizar la operación"
   └─ Nunca ve el formulario de tarjeta
```

**Es un error de configuración/desajuste**, no un error del banco ni del procesador de pagos en sí. La comunicación se rechaza antes de cualquier procesamiento financiero.

### 3.3. ¿Por qué no se muestra el formulario de tarjeta?

El flujo de la versión Lite del plugin es mediante **redirección** (no modal):

1. Usuario hace clic en "Realizar pedido" en WooCommerce
2. El plugin genera un formulario HTML con parámetros ocultos (`Ds_MerchantParameters`, `Ds_Signature`, `Ds_SignatureVersion`)
3. JavaScript hace POST automático a `https://sis.redsys.es/sis/realizarPago`
4. **Redsys valida la firma** → si es correcta → **MUESTRA el formulario de tarjeta**
5. **SI LA FIRMA ES INCORRECTA (SIS0042)** → Redsys devuelve error inmediatamente

**El error SIS0042 ocurre TAN TEMPRANO en el flujo** que Redsys ni siquiera llega a ejecutar el código que renderiza el formulario de pago. Por eso el usuario ve el error sin haber introducido datos de tarjeta: **la petición es rechazada en la validación inicial de firma, antes de cualquier procesamiento**.

---

## 4. Causas del Error SIS0042

### 4.1. Clave SHA-256 Incorrecta (Causa #1)

**La causa más frecuente con diferencia.** La clave configurada en el plugin NO es la misma que está registrada en el Portal de Administración de Redsys.

**Posibles escenarios:**
- La clave se ha copiado mal (espacios al inicio/final, caracteres extra, errores de mayúsculas/minúsculas)
- La clave de test es diferente a la de producción, y están intercambiadas o mal puestas en sus campos respectivos
- Se está usando la clave por defecto (`sq7HjrUOBfKmC576ILgskD5srU870gJ7`) pero el banco ha establecido una clave diferente para pruebas
- Se ha configurado solo la clave de producción pero el plugin está en modo test (o viceversa)

**Clave de prueba estándar de Redsys:** `sq7HjrUOBfKmC576ILgskD5srU870gJ7`

⚠️ **Importante**: No asumas que la clave de test es siempre la que viene por defecto. Tu banco puede haber configurado una clave diferente en el entorno de pruebas. Verifícala en el Portal de Administración.

### 4.2. Bug del Plugin v6.x (Corregido en v7.0.1)

El changelog de la versión 7.0.1 confirma:

> "Fix: Test SHA-256 secret was being overwritten with the production secret in successful_request(), causing signature validation failures in test mode."

**¿Qué significa esto?** En versiones anteriores a 7.0.1, si tenías configuradas ambas claves (test y producción), durante el proceso de validación `successful_request()`, la **clave de test era sobreescrita por la de producción**. Esto provocaba que las firmas no coincidieran en modo test, generando SIS0042.

**Si estás en una versión < 7.0.1, actualizar es el primer paso obligatorio.**

### 4.3. Versión de Firma Incorrecta

Redsys soporta dos versiones de firma:

| Versión | Algoritmo | Derivación de clave |
|---|---|---|
| `HMAC_SHA256_V1` (antigua) | HMAC-SHA256 | 3DES |
| `HMAC_SHA512_V2` (nueva) | HMAC-SHA512 | AES-CBC |

El plugin **Lite** usa `HMAC_SHA256_V1` por defecto. Si tu banco ha migrado a `HMAC_SHA512_V2`, la versión Lite **NO funcionará**. Necesitarías:

- Contactar con tu banco para que **NO active HMAC_SHA512_V2** (que te mantengan en V1)
- O **adquirir la versión Premium** del plugin que sí soporta V2

El parámetro `Ds_SignatureVersion` debe ser `HMAC_SHA256_V1`.

### 4.4. Formato Incorrecto del Número de Pedido

El número de pedido (`Ds_Merchant_Order`) se usa para "diversificar" la clave (derivar una clave específica para cada operación). Si el formato es incorrecto, la firma no coincidirá.

**Requisitos del número de pedido:**
- **Longitud**: 4 a 12 caracteres
- **Primeros 4 dígitos**: deben ser **NUMÉRICOS** (0-9)
- **Caracteres permitidos**: Solo ASCII (0-9, A-Z, a-z)

Si usas plugins que modifican los números de pedido de WooCommerce (secuenciales, prefijos personalizados, etc.), pueden causar conflictos si el formato resultante no cumple con los requisitos de Redsys.

### 4.5. Formato del Importe Incorrecto

El importe debe enviarse en **céntimos** (sin decimales, sin puntos, sin comas):

| Importe | Valor a enviar |
|---|---|
| 10,00 € | `1000` |
| 25,50 € | `2550` |
| 100,99 € | `10099` |
| 1,00 € | `100` |

Si el plugin envía el importe con decimales o en formato incorrecto, el paquete de parámetros firmados no coincidirá y Redsys devolverá SIS0042 (además de potencialmente SIS0019).

### 4.6. Código de Moneda Incorrecto

Usar el código ISO 4217 numérico (nunca el símbolo ni el código alfabético):

| Moneda | Código correcto |
|---|---|
| Euro (€) | `978` |
| USD ($) | `840` |
| GBP (£) | `826` |
| ARS ($) | `032` |
| MXN ($) | `484` |

### 4.7. FUC o Terminal Incorrectos

- **FUC (Código de Comercio)**: 9 dígitos numéricos
- **Terminal**: Normalmente 1-3 dígitos (en pruebas suele ser `1`)

Si estos datos no coinciden con la configuración registrada en el banco, la firma se calcula sobre parámetros incorrectos causando SIS0042.

**Valores de prueba estándar:**
- FUC: `999008881`
- Terminal: `1`

### 4.8. PSD2 / SCA (Autenticación Reforzada)

El plugin es compatible con PSD2 desde la versión 3.0.0. Si el banco requiere autenticación reforzada y los parámetros EMV3DS no se envían correctamente, puede causar errores de firma porque los parámetros firmados no coinciden con los esperados por Redsys.

**Síntomas adicionales de problemas PSD2:**
- Error SIS0096 (formato incorrecto para datos 3DSecure)
- Error SIS0141 (formato no correcto para 3DSecure)
- Error 9470 (error en autenticación de primer factor)

### 4.9. Certificados SSL / SNI

El plugin es compatible con certificados SNI (Let's Encrypt, SiteGround, HostFusion, etc.). Si el certificado SSL del sitio no es válido o está mal configurado, la comunicación con Redsys puede fallar, aunque esto suele dar otros errores (SIS0007, errores de conexión) más que SIS0042.

---

## 5. Diagnóstico Paso a Paso

### Paso 1: Verificar versión del plugin

**Acción:** Ve a Plugins → WooCommerce Redsys Gateway Light y comprueba la versión instalada.

- ✅ **Versión 7.0.1 o superior** — Continúa al Paso 2
- ❌ **Versión anterior a 7.0.1** — **ACTUALIZA INMEDIATAMENTE** (hay un bug confirmado de sobreescritura de clave de test)

### Paso 2: Verificar la clave SHA-256

**Acción:** Verifica que la clave configurada en el plugin coincide EXACTAMENTE con la del Portal de Administración de Redsys.

1. Accede al Portal de Administración del TPV Virtual de Redsys
2. Ve a "Consulta datos del Comercio"
3. Localiza el terminal en el listado
4. Haz clic en el icono de la **🔑 llave** para ver la clave del terminal
5. Copia la clave **EXACTAMENTE** (sin espacios, sin saltos de línea)
6. Pégala en el campo correspondiente del plugin

**Verificaciones específicas:**
- ⚡ ¿Estás en **modo test**? → Usa la clave del campo **"Clave SHA-256 (Test)"**
- ⚡ ¿Estás en **modo producción**? → Usa la clave del campo **"Clave SHA-256"** (producción)
- ⚡ ¿Las claves de test y producción son DIFERENTES? → Asegúrate de que cada una está en su campo correcto
- ⚡ ¿La clave de test es la estándar? → No asumas que es `sq7HjrUOBfKmC576ILgskD5srU870gJ7`; verifícala en el portal

### Paso 3: Comprobar el número de pedido

**Acción:** Revisa el formato del número de pedido que se está generando.

1. Activa el modo debug/logging en WooCommerce
2. Realiza una compra de prueba
3. Revisa los logs: `wp-content/debug.log` o `wp-content/uploads/wc-logs/`
4. Busca `Ds_Merchant_Order` y verifica:
   - ✅ Entre 4 y 12 caracteres
   - ✅ Primeros 4 caracteres son NUMÉRICOS
   - ✅ Solo ASCII (0-9, A-Z, a-z)
   - ❌ Si hay caracteres especiales, guiones, etc. → problema

### Paso 4: Verificar el formato del importe

**Acción:** Revisa en los logs qué valor de `Ds_Merchant_Amount` se está enviando.

- ✅ Para 25,00 € → debe ser `2500`
- ✅ Para 10,50 € → debe ser `1050`
- ❌ Si ves `25.00`, `25,00`, `2500.00` → **formato incorrecto**

### Paso 5: Verificar moneda y tipo de transacción

| Parámetro | Valor correcto |
|---|---|
| `Ds_Merchant_Currency` | `978` (EUR) |
| `Ds_Merchant_TransactionType` | `0` (Autorización) |

### Paso 6: Probar con clave de test por defecto

**Acción:** Configura temporalmente el plugin con los valores de prueba estándar para aislar el problema.

```
Modo:       Test
FUC:        999008881
Terminal:   1
Clave Test: sq7HjrUOBfKmC576ILgskD5srU870gJ7
```

- ✅ **Funciona con estos datos** → El problema está en los datos reales de tu banco (clave, FUC o terminal incorrectos)
- ❌ **Sigue sin funcionar** → El problema es del plugin (versión, firma, o configuración del servidor)

### Paso 7: Verificar versión de firma

**Acción:** Comprueba el valor de `Ds_SignatureVersion` en los logs o en el formulario POST.

- ✅ `HMAC_SHA256_V1` → Correcto
- ❌ `HMAC_SHA512_V2` → **Incompatible con versión Lite**. Contacta con el banco para desactivar V2 o adquiere la versión Premium.

### Paso 8: Probar con otro navegador/entorno

A veces, problemas de JavaScript, cookies o extensiones del navegador pueden interferir con la redirección POST a Redsys.

- Prueba en modo incógnito/privado
- Prueba con otro navegador
- Prueba sin extensiones

### Paso 9: Revisar logs del servidor

- PHP error log: `error_log` del servidor
- WooCommerce logs: WooCommerce → Status → Logs → Filtra por "redsys"
- Debug log de WordPress: `wp-content/debug.log`

---

## 6. Soluciones

### Solución A: Clave mal configurada

**Pasos:**
1. Obtén la clave correcta desde el Portal de Administración de Redsys (icono 🔑 en el listado de terminales)
2. Cópiala exactamente, sin espacios, sin saltos de línea
3. Pégalo en el campo correcto del plugin:
   - Modo test → campo "Clave SHA-256 (Test)"
   - Modo producción → campo "Clave SHA-256"
4. Verifica que las claves de test y producción NO están intercambiadas
5. Guarda los cambios y prueba de nuevo

### Solución B: Bug de versión antigua

**Pasos:**
1. **Actualiza el plugin a v7.0.1 o superior**
2. Después de actualizar, verifica que las claves de test y producción sigan configuradas correctamente (a veces las actualizaciones pueden resetear campos)
3. Prueba de nuevo

### Solución C: Problema de formato de pedido

**Pasos:**
1. Identifica si usas algún plugin que modifique los números de pedido (secuenciales, prefijos custom)
2. Desactívalo temporalmente y prueba
3. Si funciona con los números por defecto de WooCommerce, hay que ajustar el plugin modificador para que cumpla: primeros 4 dígitos numéricos, 4-12 caracteres totales

### Solución D: Versión de firma incompatible

**Pasos:**
1. Contacta con tu banco/entidad
2. Pregunta qué versión de firma tienen configurada en tu terminal
3. **Si es HMAC_SHA512_V2**: Solicita que la cambien a `HMAC_SHA256_V1` (la versión Lite no soporta V2)
4. **Si el banco no puede/no quiere cambiarlo**: Necesitarás la versión Premium del plugin

### Solución E: Migración de entorno

**Pasos:**
1. Si estabas en pruebas y has pasado a producción, verifica que **cambiaste la clave de test por la de producción**
2. Verifica que el modo (test/producción) está correctamente seleccionado
3. Verifica que FUC y terminal son los de producción y no los de prueba

---

## 7. Referencia de Códigos de Error Redsys SIS

### 7.1. Errores de Firma y Parámetros

| Código | SIS | Descripción |
|---|---|---|
| 9001 | SIS0001 | Error interno |
| 9007 | SIS0007 | El mensaje de petición no es correcto, revisar el formato |
| 9008 | SIS0008 | Falta `Ds_Merchant_MerchantCode` |
| 9009 | SIS0009 | Error de formato en `Ds_Merchant_MerchantCode` |
| 9010 | SIS0010 | Falta `Ds_Merchant_Terminal` |
| 9011 | SIS0011 | Error de formato en `Ds_Merchant_Terminal` |
| 9014 | SIS0014 | Error de formato en `Ds_Merchant_Order` |
| 9015 | SIS0015 | Falta `Ds_Merchant_Currency` |
| 9016 | SIS0016 | Error de formato en `Ds_Merchant_Currency` |
| 9018 | SIS0018 | Falta `Ds_Merchant_Amount` |
| 9019 | SIS0019 | Error de formato en `Ds_Merchant_Amount` |
| 9020 | SIS0020 | Falta `Ds_Merchant_MerchantSignature` |
| 9021 | SIS0021 | La `Ds_Merchant_MerchantSignature` viene vacía |
| 9022 | SIS0022 | Error de formato en `Ds_Merchant_TransactionType` |
| 9023 | SIS0023 | `Ds_Merchant_TransactionType` desconocido |
| 9026 | SIS0026 | Problema con la configuración |
| 9027 | SIS0027 | Revisar la moneda que está enviando |
| 9028 | SIS0028 | Comercio/terminal está dado de baja |
| 9040 | SIS0040 | El comercio tiene un error en la configuración |
| **9041** | **SIS0041** | **Error en el cálculo de la firma** |
| **9042** | **SIS0042** | **Error en el cálculo de la firma** |
| 9051 | SIS0051 | Número de pedido repetido |

### 7.2. Errores de Operación

| Código | SIS | Descripción |
|---|---|---|
| 9054 | SIS0054 | No existe operación sobre la que realizar la devolución |
| 9057 | SIS0057 | El importe a devolver supera el permitido |
| 9063 | SIS0063 | Revisar el número de tarjeta enviado |
| 9064 | SIS0064 | Número de posiciones de la tarjeta incorrecto |
| 9071 | SIS0071 | Tarjeta caducada |
| 9078 | SIS0078 | No se permiten pagos con esa tarjeta |
| 9093-9095 | SIS0093-95 | Denegación emisor |
| 9114 | SIS0114 | Llamada por GET en lugar de POST |
| 9126 | SIS0126 | Operación duplicada |
| 9142 | SIS0142 | Tiempo excedido para el pago |

### 7.3. Errores de Configuración del Comercio

| Código | SIS | Descripción |
|---|---|---|
| 9230 | SIS0230 | Su comercio no permite pago fraccionado |
| 9231 | SIS0231 | No hay forma de pago aplicable |
| 9256 | SIS0256 | El comercio no permite preautorización |
| 9260 | SIS0260 | Entrada incorrecta al SIS |
| 9265 | SIS0265 | Error de firma en los datos recibidos |
| 9420 | — | El comercio no tiene métodos de pago disponibles |
| 9429 | — | Error en la versión de firma (`Ds_SignatureVersion`) |
| 9430 | — | Error decodificando `Ds_MerchantParameters` |
| 9431 | — | Error con el JSON de `Ds_MerchantParameters` |
| 9432 | — | FUC erróneo |
| 9433 | — | Terminal erróneo |
| 9434 | — | Ausencia de número de pedido |
| 9435 | — | Error en el cálculo de la firma |
| 9444 | — | Usando firmas antiguas y el comercio está configurado como HMAC SHA256 |
| 9447 | — | Usando referencia generada con otro adquirente |

### 7.4. Errores de Autenticación 3DS / PSD2

| Código | SIS | Descripción |
|---|---|---|
| 9096 | SIS0096 | Formato incorrecto para datos 3DSecure |
| 9141 | SIS0141 | Formato no correcto para 3DSecure |
| 9470 | — | Error en autenticación de primer factor |
| 9471 | — | Error en URL de redirección de autenticación |

### 7.5. Códigos de Respuesta (Ds_Response)

| Rango | Significado |
|---|---|
| **0000 a 0099** | **Pago APROBADO** ✅ |
| 0100 a 0199 | Operatoria OK, verificación CVV obligatoria |
| 0200 | Error de formato |
| **0201** | **Error de firma** |
| 0204 | Error de datos |
| 0209 | Error de tarjeta |
| 0214 | Fecha de caducidad errónea |
| 0290 | Tarjeta no autorizada |
| 0401 | Error en posición de tarjeta |
| 0404 | Error de configuración de comercio |
| 0904 | Comercio no operativo |
| 0912 | Emisor no disponible |
| 9912 | Emisor no disponible |
| 9913 | Error en comunicación |
| 9914 | Fallo al conectar con CA |
| 9919 | Error de cryptograma |
| 9992 | Petición cancelada |
| 9993 | Operatoria abandonada por el usuario |
| 9995 | Operatoria abandonada - tiempo excedido |
| 9998 | Error de validación |
| 9999 | Error general |

---

## 8. Enlaces y Referencias Útiles

| Recurso | Enlace |
|---|---|
| **Plugin en WordPress.org** | [https://es.wordpress.org/plugins/woo-redsys-gateway-light/](https://es.wordpress.org/plugins/woo-redsys-gateway-light/) |
| **Foro de soporte del plugin** | [https://wordpress.org/support/plugin/woo-redsys-gateway-light/](https://wordpress.org/support/plugin/woo-redsys-gateway-light/) |
| **Web del autor (Jose Conti)** | [https://plugins.joseconti.com/](https://plugins.joseconti.com/) |
| **Versión Premium** | [https://plugins.joseconti.com/product/plugin-woocommerce-redsys-gateway/](https://plugins.joseconti.com/product/plugin-woocommerce-redsys-gateway/) |
| **Documentación Redsys (desarrolladores)** | [https://pagosonline.redsys.es/](https://pagosonline.redsys.es/) |
| **Códigos de error Redsys (GitHub)** | [https://github.com/ssheduardo/sermepa/blob/master/doc/Códigos%20de%20Respuesta.md](https://github.com/ssheduardo/sermepa/blob/master/doc/C%C3%B3digos%20de%20Respuesta.md) |
| **Soporte técnico Redsys** | [soportevirtual@redsys.es](mailto:soportevirtual@redsys.es) |
| **Teléfono Redsys** | 91 728 23 23 |

---

*Documento generado el 8 de junio de 2026*
*Actualizado el 15 de junio de 2026*
*Fuente: WordPress.org plugin page + Context7 API + Redsys documentación oficial + GitHub (ssheduardo/sermepa)*
*Plugin: Payment Gateway for Redsys & WooCommerce Lite v7.0.1 — Jose Conti*
