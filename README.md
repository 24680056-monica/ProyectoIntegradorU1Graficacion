# ProyectoIntegradorU1Graficacion
El proyecto esta basado en la tarea Escenario Procedural agregando la animación de la cámara a través del camino. 

## Limpieza de escena e import de bibibliotecas.
Se agregan las bibliotecas que se necesitan para la generacion de las figuras.
```python
import bpy
import math
```
Ahora se debe eliminar todo lo que haya en la escena.
``` python
bpy.ops.object.select_all(action='SELECT')
bpy.ops.object.delete()
```
`bpy.ops.object.select_all(action='SELECT')` :  Selecciona todo lo de la escena.
`bpy.ops.object.delete()` : Elimina todo lo seleccionado.

## Creación del material y sus parámetros principales
``` python
def crear_material(nombre, r, g, b):
    mat = bpy.data.materials.new(name=nombre)
    mat.diffuse_color = (r, g, b, 1.0)
    return mat

mat_base = crear_material("GrisOscuro", 0.1, 0.1, 0.1)
mat_acento = crear_material("Neon", 0.0, 0.8, 1.0)
```
El `def crear_material` crea un material nuevo y le asigna color RGB además que devuelve el material
Creas dos:
- `mat_base` : Gris oscuro
- `mat_acento` : Azul neón <br>

``` python
# 2. Parámetros
largo = 30           
punto_curva = 0.3    
amplitud = 8      
suavizado = 15
fps = 30          # Fotogramas por segundo
duracion_seg = 10    # Segundos que dura el recorrido
total_frames = fps * duracion_seg
```
En esta parte se definen en las variables lo siguiente: 
- `largo` : Número de bloques
- `amplitud` : Lo fuerte que es la curva
-`suavizado` : Muestra como es progresiva es la entrada a la curva
- `fps y duracion_seg` : Duración de la animación
- `total_frames`  300 frames (10 segundos) <br>

## Creación de los bloques
``` python
for i in range(largo):
    n = max(0, i - punto_curva) 
    entrada_suave = min(1.0, n / suavizado)
    offset_curva = math.sin(n * 0.3) * amplitud * entrada_suave
    pos_y = i * 2
```
En el `for` se genera 30 filas de bloques, por consiguiente se calculan las curvas.
- `n = max(0, i - punto_curva) ` : a i (número del bloque) le resta punto_curva y evita valores negativos.
- `entrada_suave = min(1.0, n / suavizado)` : hace que la curva no aparezca de golpe, sino progresivamente.
- Finalmente en `offset_curva`:
   - `math.sin()` : genera una onda.
   - `amplitud` : controla qué tan grande es.
   - `entrada_suave` : evita que la curva empiece bruscamente.
- `pos_y = i * 2` : Cada bloque se separa 2 unidades en el eje Y, creando profundidad. <br>

``` python
    # Bloque Izquierdo
    bpy.ops.mesh.primitive_cube_add(location=(-3 + offset_curva, pos_y, 1))
    obj = bpy.context.active_object
    obj.data.materials.append(mat_base if i % 2 == 0 else mat_acento)
    if i % 2 != 0: obj.scale.z = 1.5
```
- Empieza en X = -3
- Se le suma la curva
- Z = 1 (altura)
- Alterna materiales en `obj.data.materials.append(mat_base if i % 2 == 0 else mat_acento)`, también generando algunos bloques más altos (`if i % 2 != 0: obj.scale.z = 1.5`) y alternando.
```python
    # Bloque Derecho
    bpy.ops.mesh.primitive_cube_add(location=(3 + offset_curva, pos_y, 1))
    bpy.context.active_object.data.materials.append(mat_base)
```
Este genera que sea simétrico al izquierdo.

## Creación y configuración de cámara
```python
# 4. CONFIGURAR CÁMARA
bpy.ops.object.camera_add()
camara = bpy.context.active_object
camara.rotation_euler = (math.radians(85), 0, 0) # Inclinada ligeramente hacia el frente
```
Inicialmente se crea la cámara (`....camera_add()`) y posteriormente se rota (85° en X) para que esté casi mirando hacia adelante, ligeramente inclinada.

```python
# 5. ANIMACIÓN MANUAL (Por Keyframes)
bpy.context.scene.frame_start = 1
bpy.context.scene.frame_end = total_frames

for f in range(1, total_frames + 1):
    # Calculamos en qué "posición de bloque" estaría la cámara según el frame
    # (Hacemos que recorra todo el 'largo' en 'total_frames')
    i_anim = (f / total_frames) * (largo - 1)
```
- `for f in range(1, total_frames + 1)`: Es para cada frame que calcula la posición de bloque.
- `i_anim = (f / total_frames) * (largo - 1)`: Hace que la cámara recorra todo el pasillo de manera proporcional al tiempo.
  
```python
    # Misma lógica de curva que los bloques
    n_anim = max(0, i_anim - punto_curva)
    entrada_anim = min(1.0, n_anim / suavizado)
    offset_anim = math.sin(n_anim * 0.3) * amplitud * entrada_anim
```
Se aplica la misma lógica que se empleó en la generación de los bloques para que la cámara siga exactamente el camino curvo.

```python
    # Actualizamos posición de la cámara
    camara.location.x = offset_anim
    camara.location.y = i_anim * 2
    camara.location.z = 1.8  # Altura de los ojos
```
Se posiciona la cámara para poder ver la animación bien y colocando 1.8 para la altura de los ojos.

```python
    # Insertamos el fotograma clave
    camara.keyframe_insert(data_path="location", frame=f)
```
Finalmente se insertan los keyframes manualmente.

