---
source: Redsys Official Docs / GitHub (ssheduardo/sermepa)
library: Redsys
package: woo-redsys-gateway-light
topic: Códigos de Error Redsys SIS
fetched: 2026-06-08T10:00:00Z
official_docs: https://github.com/ssheduardo/sermepa/blob/master/doc/Códigos%20de%20Respuesta.md
---

# Códigos de Error Redsys (SIS) - Referencia Rápida

Los errores SIS se devuelven como código de error (ej. SIS0042) o como código numérico (ej. 9042).

## Errores de Firma y Parámetros (los más relevantes)

| Código | SIS | Descripción |
|---|---|---|
| 9001 | SIS0001 | Error interno |
| 9007 | SIS0007 | El mensaje de petición no es correcto, debe revisar el formato |
| 9008 | SIS0008 | Falta Ds_Merchant_MerchantCode |
| 9009 | SIS0009 | Error de formato en Ds_Merchant_MerchantCode |
| 9010 | SIS0010 | Falta Ds_Merchant_Terminal |
| 9011 | SIS0011 | Error de formato en Ds_Merchant_Terminal |
| 9014 | SIS0014 | Error de formato en Ds_Merchant_Order |
| 9015 | SIS0015 | Falta Ds_Merchant_Currency |
| 9016 | SIS0016 | Error de formato en Ds_Merchant_Currency |
| 9018 | SIS0018 | Falta Ds_Merchant_Amount |
| 9019 | SIS0019 | Error de formato en Ds_Merchant_Amount |
| 9020 | SIS0020 | Falta Ds_Merchant_MerchantSignature |
| 9021 | SIS0021 | La Ds_Merchant_MerchantSignature viene vacía |
| 9022 | SIS0022 | Error de formato en Ds_Merchant_TransactionType |
| 9023 | SIS0023 | Ds_Merchant_TransactionType desconocido |
| 9026 | SIS0026 | Problema con la configuración |
| 9027 | SIS0027 | Revisar la moneda que está enviando |
| 9028 | SIS0028 | Error Comercio/terminal está dado de baja |
| 9040 | SIS0040 | El comercio tiene un error en la configuración, tienen que hablar con su entidad |
| **9041** | **SIS0041** | **Error en el cálculo de la firma** |
| **9042** | **SIS0042** | **Error en el cálculo de la firma** |
| 9051 | SIS0051 | Error número de pedido repetido |

## Errores de Operación

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

## Errores de Configuración del Comercio

| Código | SIS | Descripción |
|---|---|---|
| 9230 | SIS0230 | Su comercio no permite pago fraccionado |
| 9231 | SIS0231 | No hay forma de pago aplicable |
| 9256 | SIS0256 | El comercio no permite preautorización |
| 9260 | SIS0260 | Entrada incorrecta al SIS |
| 9265 | SIS0265 | Error de firma en los datos recibidos |
| 9420 | - | El comercio no tiene métodos de pago disponibles |
| 9429 | - | Error en la versión de firma (Ds_SignatureVersion) |
| 9430 | - | Error decodificando Ds_MerchantParameters |
| 9431 | - | Error con el JSON de Ds_MerchantParameters |
| 9432 | - | FUC erróneo |
| 9433 | - | Terminal erróneo |
| 9434 | - | Ausencia de número de pedido |
| 9435 | - | Error en el cálculo de la firma |
| 9444 | - | Se está intentando usar firmas antiguas y el comercio está configurado como HMAC SHA256 |
| 9447 | - | Usando referencia generada con otro adquirente |

## Errores de Autenticación 3DS / PSD2

| Código | SIS | Descripción |
|---|---|---|
| 9096 | SIS0096 | Formato incorrecto para datos 3DSecure |
| 9141 | SIS0141 | Formato no correcto para 3DSecure |
| 9470 | - | Error en autenticación de primer factor |
| 9471 | - | Error en URL de redirección de autenticación |

## Códigos de Respuesta (Ds_Response)

Estos códigos se devuelven en el parámetro `Ds_Response` de la respuesta de Redsys:

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

## Nota importante sobre SIS0041 vs SIS0042

Ambos códigos (SIS0041 y SIS0042) tienen la misma descripción: **"Error en el cálculo de la firma"**. En la documentación oficial aparecen como dos códigos distintos pero con el mismo significado. En la práctica, SIS0042 puede darse en contextos ligeramente diferentes (por ejemplo, REST vs redirección), pero la causa raíz es la misma: la firma calculada no coincide con la esperada por Redsys.
