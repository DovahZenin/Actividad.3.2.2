# Actividad 3.2.2 - Pipeline de eventos bancarios end-to-end

Implementa el caso práctico de la guía utilizando Spring Boot y Kafka Streams.

```text
POST /api/banco/transaccion
  -> banco-transacciones (key = usuario)
  -> filtro: monto > 10 USD
  -> agregación: total acumulado por usuario (state store)
  -> detección: monto > 10.000 USD
  -> banco-alertas
```

La conversión a USD es simulada: los montos ingresados se asumen directamente en dólares, ya que el contrato definido en la guía no especifica moneda de origen. La clave `usuario` garantiza el orden secuencial de las operaciones de cada cliente dentro de su respectiva partición.

## Ejecución

Para levantar el proyecto es necesario contar con Docker Desktop, Java 17 y Maven instalados.

```powershell
docker compose up -d
mvn spring-boot:run
```

La interfaz de Kafka UI estará accesible en <http://localhost:8080>, mientras que la API REST opera en <http://localhost:8081> para evitar conflictos de puertos.

## Pruebas

Transacción estándar: se procesa correctamente y suma al acumulado sin disparar alertas.

```powershell
curl.exe -X POST "http://localhost:8081/api/banco/transaccion" -H "Content-Type: application/json" -d '{"usuario":"juan","monto":500,"origen":"app"}'
```

Transacción sospechosa: además de procesarse, emite una alerta hacia el tópico `banco-alertas`.

```powershell
curl.exe -X POST "http://localhost:8081/api/banco/transaccion" -H "Content-Type: application/json" -d '{"usuario":"juan","monto":15000,"origen":"cajero"}'
```

Transacción descartada: al no superar el monto mínimo, se filtra antes de la agregación y no genera alerta.

```powershell
curl.exe -X POST "http://localhost:8081/api/banco/transaccion" -H "Content-Type: application/json" -d '{"usuario":"juan","monto":5,"origen":"app"}'
```

Dentro de Kafka UI, navega hacia **Topics > banco-alertas > Messages**. Como resultado de la segunda prueba deberías visualizar un evento con el siguiente formato:

```json
{
  "usuario": "juan",
  "monto": 15000.0,
  "motivo": "Transaccion inusualmente alta en cajero",
  "severidad": "ALTA"
}
```

Para apagar y detener los contenedores:

```powershell
docker compose down
```
