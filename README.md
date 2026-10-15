# RentCar — Parcial 2 de PGII

Implementación de referencia basada en el proyecto `RentCar-Starter` del profesor. Mantiene Java 21, Gradle, JavaFX y FXML, y agrega el proceso de reservación junto con los siete patrones obligatorios.

## Requisitos

- JDK 21.
- Gradle Wrapper incluido en el proyecto.
- Para probar la integración de pagos, inicia el servicio `external/payment-api.jar` antes de procesar reservas.

## Ejecutar

Windows PowerShell:

```powershell
.\gradlew.bat run
.\gradlew.bat test
```

Git Bash / macOS / Linux:

```bash
./gradlew run
./gradlew test
```

La primera ejecución puede requerir conexión a Internet para descargar Gradle y dependencias.

## API de pagos

Desde la carpeta `external`, ejecuta `java -jar payment-api.jar` y deja esa terminal abierta. El cliente usa `http://localhost:8080`.

La pantalla emplea los tokens de prueba de `external/README-api-pagos.md`:

| Token | Resultado esperado |
|---|---|
| `tok_approved` | Pago aprobado |
| `tok_declined` | Pago rechazado |
| `tok_api_error` | Error técnico simulado |

Son valores de prueba para el servicio académico, no credenciales reales.

## Patrones implementados

- **Builder:** `ReservationRequest.Builder` valida y construye la solicitud.
- **Abstract Factory:** `RentalModeFactory` con fábricas Basic, Standard y Premium crea familias de tarifa, cobertura y asistencia.
- **Facade:** `RentalFacade` coordina disponibilidad, precio y pago.
- **Adapter:** `PaymentApiAdapter` adapta `PaymentApiClient` a la interfaz interna `PaymentGateway`.
- **Decorator:** `ServiceQuoteDecorator` añade extras a una cotización existente.
- **Proxy:** `VerifiedPaymentProxy` verifica autorización antes de llamar al gateway real.
- **Composite:** `ServicePackage` agrupa servicios individuales y paquetes anidados mediante `ServiceComponent`.

Consulta `IMPLEMENTACION.md` para el mapa clase por clase y el flujo de ejecución.

## Decisiones para el ejemplo

- Los datos se almacenan en memoria y se pierden al cerrar la aplicación.
- La fecha final es la fecha de salida; la duración es el número de días entre inicio y salida.
- Los recargos de modalidades y los importes de servicios son valores demostrativos, modificables en las clases de `rental`.
- Solo una reservación `CONFIRMED` bloquea fechas. Los intentos fallidos se conservan para consulta, pero no bloquean el vehículo.
- El PDF menciona un servicio `CustomerVerificationService` proporcionado por el docente, pero ese archivo no venía en el ZIP revisado. Esta base incluye una versión local que consulta `Customer.isAuthorized()`. Si el profesor comparte la versión oficial, sustituye esa implementación sin modificar el Proxy.

## Entrega académica

Esta base de código no sustituye el UML ni el pensamiento computacional. El diagrama entregado por el equipo debe coincidir con el código final. Completa además los integrantes y las evidencias solicitadas en el enunciado.
