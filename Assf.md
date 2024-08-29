# Assf

[ASSFoundation](https://github.com/TypesettingTools/ASSFoundation) es un módulo que tiene como objetivo hacer que el trabajo con objetos de subtítulos sea eficiente. Realiza la mayor parte del trabajo pesado para que podamos hacer más con menos líneas de código en nuestro script. Proporciona diversas funciones que nos permiten trabajar con líneas, etiquetas, dibujos, texto y comentarios. Esto hace innecesario reinventar la rueda y escribir tus propias funciones para la mayoría de las tareas comunes, mientras te permite realizar tareas complejas de manera más sencilla.

Esta guía asume que ya sabes [cómo escribir scripts para Aegisub](https://unanimated.github.io/ts/ts-lua.htm) y conoces los conceptos básicos de [Moonscript](https://moonscript.org/reference/).

## Esqueleto de Scripts de Aegisub

```lua
export script_name = "nombre del script"
export script_description = "descripción de tu script"
export script_version = "0.0.1"
export script_author = "tú"
export script_namespace = "espacio de nombres de tu script"

DependencyControl = require "l0.DependencyControl"
depctrl = DependencyControl{
  {
    {"a-mo.LineCollection", version: "1.3.0", url: "https://github.com/TypesettingTools/Aegisub-Motion",
      feed: "https://raw.githubusercontent.com/TypesettingTools/Aegisub-Motion/DepCtrl/DependencyControl.json"},
    {"l0.ASSFoundation", version: "0.4.0", url: "https://github.com/TypesettingTools/ASSFoundation",
      feed: "https://raw.githubusercontent.com/TypesettingTools/ASSFoundation/master/DependencyControl.json"}
  }
}
LineCollection, ASS = depctrl\requireModules!
logger = depctrl\getLogger!

functionName = (sub, sel, act) ->
  -- aquí va el contenido

depctrl\registerMacro functionName
```

Este es el marco que tendrán todos tus scripts. Aquí importamos **LineCollection** y **ASSFoundation**. También importamos **Logger** para fines de registro, pero si no necesitas registrar nada, puedes eliminar esa línea. Finalmente, definimos una función llamada `functionName`. Este nombre de función es el que registramos en Aegisub en la última línea, y se ejecuta tan pronto como ejecutamos el script. Todo lo que hagamos en la guía a continuación irá dentro de la función donde dice `--aquí va el contenido`.

## LineCollection

LineCollection no es parte de ASSFoundation, pero casi todo lo que ASSFoundation hace actuará sobre la tabla de líneas generada por LineCollection. La tabla de líneas generada por LineCollection tendrá todos los campos de una [tabla de líneas](https://aeg-dev.github.io/AegiSite/docs/3.2/automation/lua/modules/karaskel.lua/) normal, pero también agrega otros campos. Algunos de los nuevos campos que son útiles para escribir automatizaciones en Aegisub son:

| Campos          | Significado                                   | Tipo    |
|-----------------|-----------------------------------------------|---------|
| duration        | Duración de la línea en ms                    | entero  |
| startFrame      | Primer fotograma de una línea                 | entero  |
| endFrame        | Último fotograma de una línea                 | entero  |
| styleRef        | Tabla de estilos                              | tabla   |
| number          | Número de una línea en el archivo de subtítulos | entero  |
| humanizedNumber | Número de una línea como se ve en Aegisub     | entero  |

Algunos de los métodos que LineCollection nos proporciona para modificar subtítulos son:

| Método                  | Uso                              | Significado                                                                 |
|-------------------------|----------------------------------|-----------------------------------------------------------------------------|
| LineCollection        | lines = LineCollection sub, sel | Añade todas las líneas seleccionadas a una variable llamada `lines` (ignora las líneas comentadas). |
| replaceLines          | lines\replaceLines!             | Cualquier cambio que hagas en `lines` se volverá a aplicar al archivo de subtítulos. |
| deleteLines           | lines\deleteLines!              | Elimina todas las líneas.                                                    |
|  | lines\deleteLines tbl           | Proporciona una tabla de líneas para eliminar solo esas líneas.               |
| insertLines           | lines\insertLines!              | Inserta líneas en el archivo de subtítulos.                                   |
|  | newLines\insertLines            | Inserta un nuevo conjunto de líneas que has definido llamado newLines.      |
| addLine               | lines\addLine                   | Añade una línea al archivo de subtítulos.                                      |


Puedes trabajar en la tabla de líneas generada por LineCollection sin usar ASSFoundation, como se muestra en el siguiente ejemplo donde cambiamos el efecto de la línea a "Actor":

```lua
functionName = (sub, sel) ->
  lines = LineCollection sub, sel
  for line in *lines
    line.effect = "Actor"
  lines\replaceLines!
```

Sin embargo, utilizaremos LineCollection junto con ASSFoundation para aprovechar al máximo ambos módulos.

## Logger

Logger es un módulo de registro de [DependencyControl](https://fansubbers.miraheze.org/wiki/DependencyControl) que puedes usar para registrar mensajes. Si no especificas un nivel de registro, el nivel de registro predeterminado es 2. Por defecto, el nivel de registro de Aegisub está configurado en 3, lo que significa que los mensajes con un nivel superior a 3 no serán vistos por el usuario final a menos que ellos mismos configuren un nivel de registro más alto. El script se detiene después de mostrar el mensaje si el nivel de registro es inferior a 2.

```lua
logger\log "Un mensaje simple entre comillas"
logger\log 4, "Un mensaje simple entre comillas pero con nivel de registro 4"

-- dump es la parte más útil de logger en lo que respecta a la depuración. 
-- Puedes pasar una tabla y mostrará una vista formateada con todas sus claves y valores.
logger\dump table

-- Con niveles de registro predefinidos:
logger\fatal "mensaje"                                  -- nivel de registro 0
logger\error "mensaje"                                  -- nivel de registro 1
logger\warn "mensaje"                                   -- nivel de registro 2
logger\hint "mensaje"                                   -- nivel de registro 3
logger\debug "mensaje"                                  -- nivel de registro 4
logger\trace "mensaje"                                  -- nivel de registro 5
logger\assert condition, "Mostrar este mensaje si la condición es false" -- nivel de registro 1
```

## Nombres de las etiquetas entendidas por ASSFoundation

| Etiqueta | Nombre en ASSFoundation  |
|----------|--------------------------|
| \fscx    | scale_x                  |
| \fscy    | scale_y                  |
| \an      | align                    |
| \frz     | angle                    |
| \fry     | angle_y                  |
| \frx     | angle_x                  |
| \bord    | outline                  |
| \xbord   | outline_x                |
| \ybord   | outline_y                |
| \shad    | shadow                   |
| \xshad   | shadow_x                 |
| \yshad   | shadow_y                 |
| \r       | reset                    |
| \pos     | position                 |
| \move    | move                     |
| \org     | origin                   |
| \alpha   | alpha                    |
| \1a      | alpha1                   |
| \2a      | alpha2                   |
| \3a      | alpha3                   |
| \4a      | alpha4                   |
| \1c      | color1                   |
| \2c      | color2                   |
| \3c      | color3                   |
| \4c      | color4                   |
| \clip    | clip_vect                |
| \iclip   | iclip_vect               |
| \clip    | clip_rect                |
| \iclip   | iclip_rect               |
| \p       | drawing                  |
| \be      | blur_edges               |
| \blur    | blur                     |
| \fax     | shear_x                  |
| \fay     | shear_y                  |
| \b       | bold                     |
| \i       | italic                   |
| \u       | underline                |
| \s       | strikeout                |
| \fsp     | spacing                  |
| \fs      | fontsize                 |
| \fn      | fontname                 |
| \k       | k_fill                   |
| \kf      | k_sweep                  |
| \ko      | k_bord                   |
| \q       | wrapstyle                |
| \fad     | fade_simple              |
| \fade    | fade                     |
| \t       | transform                |

## Recorrer todas las líneas usando LineCollection

```lua
lines = LineCollection sub, sel
return if #lines.lines == 0
lines\runCallback (lines, line, i) ->
  -- hacer algo con cada línea
```

Este es el método usual para recorrer todas las líneas seleccionadas. Por defecto, omite todas las líneas comentadas y recorre en orden inverso. Por lo tanto, la última línea seleccionada será la primera en la que trabajes. Generalmente, si tu script está destinado a trabajar en todas las líneas seleccionadas, esto es lo que usarías.

Si el orden es importante para ti, recorrerías las líneas seleccionadas de la siguiente manera:

```lua
lines = LineCollection sub, sel
return if #lines.lines == 0
lines\runCallback ((lines, line, i) ->
  -- hacer algo con cada línea pero en orden
), true
```

Si no te gusta que omita las líneas comentadas y quieres que se incluyan, recoge las líneas como se muestra a continuación:

```lua
lines = LineCollection sub, sel, (line) ->
  return true
```

De hecho, puedes dar cualquier condición aquí y si esa condición se cumple, solo se recogerán esas líneas. Por ejemplo, si solo quieres recoger líneas comentadas:

```lua
lines = LineCollection sub, sel, (line) ->
  return line.comment
```

Como ejemplo final, si solo quieres recoger líneas cuyo layer sea 1:

```lua
lines = LineCollection sub, sel, (line) ->
  return line.layer == 1
```

Hasta ahora, solo hemos recorrido las líneas seleccionadas, pero ¿qué pasa si queremos recorrer todas las líneas de diálogo? Naturalmente, asumí que `lines = LineCollection sub` sería el camino, pero no funcionó. Así que ideé una forma algo improvisada:

```lua
ln = LineCollection sub, sel
return if #ln == 0
start = ln[1].number - ln[1].humanizedNumber + 1
sel = [x for x = start, #sub]
lines = LineCollection sub, sel
```

Esto recorrerá todas las líneas de diálogo en tu archivo. Obviamente, puedes modificar la última línea para cambiar el orden del recorrido, así como la condición que debe cumplir una línea para ser recogida.

## Datos de Línea

Cuando recorremos cada línea, necesitamos crear algo llamado datos de línea. Esta es una tabla extensa que ASSFoundation crea después de analizar una línea y consiste en toda la información sobre tu línea. Todo lo que haremos al usar ASSFoundation es hacer cambios en esta tabla, lo cual a su vez hará cambios en la línea real. En el siguiente ejemplo, analizamos una línea y guardamos sus datos en una variable llamada `data`.

```lua
lines\runCallback (lines, line, i) ->
  data = ASS\parse line
```

La información que contienen los datos de línea es:

**line**: tabla de líneas con todos los campos.
**scriptInfo**: información sobre tu subtítulo como resolución, matriz de colores, etc.
**styles**: lista de todos los estilos en tu subtítulo con sus parámetros.
**sections**: diferentes secciones de una línea. Se explica a continuación.

## Secciones

En ASSFoundation, una línea puede tener cuatro tipos diferentes de secciones. Sus nombres hacen que sean autoexplicativos, así que solo se enumeran aquí:

 1. ASS.Section.Text
 2. ASS.Section.Tag
 3. ASS.Section.Drawing
 4. ASS.Section.Comment

## Métodos de Foundation

### [createLine](https://github.com/TypesettingTools/ASSFoundation/blob/ba2cace60efc39edfdedce1747b2b68aeff0af01/l0/ASSFoundation/FoundationMethods.moon#L74-L144)

Esto crea una nueva línea a partir de los datos que proporcionamos. Al crear una línea, puedes definir cualquier parámetro de la tabla de líneas, como efectos, actor, tiempo de inicio, etc. La línea creada se puede añadir al subtítulo usando **LineCollection**.

Si tienes una línea de subtítulo en bruto, puedes crear una línea de la siguiente manera:

```lua
lines = LineCollection sub, sel
return if #lines.lines == 0
newLine = ASS\createLine {
  "Dialogue: 0,0:00:00.00,0:00:05.00,Default,,0,0,0,,Test"
  lines
  actor: "Actor"
  start_time: 500
  end_time: 1000
  layer: 5
}
lines\addLine newLine
lines\insertLines!
```

También puedes crear una nueva línea basada en la línea en la que estás trabajando actualmente. Si haces cualquier modificación a los datos de la línea, todos esos cambios se reflejarán en la nueva línea.

```lua
lines = LineCollection sub, sel
lines\runCallback (lines, line, i) ->
  data = ASS\parse line
  data\stripTags!                      -- Haz algunos cambios en los datos de la línea, por ejemplo, elimina todas las etiquetas de la línea. Estos cambios afectarán a la nueva línea.
  newLine = ASS\createLine {
    line
    effect: "Effect"
    layer: line.layer - 1
  }
  lines\addLine newLine
lines\insertLines!
```

### [createTag](https://github.com/TypesettingTools/ASSFoundation/blob/ba2cace60efc39edfdedce1747b2b68aeff0af01/l0/ASSFoundation/FoundationMethods.moon#L65-L71)

Puedes crear una nueva instancia de etiqueta que se puede usar en varios lugares en ASSFoundation. Su uso será claro en la guía a continuación.

```lua
pos = ASS\createTag 'position', 5, 50                     -- \pos(5,50)
bord = ASS\createTag 'outline', 5                         -- \bord5
blur = ASS\createTag 'blur', 0.8                          -- \blur0.8
move = ASS\createTag 'move', 0, 0, 50, 50                 -- \move(0,0,50,50)
move = ASS\createTag 'move', 0, 0, 50, 50, 25, 500        -- \move(0,0,50,50,25,500)
clip_rect = ASS\createTag 'clip_rect', 50, 50, 500, 500   -- \clip(50,50,500,500)
drawing = ASS\createTag 'drawing', 1                      -- \p1
transfrom = ASS\createTag "transform", {tags}, t1, t2     -- transforma todas las etiquetas dentro de {tags} desde el tiempo t1 hasta el t2

-- creando Clip Vectorial
m = ASS.Draw.Move
l = ASS.Draw.Line
clip_vect = ASS\createTag 'clip_vect', {m(0,0), l(500,500), l(700,100)}         -- \clip(m 0 0 l 500 500 700 100)
```

## Varios Métodos para Datos de Línea

### [callback](https://github.com/TypesettingTools/ASSFoundation/blob/ba2cace60efc39edfdedce1747b2b68aeff0af01/l0/ASSFoundation/LineContents.moon#L174-L203)

Una función que se puede usar para recorrer todas o secciones específicas de una línea.

| Parámetro     | Significado                                                                 | Predeterminado | Tipo    |
|---------------|-----------------------------------------------------------------------------|----------------|---------|
| sectionClass  | Tipo de sección a recorrer                                                   | Si es nil, se recorre todas las secciones de la línea.            | Tipo de Sección ASS |
| start         | Índice de la sección desde la que empezar a recorrer (si es negativo, empieza a contar desde el final) | 1              | entero  |
| end           | Índice de la sección hasta la que finalizar el recorrido (si es negativo, empieza a contar desde el final) | Índice de la última sección de la línea | entero  |
| relative      | Contar el índice de inicio relativo al tiempo final                           | false          | booleano |
| reverse       | Invertir el orden del recorrido                                               | false          | booleano |

```lua
lines\runCallback (lines, line, i) ->
  data = ASS\parse line
  data\callback (section) ->
    if section.class == ASS.Section.Tag
      -- hacer algo con las etiquetas
    elseif section.class == ASS.Section.Text
      -- hacer algo con el texto
    elseif section.class == ASS.Section.Comment
      -- hacer algo con el comentario
    elseif section.class == ASS.Section.Drawing
      -- hacer algo con el dibujo
```

Si deseas trabajar en una sección individual, puedes hacer lo siguiente:

En el siguiente ejemplo, recorremos solo la sección de texto. Puedes reemplazar `ASS.Section.Text` con cualquier otra sección para recorrer solo esas secciones.

```lua
data = ASS\parse line
data\callback ((section) ->
  -- hacer algo con el texto de la sección
), ASS.Section.Text
```

Recorrer las primeras cinco secciones:

```lua
data = ASS\parse line
data\callback ((section) ->
  -- hacer algo con las etiquetas de la sección
), nil, 1, 5
```

Recorrer solo las primeras secciones de etiqueta dentro del índice 1 y 5:

```lua
data = ASS\parse line
data\callback ((section) ->
  -- hacer algo con la etiqueta de la sección
), ASS.Section.Tag, 1, 5
```

Recorrer secciones de etiqueta dentro de las últimas cinco secciones:

```lua
data = ASS\parse line
data\callback ((section) ->
  -- hacer algo con la etiqueta de la sección
), ASS.Section.Tag, -5
```

Usando relative, lo siguiente comenzará desde el índice '-2' y recorrerá hacia atrás 5 veces, recorriendo todas las secciones de etiqueta que encuentre.

```lua
data = ASS\parse line
data\callback ((section) ->
  -- hacer algo con la etiqueta de la sección
), ASS.Section.Tag, -5, -2, true
```

### [getEffectiveTags](https://github.com/TypesettingTools/ASSFoundation/blob/ba2cace60efc39edfdedce1747b2b68aeff0af01/l0/ASSFoundation/LineContents.moon#L353-L360)

| Parámetro         | Significado                                                                                                 | Tipo     |
|-------------------|-------------------------------------------------------------------------------------------------------------|----------|
| index             | Índice de la sección de la cual se requiere el valor efectivo de la etiqueta                                 | entero   |
| includeDefault    | Incluir el valor de la etiqueta de estilo si no se encuentra la etiqueta de anulación                        | booleano |
| includePrevious   |                                                        | booleano |
| copyTags          |                                                                                          | booleano |

Puedes usar este método para obtener el valor efectivo de cualquier etiqueta para una sección en particular. Si la etiqueta no está presente como etiqueta de anulación, también puedes obtener el valor predeterminado del estilo.

```lua
data = ASS\parse line

-- Obtener valores de etiquetas efectivas para la última sección
tags = (data\getEffectiveTags -1, true, true, false).tags

-- Luego puedes obtener un objeto de etiqueta como:
align = tags.align
outline = tags.outline

-- Luego puedes hacer cambios a la etiqueta. Estos cambios se aplicarán al índice que elegiste al obtener las etiquetas efectivas.
-- Establecer alineación a 7
align\set 7
-- Añadir 5 al borde
outline\add 5

-- Aplicar estos cambios a la línea
data\commit!
```

### [insertTags](https://github.com/TypesettingTools/ASSFoundation/blob/ba2cace60efc39edfdedce1747b2b68aeff0af01/l0/ASSFoundation/LineContents.moon#L331-L347)

| Parámetro         | Significado                                                                                                 | Tipo         |
|-------------------|-------------------------------------------------------------------------------------------------------------|--------------|
| tag               | Instancia de la etiqueta que deseas insertar                                                                | objeto de etiqueta |
| index             | Índice en el cual insertar la etiqueta. Solo considera secciones de etiquetas                               | entero       |
| sectionPosition   | Posición dentro del bloque de etiquetas donde la etiqueta debe ser insertada                                | entero       |
| direct            | Si es true, intenta insertar etiquetas directamente en el índice proporcionado. Si la sección no es de tipo etiqueta, falla | booleano     |

Este método se puede usar para insertar una etiqueta en la línea. La etiqueta que estás insertando debe ser un objeto de etiqueta como lo entiende AssFoundation. Puedes definir el bloque de etiquetas donde deseas insertar la etiqueta, y también puedes definir el orden en el que se insertará la etiqueta. Para una línea `{\tag1\tag2}Sección de Texto{\tag3\tag4}`, 'index' y 'sectionPosition' serán como se muestra en la imagen a continuación.

![image](https://static.miraheze.org/fansubberswiki/7/7b/InsertTags.png)

```lua
data = ASS\parse line

-- Obtener valores de etiquetas efectivas para la última sección
tags = (data\getEffectiveTags -1, true, true, false).tags

-- Insertar ese valor de etiqueta en la línea
data\insertTags tags.shadow       -- Añade \shad al primer bloque de etiquetas
data\insertTags tags.scale_x, 2   -- Añade \fscx al segundo bloque de etiquetas
data\insertTags tags.scale_y, -1  -- Añade \fscy al último bloque de etiquetas
```

### [getDefaultTags](https://github.com/TypesettingTools/ASSFoundation/blob/ba2cace60efc39edfdedce1747b2b68aeff0af01/l0/ASSFoundation/LineContents.moon#L636-L697)

| Parámetro    | Significado                                                      | Tipo     | Predeterminado             |
|--------------|------------------------------------------------------------------|----------|----------------------------|
| style        | Estilo para obtener el valor                                      | string   | Estilo de la línea actual  |
| copyTags     |                                                   | booleano | true                       |
| useOvrAlign  | Considerar la etiqueta de alineación para obtener la posición actual | booleano | true                       |

Este método se puede utilizar para obtener los valores de las etiquetas en el estilo. Por defecto, encuentra las etiquetas de la línea actual, pero podemos pasar cualquier estilo presente en los subtítulos para obtener el valor de la etiqueta de ese estilo.

```lua
data = ASS\parse line
styleTags = (data\getDefaultTags!).tags

-- Luego podemos acceder a una tabla para cada etiqueta de la siguiente manera
angleTable = styleTags.angle

-- Podemos obtener directamente el valor de la etiqueta así:
angle = styleTags.angle\get!
```

### [insertDefaultTags](https://github.com/TypesettingTools/ASSFoundation/blob/ba2cace60efc39edfdedce1747b2b68aeff0af01/l0/ASSFoundation/LineContents.moon#L349-L351)

| Parámetro        | Significado                                                                 | Tipo       |
|------------------|-----------------------------------------------------------------------------|------------|
| tagnames         | Nombres de la etiqueta o tabla con nombres de etiquetas                     | string o tabla |
| index            | Índice en el cual insertar la etiqueta. Solo considera secciones de etiquetas | entero     |
| sectionPosition  | Posición dentro del bloque de etiquetas donde la etiqueta debe ser insertada | entero     |
| direct           | Si es true, intenta insertar etiquetas directamente en el índice proporcionado. Si la sección no es de tipo etiqueta, falla | booleano   |

Este método inserta los valores de estilo de una etiqueta directamente en la línea.

```lua
data = ASS\parse line

-- Insertar una única etiqueta en el primer bloque de etiquetas. Si el primer bloque de etiquetas no existe, se agrega uno.
data\insertDefaultTags "align"

-- Insertar múltiples etiquetas en el primer bloque de etiquetas
data\insertDefaultTags {"scale_x", "scale_y", "blur"}

-- Insertar la etiqueta en el segundo bloque de etiquetas. Si el segundo bloque de etiquetas no existe, no sucede nada.
data\insertDefaultTags "fontname", 2

-- Dado que direct es true, intenta encontrar la tercera sección. Si la tercera sección no existe o no es una sección de etiquetas, genera un error.
data\insertDefaultTags "outline", 3, nil, true
```

Una forma de cambiar el valor de la etiqueta predeterminada que insertaste es:

```lua
data = ASS\parse line
blur = data\insertDefaultTags "blur"      -- Insertar el valor predeterminado de blur
blur.value = 5                            -- Cambiar el valor de la etiqueta que ya has insertado
data\commit!
```

### [cleanTags](https://github.com/TypesettingTools/ASSFoundation/blob/ba2cace60efc39edfdedce1747b2b68aeff0af01/l0/ASSFoundation/LineContents.moon#L395-L449)

Este método se usa para limpiar, ordenar y fusionar etiquetas en una línea.

| Parámetro                  | Significado                                                                                     | Tipo      | Predeterminado |
|----------------------------|-------------------------------------------------------------------------------------------------|-----------|----------------|
| level                       | *Explicado a continuación*                                                                        | entero    | 3              |
| mergeConsecutiveSections    | Fusionar secciones de etiquetas consecutivas                                                   | booleano  | true           |
| defaultToKeep               | Evita la eliminación de estas etiquetas                                                         | tabla     | {}             |
| tagSortOrder                | Determina el orden en que se ordenarán las etiquetas limpiadas dentro de una sección de etiquetas. Resets siempre van primero, transforms al final | tabla     | {}             |

Niveles de Limpieza:

- 0: Sin limpieza
- 1: Eliminar secciones de etiquetas vacías
- 2: Eliminar etiquetas duplicadas dentro de secciones
- 3: Eliminar etiquetas duplicadas globalmente
- 4: Eliminar etiquetas que coinciden con los valores predeterminados del estilo y otras etiquetas inefectivas

```lua
data = ASS\parse line
data\cleanTags!                                         -- Limpiar etiquetas usando los valores predeterminados de los parámetros
data\cleanTags 1                                        -- Limpiar etiquetas usando el nivel de limpieza 1
```

ASSFoundation ofrece una tabla para ordenar etiquetas. El orden de las etiquetas es el siguiente:

```lua
{"align", "position", "move", "origin", "scale_x", "scale_y", "angle", "angle_y", "angle_x", "shear_x", "shear_y", "fontname", "fontsize", "spacing", "bold", "italic", "underline", "strikeout", "outline", "outline_x", "outline_y", "shadow", "shadow_x", "shadow_y", "color1", "color2", "color3", "color4", "alpha", "alpha1", "alpha2", "alpha3", "alpha4", "blur", "blur_edges", "fade_simple", "fade", "clip_rect", "iclip_rect", "clip_vect", "iclip_vect", "wrapstyle", "drawing", "k_fill", "k_sweep", "k_bord", "junk", "unknown"}
```

```lua
data\cleanTags nil, nil, nil, ASS.tagSortOrder
```

Si deseas definir tu propio orden, asegúrate de que **tagSortOrder** incluya todas las etiquetas de la tabla, ya que si omites alguna, será eliminada durante la limpieza. Las etiquetas en la tabla **tagSortOrder** deben ser los nombres de las etiquetas según lo entendido por ASSFoundation. Debes reorganizar la tabla de etiquetas anterior según cómo te convenga. Sin embargo, puedes convertir los nombres de las etiquetas de sobrescritura como se muestra a continuación.

```lua
-- Primero define una tabla con las etiquetas en el orden que deseas
tagSortOrder = {"\\an", "\\pos", "\\move", "\\org", "\\fscx", "\\fscy", "\\frz", "\\fry", "\\frx", "\\fax", "\\fay", "\\fn", "\\fs", "\\fsp", "\\b", "\\i", "\\u", "\\s", "\\bord", "\\xbord", "\\ybord", "\\shad", "\\xshad", "\\yshad", "\\1c", "\\2c", "\\3c", "\\4c", "\\alpha", "\\1a", "\\2a", "\\3a", "\\4a", "\\blur", "\\be", "\\fad", "\\fade", "clip_rect", "iclip_rect", "clip_vect", "iclip_vect", "\\q", "\\p", "\\k", "\\kf", "\\K", "\\ko", "junk", "unknown"}

-- Convierte los nombres de las etiquetas de sobrescritura a los nombres de etiquetas entendidos por ASSFoundation
tagSortOrder = ASS\getTagNames tagSortOrder

-- Luego úsalos en cleanTags
data\cleanTags nil, nil, nil, tagSortOrder
```

### [removeTags](https://github.com/TypesettingTools/ASSFoundation/blob/ba2cace60efc39edfdedce1747b2b68aeff0af01/l0/ASSFoundation/LineContents.moon#L313-L329)

Este método se puede utilizar para eliminar etiquetas ya presentes en la línea.

| Parámetro | Significado                                            | Tipo   |
|-----------|--------------------------------------------------------|--------|
| tags      | nombre de la etiqueta o tabla de nombres de etiquetas | string o tabla |
| start     | índice de la sección desde la cual comenzar a eliminar etiquetas | entero |
| end       | índice de la sección hasta la cual eliminar etiquetas | entero |
| relative  |  | booleano |

```lua
data = ASS\parse line

-- Puedes pasar un nombre de etiqueta único para eliminarlo. Esto elimina \bord de todos los bloques de etiquetas.
data\removeTags "outline"

-- Puedes pasar una tabla de nombres de etiquetas para eliminarlas todas.
data\removeTags {"align", "blur"}

-- Elimina la etiqueta blur del primer bloque de etiquetas.
data\removeTags "blur", 1, 1

-- Elimina la etiqueta blur del segundo al quinto bloque de etiquetas.
data\removeTags "blur", 2, 5

-- Elimina \shad del último bloque de etiquetas.
data\removeTags "shadow", -1
```

Después de eliminar etiquetas usando **removeTags**, podrían quedar llaves {} sueltas si esa es la única etiqueta en ese bloque de etiquetas. Se recomienda usar [cleanTags](https://fansubbers.miraheze.org/wiki/User:PhosCity/Assf#cleanTags) para eliminarlas.

Si deseas eliminar una etiqueta pero al mismo tiempo modificarla de alguna manera, es muy útil guardar la etiqueta en una variable utilizando **removeTags**. Una vez guardada en una variable, esta variable puede ser manipulada por separado mientras la etiqueta se habrá eliminado de la línea.

```lua
path = data\removeTags({"clip_vect", "iclip_vect"})
```

### [replaceTags](https://github.com/TypesettingTools/ASSFoundation/blob/ba2cace60efc39edfdedce1747b2b68aeff0af01/l0/ASSFoundation/LineContents.moon#L265-L310)

| Parámetro        | Significado                                                                 | Tipo      |
|------------------|-----------------------------------------------------------------------------|-----------|
| tagList        | Instancia de etiqueta que deseas reemplazar. | objeto de etiqueta de ASSFoundation |
| start          | Índice de la sección desde la cual comenzar a reemplazar etiquetas.        | entero    |
| end            | Índice de la sección hasta la cual reemplazar etiquetas.                    | entero    |
| relative       | Si es true, solo se consideran las secciones de etiqueta (es decir, el índice 2 significa la segunda sección de etiqueta, no la segunda sección literal de la línea). | booleano  |
| insertRemaining| Si es true (por defecto), si la etiqueta que estás reemplazando no existe en la línea, se agrega la etiqueta a la línea en lugar de reemplazar la existente. | booleano  |

Este método inserta los valores de estilo de una etiqueta directamente en la línea.

```lua
data = ASS\parse line

-- Define objetos de etiquetas \bord y \shad
bord = ASS\createTag "outline", 5
shad = ASS\createTag "shadow", 10

-- Reemplaza todas las etiquetas bord
data\replaceTags bord

-- Reemplaza bord desde el segundo hasta el quinto bloque de etiquetas
data\replaceTags bord, 2, 5, true

-- Reemplaza la etiqueta bord solo si existe en la línea
data\replaceTags bord, _, _, _, false

-- Reemplaza múltiples etiquetas a la vez pasando una tabla de etiquetas
data\replaceTags {bord, shad}
```

También puedes crear una nueva instancia de etiqueta y reemplazarla en una sola línea.

```lua
data = ASS\parse line
data\replaceTags {ASS\createTag "angle", 5}
data\commit!
```

## Modificación de texto

### Cambiar todo el texto

### Añadir al texto existente

### Anteponer al texto existente