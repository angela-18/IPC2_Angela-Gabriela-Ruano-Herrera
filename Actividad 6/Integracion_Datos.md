# Universidad de San Carlos de Guatemala

## Facultad de Ingeniería - Escuela de Ciencias

**Curso:** Introducción a la Programación y Computación 2

**Catedrático:** Ing. Jaime Francisco Yuman Ramirez

**Auxiliar:** Fernando José Vicente Velásquez

**Estudiante:** Angela Gabriela Ruano Herrera - 202406250

---

## Parte 1: Evaluación Conceptual y Buenas Prácticas

### Comparativa: CSV vs XML

| Característica | CSV | XML |
| --- | --- | --- |
| Legibilidad | Muy simple y fácil de leer en texto plano. | Más verboso, pero estructurado para jerarquías. |
| Tamaño | Menor tamaño, ideal para grandes volúmenes. | Más grande por etiquetas y metadatos. |
| Complejidad de parseo | Fácil de parsear con reglas simples. | Requiere parseo XML y manejo de nodos. |
| Soporte de jerarquía | Limitado a filas y columnas. | Soporta estructuras anidadas y metadatos. |
| Compatibilidad | Ampliamente compatible con hojas de cálculo y herramientas ETL. | Compatible con servicios web y datos estructurados complejos. |
| Validación | Sin validación de esquema nativa. | Se puede validar con DTD o XSD. |

### Serialización vs Deserialización con System.Text.Json

Serialización es el proceso de convertir un objeto de C# en una cadena JSON, usando `JsonSerializer.Serialize` para generar datos que pueden transmitirse o almacenarse. Deserialización es el proceso inverso: convertir una cadena JSON en una instancia de un tipo de C# mediante `JsonSerializer.Deserialize`, reconstruyendo el objeto desde el texto.

### Error N+1 y optimización por lotes

El error N+1 ocurre cuando una aplicación realiza una consulta principal y luego ejecuta N consultas adicionales para cada registro obtenido, lo que provoca muchas llamadas redundantes y mala performance. Para evitarlo en la lectura de archivos masivos se usa batching, es decir, agrupar registros en bloques y procesarlos o persistirlos en lotes grandes en lugar de fila por fila.

---

## Parte 2: Implementación Práctica en C#

### Desafío 1

```csharp
public async Task<Alumno?> ObtenerAlumnoAsync()
{
    try
    {
        using var client = new HttpClient();
        var url = "https://api.usac.edu/v1/alumnos";
        using var response = await client.GetAsync(url);
        response.EnsureSuccessStatusCode();
        var content = await response.Content.ReadAsStringAsync();
        var options = new JsonSerializerOptions
        {
            PropertyNameCaseInsensitive = true
        };
        return JsonSerializer.Deserialize<Alumno>(content, options);
    }
    catch (HttpRequestException)
    {
        return null;
    }
    catch (Exception)
    {
        return null;
    }
}
```

### Desafío 2

```csharp
[HttpPost]
public async Task<IActionResult> ImportarArchivoMasivo(IFormFile archivo)
{
    if (archivo == null || archivo.Length == 0)
    {
        return BadRequest();
    }

    var registros = new List<Alumno>();

    using var stream = archivo.OpenReadStream();
    using var reader = new StreamReader(stream);

    string? linea;
    while ((linea = await reader.ReadLineAsync()) != null)
    {
        var campos = linea.Split(',');
        var alumno = new Alumno
        {
            Id = int.Parse(campos[0]),
            Nombre = campos[1],
            Correo = campos[2]
        };
        registros.Add(alumno);
    }

    _context.Alumnos.AddRange(registros);
    await _context.SaveChangesAsync();

    return Ok();
}
```

---

## Referencias Bibliográficas

* Facultad de Ingeniería, USAC. (2026). Sesión 20: Integración de Datos. Consumo de APIs Externas y Carga Masiva (CSV/XML). Laboratorio del curso Introducción a la Programación y Computación 2. Guatemala.
