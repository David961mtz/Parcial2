# Guía de implementación — RentCar

Esta guía explica cada archivo agregado o modificado. No incluye el UML ni el pensamiento computacional, ya que esos entregables los desarrolla tu compañera.

## 1. Clases originales que se conservan

- `app/RentCarApplication.java`: punto de entrada de JavaFX.
- `controller/MainController.java`: navegación entre pantallas.
- `controller/CustomerController.java` y `controller/VehicleController.java`: vistas existentes.
- `data/SampleData.java`: datos de ejemplo.
- `model/Customer.java` y `model/Vehicle.java`: modelos base.
- `repository/VehicleRepository.java` y `service/VehicleService.java`: registro y consulta de vehículos.
- `client/PaymentClientExample.java`: ejemplo de uso del cliente del profesor.

## 2. Archivos nuevos

### `model`

- `ReservationStatus.java`: enum de estados `PENDING`, `CONFIRMED` y `FAILED`.
- `ReservationRequest.java`: entrada inmutable del proceso. Su clase interna `Builder` construye y valida campos obligatorios, fechas, tarifa del vehículo y token de prueba.
- `RentalOutcome.java`: informa si terminó bien, el registro de reservación y el mensaje para la interfaz.

### `repository`

- `ReservationRepository.java`: almacena reservaciones en memoria, genera IDs, busca por ID y comprueba solapamientos entre reservas confirmadas del mismo vehículo.

### `service`

- `CustomerVerificationService.java`: punto de integración de la autorización. El ZIP no traía el servicio que el PDF dice que suministra el docente; la versión local consulta `Customer.isAuthorized()`. Se puede sustituir cuando se consiga el servicio oficial.
- `ReservationAvailabilityService.java`: comprueba disponibilidad.
- `RentalPricingService.java`: calcula la cotización según la modalidad y añade los servicios seleccionados mediante decoradores.
- `RentalFacade.java`: Facade que coordina disponibilidad, precio, pago y estado final.

### `rental`

- `RentalModeFactory.java`: abstracción de Abstract Factory.
- `RatePolicy.java`, `RentalCoverage.java`, `RoadsideAssistance.java`: productos abstractos de cada modalidad.
- `BasicRentalFactory.java`, `StandardRentalFactory.java`, `PremiumRentalFactory.java`: familias de productos para las tres modalidades.
- `BasicRatePolicy.java`, `StandardRatePolicy.java`, `PremiumRatePolicy.java`: aplican 1,00, 1,15 y 1,30 veces la tarifa diaria del vehículo por los días alquilados.
- `BasicCoverage.java`, `StandardCoverage.java`, `PremiumCoverage.java`: describen coberturas diferenciadas.
- `BasicAssistance.java`, `StandardAssistance.java`, `PremiumAssistance.java`: describen asistencias diferenciadas.
- `ServiceComponent.java`: contrato de Composite para servicios y paquetes.
- `GpsService.java`, `ChildSeatService.java`, `AdditionalDriverService.java`: extras individuales de COP 15.000, COP 10.000 y COP 20.000 por día, respectivamente.
- `ServicePackage.java`: paquete que contiene componentes y agrega recursivamente precios y descripciones; permite anidamiento.
- `ServiceCatalog.java`: crea un paquete familiar y otro paquete de viaje que contiene el paquete familiar.
- `RentalQuote.java`, `BaseRentalQuote.java`: contrato y cotización inicial.
- `RentalQuoteDecorator.java`, `ServiceQuoteDecorator.java`: Decorator; cada instancia incorpora un servicio o paquete a la cotización existente.

### `payment`

- `PaymentStatus.java`: estados normalizados `APPROVED`, `DECLINED`, `ERROR` y `BLOCKED`.
- `PaymentResult.java`: resultado independiente de la API externa.
- `PaymentGateway.java`: contrato de pago interno.
- `PaymentApiAdapter.java`: Adapter que construye `PaymentApiRequest`, invoca `PaymentApiClient` y traduce la respuesta al resultado interno.
- `VerifiedPaymentProxy.java`: Proxy que comprueba autorización antes de invocar el gateway real. Si el cliente no está autorizado, no se realiza la llamada externa.

## 3. Archivos existentes modificados

- `model/Reservation.java`: incluye estado tipado, total, descripción y mensaje del pago.
- `app/ApplicationServices.java`: configura objetos compartidos y conecta repositorios, servicios, Proxy, Adapter y Facade.
- `controller/ReservationController.java`: recoge los datos del formulario, construye la solicitud con Builder y delega en Facade.
- `resources/edu/rentcar/view/reservation-view.fxml`: formulario de reserva, modalidades, fechas, extras, paquetes, token de prueba, resultado e historial.

## 4. Recorrido de la reserva

1. El usuario selecciona cliente, vehículo, fechas, modalidad y extras en JavaFX.
2. `ReservationRequest.Builder` valida la solicitud.
3. `RentalFacade` consulta `ReservationAvailabilityService`.
4. `RentalPricingService` utiliza la familia de productos de la modalidad y calcula `RentalQuote`.
5. `ServiceQuoteDecorator` añade servicios o paquetes a la cotización.
6. `VerifiedPaymentProxy` verifica al cliente.
7. Si está autorizado, el Proxy delega en `PaymentApiAdapter`, que llama al cliente HTTP suministrado.
8. Solo el pago aprobado deja la reservación `CONFIRMED`; rechazo, falta de autorización o error dejan `FAILED`.
9. El controlador actualiza el resultado y el historial.

## 5. Reglas asumidas para el ejemplo

- La fecha final es la fecha de salida; duración = días entre inicio y salida.
- Los intervalos se tratan como `[startDate, endDate)`: una reservación puede empezar el mismo día en que termina otra.
- Las tarifas de modalidad y servicio son de demostración, no valores estipulados por el profesor.
- Solo las reservas confirmadas bloquean disponibilidad.

## 6. Pruebas

Los tests agregados cubren solicitud válida e inválida, alquiler aprobado, rechazo de pago, vehículo no disponible, cliente no autorizado sin llamada a la API, diferencias entre tarifas, Decorator con dos servicios y Composite anidado. El test de negocio simula el gateway para no depender de que el API HTTP esté levantado.

Desde JDK 21, desde la raíz del proyecto:

```powershell
.\gradlew.bat test
.\gradlew.bat run
```

En Git Bash / macOS / Linux:

```bash
./gradlew test
./gradlew run
```
