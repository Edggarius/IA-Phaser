### Encontrar a Wally/Waldo

## Introducción
Wally/Waldo es un personaje muy conocido alrededor del mundo, la finalidad de su creación es el de jugar a encontrarlo dentro de diferentes imagenes en distintos escenarios, encontrar a W se volvió una actividad práctica para niños y así desarrollar sus habilidades cognitivas, desarrollar una herramienta que sea capaz de poder reconocerlo sería de gran ayuda para resolver en tiempo record algunas imagenes.

## Objetivo
Con la ayuda de imagenes positivas y negativas darle uso a la herramienta Cascade-Trainer-GUI para entrenamiento y posterior ejecución del archivo .xml que contiene todo el entrenamiento previo.


## Desarrollo 
El siguiente programa nos ayudara a poder realizar el entrenamiento, pero para poder entrenar nuestro .xml necesitamos tener imagenes para esto, tenemos que tener dos tipos, positivas y negativas. 




| Positiva | Negativa | 
|----------|----------|
| Imagenes donde aparezca Wally, puede ser de cuerpo completo, caras y formas   | Imagenes basura como arboles o cualquier cosa que no queramos que sea reconocida    | 



### Código

```
import numpy as np
import cv2 as cv

# Cargar el clasificador entrenado
rostro = cv.CascadeClassifier('C:\\Users\edgar\\classifier\\cascade.xml')

# Cargar la imagen de Wally
img = cv.imread('C:\\Users\\edgar\\classifier\wallyprueba.jpeg')

# Convertir la imagen a escala de grises
gray = cv.cvtColor(img, cv.COLOR_BGR2GRAY)

# Detectar rostros en la imagen
rostros = rostro.detectMultiScale(gray, 1.1, 40)

# Procesar cada rostro detectado
for (x, y, w, h) in rostros:
    # Dibujar un rectángulo verde alrededor del rostro detectado
    img = cv.rectangle(img, (x, y), (x + w, y + h), (0, 255, 0), 2)

# Mostrar la imagen con los rostros detectados y los rectángulos
cv.imshow('Wally', img)
cv.waitKey(0)
cv.destroyAllWindows()
```
## Explicación código 

```
for (x, y, w, h) in rostros:
    # Dibujar un rectángulo verde alrededor del rostro detectado
    img = cv.rectangle(img, (x, y), (x + w, y + h), (0, 255, 0), 2)

```
Para cada rostro detectado, se dibuja un rectángulo verde alrededor del área del rostro en la imagen original.

(x, y) son las coordenadas de la esquina superior izquierda del rectángulo.
(x + w, y + h) son las coordenadas de la esquina inferior derecha del rectángulo.
(0, 255, 0) especifica el color del rectángulo (verde en este caso).
2 es el grosor del rectángulo.

```
rostros = rostro.detectMultiScale(gray, 1.1, 40)
```
Se utiliza el método detectMultiScale del clasificador para detectar rostros en la imagen en escala de grises. Los parámetros 1.1 y 40 son el factor de escala y el número mínimo de vecinos respectivamente:

1.1: Especifica cuánto se reduce el tamaño de la imagen en cada escala de imagen.

40: Especifica cuántos vecinos deben tener cada ventana candidata para ser retenida.
```
rostro = cv.CascadeClassifier('C:\\Users\\edgar\\classifier\\cascade.xml')
```
Se carga el clasificador Haar Cascade previamente entrenado que está almacenado en un archivo XML (cascade.xml). Este clasificador es utilizado para detectar rostros en las imágenes.


## Conclusión
Aprender a hacer estas imagenes nos ayuda a poder tomar ventajas y a a retroalimentar diferentes situaciones, esto es importante porque siempre hay imagenes diferentes y con posibilidades diferentes por lo que ajustar los parametros es importante.
