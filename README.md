Actividad 3.2.2 - Pipeline de Transacciones y Alertas Antifraude (Kafka Streams)
Implementación del caso de estudio corporativo utilizando Spring Boot y Kafka Streams para el procesamiento de eventos de pagos en tiempo real.

Plaintext
POST /api/pagos/procesar
  -> tx-pagos-entrantes (key = cliente)
  -> filtro: monto > 15 USD
  -> agregación: saldo acumulado por cliente (RocksDB State Store)
  -> detección: umbral crítico > 8.000 USD
  -> tx-alertas-criticas
La conversión de moneda está normalizada a USD directamente en el contrato de entrada. La clave cliente garantiza el ordenamiento estricto de los eventos por partición.

Requisitos del Entorno
Asegúrate de contar con las siguientes herramientas instaladas y configuradas en tu sistema:

Java 17 (JDK)

Maven (para la gestión de dependencias y empaquetado)

Docker Desktop (para levantar el broker de Kafka y los servicios de soporte)

Puesta en Marcha
Levantar la infraestructura con Docker:

PowerShell
docker compose up -d
Ejecutar la aplicación Spring Boot:

PowerShell
mvn spring-boot:run
Nota de puertos: La consola de Kafka UI operará en el puerto tradicional http://localhost:8080, mientras que la API REST del microservicio se configuró en el puerto http://localhost:9090 para evitar conflictos locales.

Pruebas y Validación End-to-End
Puedes utilizar curl desde PowerShell para simular el ingreso de transacciones al sistema:

1. Transacción regular (Suma al acumulador, sin alertas)
PowerShell
curl.exe -X POST "http://localhost:9090/api/pagos/procesar" -H "Content-Type: application/json" -d '{"cliente":"carlos_dev","monto":350.0,"canal":"web"}'
2. Transacción de alto riesgo (Supera el umbral y genera alerta)
PowerShell
curl.exe -X POST "http://localhost:9090/api/pagos/procesar" -H "Content-Type: application/json" -d '{"cliente":"carlos_dev","monto":9500.0,"canal":"pos_fisico"}'
3. Transacción descartada por el filtro inicial (Monto menor al mínimo)
PowerShell
curl.exe -X POST "http://localhost:9090/api/pagos/procesar" -H "Content-Type: application/json" -d '{"cliente":"carlos_dev","monto":10.0,"canal":"web"}'
Verificación en Kafka UI
Ingresa a http://localhost:8080 en tu navegador.

Dirígete a Topics > tx-alertas-criticas > Messages.

Deberías visualizar el evento de alerta generado por la segunda prueba con una estructura similar a esta:

JSON
{
  "cliente": "carlos_dev",
  "monto": 9500.0,
  "alerta": "Alerta por transaccion anomala detectada en canal pos_fisico",
  "nivelRiesgo": "CRITICO"
}
Limpieza del Entorno
Para apagar y eliminar los contenedores de infraestructura de forma limpia:

PowerShell
docker compose down -v
