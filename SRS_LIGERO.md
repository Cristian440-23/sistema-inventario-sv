# SRS_LIGERO.md
## 1. Alcance del Sistema (Scope)
El sistema consistirá en una plataforma web de control de inventario para una tienda local que permita el seguimiento riguroso de productos y stock.

* **Gestión de Inventario:** Registro, actualización, visualización y eliminación de productos.
* **Control Estricto de Existencias:** El sistema impedirá transacciones que resulten en stock negativo. Cada movimiento de inventario quedará registrado con bitácora de usuario, fecha y hora.
* **Alertas de Stock Mínimo:** Generación de alertas visuales automáticas cuando las unidades de un producto sean iguales o inferiores al límite mínimo establecido.
* **Validación de Identificadores:** Verificación obligatoria de que los productos correspondan al mercado de El Salvador.

---

## 2. Restricciones Técnicas
* **Base de Datos:** Uso obligatorio y exclusivo de **PostgreSQL** como motor de base de datos relacional para garantizar la integridad referencial y concurrencia.
* **Estándar de Validación Geográfica:**
  * El sistema solo aceptará códigos de barras bajo el estándar **EAN-13**.
  * Se validará estrictamente que el código inicie con el prefijo de país **741** (perteneciente a El Salvador) y que posea exactamente 13 dígitos numéricos válidos según el algoritmo del dígito verificador.
* **Plataforma:** Aplicación web accesible mediante navegadores modernos (Chrome, Firefox, Edge, Safari).

---

## 3. Historias de Usuario (US)

### US-01: Validación de Códigos de Barra de El Salvador al Registrar Productos
* **COMO:** Administrador de inventario.
* **QUIERO:** Que el sistema valide que el código de barras ingresado sea de El Salvador (prefijo 741 y de 13 dígitos).
* **PARA:** Asegurar que solo se registren productos distribuidos legalmente en el territorio nacional.

**Criterios de Aceptación (Gherkin):**
* **Escenario 1: Registro exitoso de producto nacional**
  * **Dado** que el administrador está en la pantalla de "Registrar Producto"
  * **Cuando** ingresa un código de barras válido de 13 dígitos que inicia con "741" (Ej: `7411234567890`)
  * **Entonces** el sistema debe permitir guardar el registro en la base de datos de PostgreSQL.
* **Escenario 2: Rechazo de código extranjero o inválido**
  * **Dado** que el administrador está en la pantalla de "Registrar Producto"
  * **Cuando** ingresa un código que no inicia con "741" (Ej: `7501234567890`) o no posee 13 dígitos
  * **Entonces** el sistema debe denegar el guardado y mostrar un mensaje de alerta: *"Error: El código de barras debe ser EAN-13 válido y pertenecer a El Salvador (Prefijo 741)"*.

### US-02: Control de Stock Estricto y Alerta de Stock Mínimo
* **COMO:** Encargado de tienda.
* **QUIERO:** Que el sistema me impida registrar salidas si no hay existencias, y me notifique cuando un producto se esté agotando.
* **PARA:** Evitar descuadres de inventario (números negativos) y gestionar compras de reabastecimiento a tiempo.

**Criterios de Aceptación (Gherkin):**
* **Escenario 1: Bloqueo de venta sin existencias**
  * **Dado** que un producto tiene un stock disponible de `0` unidades
  * **Cuando** se intenta registrar una salida o venta de dicho producto
  * **Entonces** el sistema debe impedir la transacción y mostrar el error: *"Acción no permitida: Stock insuficiente"*.
* **Escenario 2: Activación de alerta por stock mínimo alcanzado**
  * **Dado** que un producto tiene un stock mínimo configurado de `5` unidades y stock actual de `6` unidades
  * **Cuando** se registra la salida de `2` unidades (dejando el stock en `4`)
  * **Entonces** el sistema debe actualizar las existencias en la base de datos y activar inmediatamente un indicador visual de "Stock Crítico / Alerta" en el panel principal.

---

## 4. Matriz de Trazabilidad

| ID Req. | Descripción del Requerimiento | Historia de Usuario | Componente de Software / Base de Datos | Caso de Prueba Relacionado |
| :--- | :--- | :--- | :--- | :--- |
| **REQ-01** | El sistema debe validar que los códigos de barra correspondan estrictamente a El Salvador. | **US-01** | Función `validarEAN13SV()` en backend / Atributo `codigo_barra` en tabla `productos` de PostgreSQL. | **CP-01:** Intento de registro de código con prefijo `750` (Resultado esperado: Rechazado y bloqueado). |
| **REQ-02** | Control estricto de existencias y generación de alertas de stock mínimo. | **US-02** | Trigger de base de datos en tabla `movimientos_inventario` / Vista de Alertas en Dashboard. | **CP-02:** Intento de registrar salida de producto con stock `0` (Resultado esperado: Transacción revertida). |
