# Guía de errores frecuentes

Esta guía **no da la solución** — orienta sobre los errores que más se
repiten en este tipo de ejercicio, para que los puedas evitar o
reconocerlos vos mismo si te aparecen.

## 1. Devolver `404` cuando la sala no existe

`GetById` en el repositorio puede devolver `null` sin problema (así lo
pide la interfaz: `Sala? GetById(int id)`). El error típico está en el
**controller**: si no revisás ese `null` antes de devolver la
respuesta, terminás mandando `200 OK` con el cuerpo vacío
(`Ok(null)`), cuando el enunciado pide explícitamente `404 Not Found`.
Revisá el resultado del repositorio antes de armar la respuesta.

## 2. Inyectar la interfaz, no la clase concreta — y registrarla en `Program.cs`

El controller recibe `IRepository` en el constructor (no
`SalaRepository`). Si en algún momento cambiás eso a la clase
concreta, vas a tener que volver a dejarlo como `IRepository` para que
compile contra lo que ya está declarado.

Program.cs **ya registra el servicio** — no hace falta tocarlo. Si
igual te aparece un error de inyección de dependencias al levantar el
proyecto, es señal de que algo en el constructor del repositorio no
coincide con lo que espera `Program.cs` (por ejemplo, el tipo del
parámetro del `DbContext`).

## 3. `DateTime` vs `DateOnly`

`Funcion.Fecha` es de tipo `DateOnly`, no `DateTime`. Para sacar "la
fecha de hoy" en UTC, la conversión se hace así:

```csharp
DateOnly.FromDateTime(DateTime.UtcNow)
```

Si intentás asignar un `DateTime` directo a `Fecha`, no compila.

## 4. Cargar la colección relacionada cuando hace falta (`Include`)

Las funciones de una sala (`sala.Funcions`) no se cargan solas al
traer una `Sala` de la base — por defecto viene vacía. Si tu consulta
necesita mirar las funciones de cada sala (para filtrar por
"pendientes", por ejemplo), tenés que pedirle explícitamente a EF
Core que las traiga con la sala.

*(Ojo con el nombre de la propiedad de navegación: es `Funcions`, no
`Funciones` — así quedó generado el modelo. Fijate con el autocompletado
del editor si no te cierra el nombre.)*

## 5. Filtrar con un parámetro opcional que puede venir nulo

El formato (`string? formato`) es opcional. Si no te llega nada (o
viene vacío), la consulta tiene que devolver **todas** las salas
activas con funciones pendientes, sin filtrar por formato. El error
típico es comparar contra un `string` nulo sin chequearlo antes, lo
que puede romper la consulta o, peor, hacer que nunca devuelva nada.

## 6. Comparar sin distinguir mayúsculas y minúsculas

El enunciado pide que el filtro por formato no distinga mayúsculas de
minúsculas (`imax`, `IMAX`, `Imax` tienen que dar el mismo resultado).
Comparar los strings tal cual vienen de la base y del parámetro, sin
normalizar antes, es la causa más común de que el filtro "a veces
funcione y a veces no".

## 7. Cada regla devuelve `false`, el controller traduce a `400` — nunca lanzar excepciones

Las reglas de negocio de `AssignScreeningToRoom` se expresan devolviendo
`true`/`false`, **no** lanzando excepciones. Si una regla falla,
el método corta ahí y devuelve `false` sin guardar nada en la base. Es
el **controller** el que decide qué código HTTP corresponde según ese
booleano (`400` si es `false`). Si en el repositorio lanzás una
excepción en vez de devolver `false`, el resultado va a ser un
`500 Internal Server Error` en vez del `400` que pide el enunciado.

## 8. Probar con el `.http` antes de entregar

El archivo `RepasoCineWebAPI.http` (dentro del proyecto) tiene todos
los casos que se van a revisar, con el código esperado en un
comentario arriba de cada uno. Correrlos todos antes de entregar es la
forma más rápida de darte cuenta si falta algo — sobre todo los casos
de los bordes (duración exactamente 240, una sala con exactamente 4
funciones pendientes).