## Suelo y luz
``` python
bpy.ops.mesh.primitive_plane_add(location=(0, largo, 0))
bpy.context.active_object.scale = (20, largo + 10, 1)
bpy.ops.object.light_add(type='POINT', location=(0, 10, 15))
luz = bpy.context.active_object
luz.data.energy = 10000

# Añadir una luz extra al final para que no esté oscuro
bpy.ops.object.light_add(type='POINT', location=(0, largo*2, 10))
bpy.context.active_object.data.energy = 5000
```
Para el suelo se crea la base donde se apoyan los bloques y donde se proyecta la luz.
- `bpy.ops.mesh.primitive_plane_add(location=(0, largo, 0))` : Creando un plano (Plane) en la escena y lo coloca centrado en X, al medio del recorrido en Y y en Z=0.
- `bpy.context.active_object.scale = (20, largo + 10, 1)` : escala el plano para que cubra todo el recorrido.
   - 20 : ancho del suelo en X.
   - largo + 10 : profundidad suficiente para cubrir todos los bloques.
   - 1 : no afecta la altura (porque es plano). <br>
Para la luz principal que ilumina el pasillo.
- `bpy.ops.object.light_add(type='POINT', location=(0, 10, 15))` : Agrega una luz tipo POINT (ilumina en todas direcciones).
  - X=0 → centrada.
  - Y=10 → cerca del inicio.
  - Z=15 → elevada.
- `luz.data.energy = 10000` : define la intensidad de la luz. <br>
Se añade una luz extra al final del recorrido para evitar que quede oscuro.
- `bpy.ops.object.light_add(type='POINT', location=(0, largo*2, 10))` : coloca la luz al final del pasillo.
    - largo*2: coincide con la última posición de los bloques.
- `bpy.context.active_object.data.energy = 5000` : intensidad menor que la principal.

## Vuelta al frame 1
Esto para que cada que termine el recorrido regresará al frame 1.
``` python
bpy.context.scene.frame_set(1)
```
# Muestra de como se ve
La figura se verá así: <br>
<img width="556" height="690" alt="image" src="https://github.com/user-attachments/assets/a0418f28-de3a-4f86-9fc8-937581a5481a" /> <br>
Mientras que en cuanto empiece el video de la cámara se ve así: <br>
<img width="695" height="622" alt="image" src="https://github.com/user-attachments/assets/556fb279-2257-433e-8745-a0ed8d1ff369" />

## Código completo
```python
import bpy
import math

# 1. Limpieza absoluta
bpy.ops.object.select_all(action='SELECT')
bpy.ops.object.delete()

def crear_material(nombre, r, g, b):
    mat = bpy.data.materials.new(name=nombre)
    mat.diffuse_color = (r, g, b, 1.0)
    return mat

mat_base = crear_material("GrisOscuro", 0.1, 0.1, 0.1)
mat_acento = crear_material("Neon", 0.0, 0.8, 1.0)

# 2. Parámetros
largo = 30           
punto_curva = 0.3    
amplitud = 8      
suavizado = 15
fps = 30          # Fotogramas por segundo
duracion_seg = 10    # Segundos que dura el recorrido
total_frames = fps * duracion_seg

# 3. CREACIÓN DE BLOQUES
for i in range(largo):
    n = max(0, i - punto_curva)
    entrada_suave = min(1.0, n / suavizado)
    offset_curva = math.sin(n * 0.3) * amplitud * entrada_suave
    pos_y = i * 2

    # Bloque Izquierdo
    bpy.ops.mesh.primitive_cube_add(location=(-3 + offset_curva, pos_y, 1))
    obj = bpy.context.active_object
    obj.data.materials.append(mat_base if i % 2 == 0 else mat_acento)
    if i % 2 != 0: obj.scale.z = 1.5

    # Bloque Derecho
    bpy.ops.mesh.primitive_cube_add(location=(3 + offset_curva, pos_y, 1))
    bpy.context.active_object.data.materials.append(mat_base)

# 4. CONFIGURAR CÁMARA
bpy.ops.object.camera_add()
camara = bpy.context.active_object
camara.rotation_euler = (math.radians(85), 0, 0) # Inclinada ligeramente hacia el frente

# 5. ANIMACIÓN MANUAL (Por Keyframes)
bpy.context.scene.frame_start = 1
bpy.context.scene.frame_end = total_frames

for f in range(1, total_frames + 1):
    # Calculamos en qué "posición de bloque" estaría la cámara según el frame
    # (Hacemos que recorra todo el 'largo' en 'total_frames')
    i_anim = (f / total_frames) * (largo - 1)
    
    # Misma lógica de curva que los bloques
    n_anim = max(0, i_anim - punto_curva)
    entrada_anim = min(1.0, n_anim / suavizado)
    offset_anim = math.sin(n_anim * 0.3) * amplitud * entrada_anim
    
    # Actualizamos posición de la cámara
    camara.location.x = offset_anim
    camara.location.y = i_anim * 2
    camara.location.z = 1.8  # Altura de los ojos
    
    # Insertamos el fotograma clave
    camara.keyframe_insert(data_path="location", frame=f)

# 6. Suelo y Luz
bpy.ops.mesh.primitive_plane_add(location=(0, largo, 0))
bpy.context.active_object.scale = (20, largo + 10, 1)
bpy.ops.object.light_add(type='POINT', location=(0, 10, 15))
luz = bpy.context.active_object
luz.data.energy = 10000

# Añadir una luz extra al final para que no esté oscuro
bpy.ops.object.light_add(type='POINT', location=(0, largo*2, 10))
bpy.context.active_object.data.energy = 5000

# Volver al frame 1
bpy.context.scene.frame_set(1)
```


