# Resumen de Funcionalidades de OpenCV: Transformaciones Geométricas

Este documento recopila las principales funciones, métodos y conceptos aplicados durante la práctica de procesamiento de imágenes con OpenCV y Python.

## 1. Transformaciones Afines (2D)

Las transformaciones afines preservan las líneas paralelas. Se representan mediante una matriz de transformación de dimensión $2 \times 3$.

### Función Principal

```
cv2.warpAffine(src, M, dsize, borderMode=cv2.BORDER_CONSTANT, borderValue=0)

```

* **`src`**: Imagen de entrada.

* **`M`**: Matriz de transformación afín ($2 \times 3$).

* **`dsize`**: Tupla `(ancho, alto)` que define la dimensión de la imagen de salida.

* **`borderMode`**: Método para rellenar las zonas desocupadas tras la transformación.

* **`borderValue`**: Color de relleno en formato escalar o tupla (ej. `(255, 255, 255)` para fondo blanco).

### Construcción de Matrices Afines

1. **Traslación Manual:**
   

   $$
   M = \begin{bmatrix} 1 & 0 & t_x \\ 0 & 1 & t_y \end{bmatrix}
   $$

   * $t_x$: Desplazamiento horizontal en píxeles.

   * $t_y$: Desplazamiento vertical en píxeles.

2. **Rotación 2D:**

   ```
   M = cv2.getRotationMatrix2D(center, angle, scale)
   
   ```

   * **`center`**: Punto de giro `(x, y)`.

   * **`angle`**: Ángulo en grados (valores positivos rotan en sentido antihorario; negativos en sentido horario).

   * **`scale`**: Factor de escala (1.0 mantiene el tamaño original).

3. **Matriz Afín a partir de Puntos (Cizallamiento / Inclinación):**

   ```
   M = cv2.getAffineTransform(pts1, pts2)
   
   ```

   * **`pts1`**: Matriz `np.float32` de $3$ puntos de origen.

   * **`pts2`**: Matriz `np.float32` de $3$ puntos de destino.

## 2. Transformaciones Proyectivas (Perspectiva / Homografía)

Se utilizan para corregir distorsiones de perspectiva (por ejemplo, alineación de documentos fotografiados desde un ángulo). Operan con matrices de $3 \times 3$.

```
# 1. Obtener matriz de perspectiva (3x3) a partir de 4 puntos
M = cv2.getPerspectiveTransform(pts1, pts2)

# 2. Aplicar la transformación de perspectiva
dst = cv2.warpPerspective(src, M, dsize)

```

* **`pts1`**: Matriz `np.float32` con $4$ puntos de origen (vértices del polígono distorsionado).

* **`pts2`**: Matriz `np.float32` con $4$ puntos de destino (vértices del rectángulo plano final).

* **`dsize`**: `(ancho_doc, alto_doc)` correspondiente a las dimensiones finales deseada.

## 3. Relleno y Gestión de Bordes (`borderMode`)

Determina el comportamiento de los píxeles situados en las fronteras durante una transformación:

* **`cv2.BORDER_CONSTANT`**: Aplica un valor de color fijo especificado mediante `borderValue`.

* **`cv2.BORDER_REPLICATE`**: Duplica los píxeles del borde exterior hacia las zonas vacías.

## 4. Manipulación y Concatenación de Matrices

* **Reflexión / Volteado:**

  ```
  dst = cv2.flip(src, flipCode)
  
  ```

  * `flipCode = 0`: Volteado vertical (eje $X$).

  * `flipCode > 0`: Volteado horizontal (eje $Y$).

  * `flipCode < 0`: Volteado simultáneo en ambos ejes.

* **Concatenación:**

  ```
  v_concat = cv2.vconcat([img1, img2]) # Concatenación vertical
  h_concat = cv2.hconcat([img1, img2]) # Concatenación horizontal
  
  ```

* **Recorte de Región de Interés (ROI):**
  A través de *slicing* sobre la matriz de NumPy:

  ```
  roi = img[y_min:y_max, x_min:x_max]
  
  ```

## 5. Escalado e Interpolación de Imágenes

La función `cv2.resize` escala las dimensiones de una imagen mediante distintos métodos numéricos:

```
dst = cv2.resize(src, dsize=(ancho, alto), fx=scale_x, fy=scale_y, interpolation=metodo)

```

| Método de Interpolación | Descripción / Comportamiento | Uso Recomendado | 
| ----- | ----- | ----- | 
| **`cv2.INTER_NEAREST`** | Asigna el valor del vecino más cercano. Produce un efecto pixelado (*aliasing*). | Operaciones donde la velocidad es crítica o en imágenes de bajo píxel. | 
| **`cv2.INTER_LINEAR`** | Interpolación bilineal ($2 \times 2$ píxeles). Algoritmo rápido con suavizado básico. | Método por defecto en OpenCV. Buen equilibrio general. | 
| **`cv2.INTER_CUBIC`** | Interpolación bicúbica ($4 \times 4$ píxeles). Genera contornos más nítidos. | Ampliación y escalado de imágenes manteniendo detalle. | 
| **`cv2.INTER_LANCZOS4`** | Interpolación de Lanczos ($8 \times 8$ píxeles). Máximo nivel de detalle en frecuencias altas. | Escalado de alta calidad donde se requiere máxima definición en bordes. | 

## 💡 Aspectos Clave para Evaluación

1. **Diferencia de Entidades Geométricas:**

   * **Afín (**$2 \times 3$**):** Requiere **3 puntos** (`cv2.getAffineTransform`). Conserva el paralelismo.

   * **Perspectiva (**$3 \times 3$**):** Requiere **4 puntos** (`cv2.getPerspectiveTransform`). Corrige la convergencia de líneas paralelas.

2. **Convención de Coordenadas:**

   * **OpenCV (`dsize`, puntos):** Utiliza el sistema $(x, y) \rightarrow (\text{ancho}, \text{alto})$.

   * **NumPy (`shape`, *slicing*):** Utiliza el sistema $(\text{filas}, \text{columnas}) \rightarrow [y, x]$.

3. **Representación de Color:** OpenCV lee las imágenes por defecto en formato **BGR**, por lo que es necesario convertirlas a **RGB** (`cv2.cvtColor(img, cv2.COLOR_BGR2RGB)`) para su correcta visualización en bibliotecas como `matplotlib`.