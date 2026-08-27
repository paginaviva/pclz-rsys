# Verificación: Requisitos de BBVA contra Estado Real de la Web y Plugin Redsys

**Propósito:** Contrastar punto por punto lo que BBVA exige en `02_respuesta-redsys-20260615.md` contra lo que realmente existe en la web y en la configuración del plugin Redsys.

**Fecha de verificación:** 15 de junio de 2026

**Fuentes consultadas:**
- Email de BBVA: `02_respuesta-redsys-20260615.md`
- Web: https://www.centroluz.net/ (aviso-legal, política-privacidad, política-cookies, qué-es-portal-centro-luz)
- Plugin Redsys: WooCommerce REST API (woo-redsys-gateway-light v7.0.1 activo)

---

## Sobre el email

BBVA responde a una solicitud de **activación del TPV en modo real** para el comercio **361868854 - YOELI**, con URL **PORTAL.CENTROLUZ.NET**. La solicitud es **denegada temporalmente** porque no pueden validar la URL.

**Dos validaciones que BBVA necesita hacer:**
1. Verificar contenido legal de la web (bloqueado actualmente)
2. Realizar una simulación de compra como cliente final (pendiente hasta que se supere la primera)

---

## Requisito por requisito

### a) Nombre o denominación social

**El email dice:** debe incluirse en la web y coincidir con el contrato.

**Qué hay en la web:** "Yolanda González García" en el Aviso Legal.

**Se puede confirmar que coincide con el contrato?** NO. No tengo acceso al contrato firmado con BBVA. El email identifica al comercio como "361868854 - YOELI". En el plugin, el nombre de comercio configurado es "Portal Centro Luz (YOELI)".

### b) Inscripción en el Registro Mercantil

**El email dice:** debe incluirse si aplica ("en tu caso").

**Qué hay en la web:** Nada. No aparece en ninguna página.

**Se puede confirmar si aplica?** NO. No tengo acceso a los datos registrales del titular.

### c) CIF o NIF

**El email dice:** debe incluirse en la web y coincidir con el contrato.

**Qué hay en la web:** "25129287Q" en el Aviso Legal.

**Se puede confirmar que coincide con el contrato?** NO. No tengo acceso al contrato.

### d) Términos y condiciones con políticas de cancelación y reembolso

**El email dice:** deben incluirse términos y condiciones que mencionen las políticas de cancelación de pedidos y reembolso.

**Qué hay en la web:** No existe página de Términos y Condiciones. El Aviso Legal tiene "Condiciones de Uso" que cubren el uso del sitio web, no la contratación de productos o servicios. No hay ninguna mención a cancelación ni reembolso.

---

## Estado de la configuración técnica del plugin Redsys

Datos obtenidos del WooCommerce REST API (no están en el email, son contexto adicional):

| Parámetro | Valor | Notas |
|-----------|-------|-------|
| Plugin | woo-redsys-gateway-light v7.0.1 | Versión correcta (bug SIS0042 ya corregido) |
| Estado | Activado (enabled: true) | - |
| Modo | TEST (testmode: yes) | Sigue en pruebas, no en producción |
| FUC | 361868854 | Coincide con el email de BBVA |
| Nombre comercio | "Portal Centro Luz (YOELI)" | - |
| Terminal | 001 | - |
| Clave SHA-256 (producción) | `sq7HjrUOBfKmC576ILgskD5srU870gJ7` | Es la clave de test por defecto de Redsys |
| Clave SHA-256 (test) | `sq7HjrUOBfKmC576ILgskD5srU870gJ7` | Es la clave de test por defecto de Redsys |
| SNI | Activado (not_use_https: yes) | Necesario para el certificado SSL del sitio |
| Debug | Activado (debug: yes) | - |
| Idiomas pasarela | Español (001) | - |
| Otros gateways | Bizum, Google Pay, Inespay: desactivados | - |

---

## Qué implica esto

**La web** no puede ser validada por BBVA. Faltan:
- El Aviso Legal existe con nombre y NIF, pero no se puede confirmar si coinciden con el contrato.
- El Registro Mercantil no está mencionado en ninguna parte.
- No hay Términos y Condiciones de venta.
- No hay políticas de cancelación ni reembolso.

**El plugin** está listo técnicamente pero en modo test. Cuando BBVA valide la web y active la producción, habrá que:
- Cambiar testmode a "no".
- Sustituir la clave SHA-256 de producción por la real que asigne BBVA (la actual es la de test por defecto).

**Qué hay que hacer:** añadir la información faltante en la web y responder a **PASOPRODUCCIONTPVVIRTUAL@BBVA.COM** para que BBVA reanude la validación (incluyendo la simulación de compra).

---

## Tabla resumen: lo que pide BBVA, situación actual y acción necesaria

| # | Lo que pide BBVA / lo que falla | Situación actual en la web | Qué hay que hacer para solucionarlo |
|---|--------------------------------|---------------------------|--------------------------------------|
| 1 | **Nombre o denominación social** (art. 10.a LSSICE). Debe estar visible en la web y coincidir con el contrato. | Aparece "Yolanda González García" en el Aviso Legal. No se puede verificar si coincide con el contrato (no tengo acceso). | Confirmar que "Yolanda González García" coincide con el nombre del contrato de BBVA. Si no coincide, cambiarlo por el nombre exacto del contrato. |
| 2 | **Inscripción en el Registro Mercantil** (art. 10.b LSSICE). Debe incluirse "en tu caso" (solo si aplica). | No aparece en ninguna página de la web. | Determinar la forma jurídica del titular. Si es autónomo (persona física), no aplica. Si es sociedad, obtener los datos registrales (tomo, folio, hoja, inscripción) y publicarlos en el Aviso Legal. |
| 3 | **CIF o NIF** (art. 10.c LSSICE). Debe estar visible en la web y coincidir con el contrato. | Aparece "25129287Q" en el Aviso Legal. No se puede verificar si coincide con el contrato (no tengo acceso). | Confirmar que "25129287Q" coincide con el NIF/CIF del contrato de BBVA. Si no coincide, cambiarlo por el del contrato. |
| 4 | **Términos y condiciones** que incluyan las políticas de cancelación de pedidos y reembolso. | No existe página de Términos y Condiciones. El Aviso Legal solo contiene "Condiciones de Uso" del sitio web, no condiciones de contratación. No hay mención a cancelación ni reembolso. | Crear una página de Términos y Condiciones que regule la contratación de productos/servicios, e incluya expresamente las políticas de cancelación de pedidos y reembolso. |
| 5 | **Simulación de compra** (lo hará BBVA cuando la web sea validable). El email dice que necesitan "realizar una simulación de compra como si fuésemos el cliente final". | Pendiente. No se puede realizar hasta que BBVA pueda validar la URL. | Una vez corregidos los puntos 1-4, notificar a BBVA en **PASOPRODUCCIONTPVVIRTUAL@BBVA.COM**. Ellos harán la simulación de compra como parte de su proceso de validación. |
