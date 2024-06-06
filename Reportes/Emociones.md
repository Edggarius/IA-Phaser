### Emociones - Feliz, Enojado y Sorprendido

## Introducción 
Para el proyecto de reconocimiento de emociones se utilizó LBPH con un dataSet de un aproximado de 25 personas en distintas iluminaciones
FacialRecognize
mediante el archivo xml puedes reconocer ciertos patrones asociados a un rostro.

## Desarrollo
Empezamos con 
```
import cv2 as cv
cap = cv.VideoCapture(0)
while(True):
    ret, img = cap.read()
    if ret == True:
        img2 = cv.cvtColor(img, cv.COLOR_BGR2HSV)
        ubb=(100,0,0)
        uba=(110, 255, 255)
        mask = cv.inRange(img2, ubb, uba) 
        res = cv.bitwise_and(img,img, mask-mask)
        cv.imshow('img2', img)
        cv.imshow('res', res)
        cv.imshow('mask', mask)
        k =cv.waitKey(20) & 0xFF
        if k == 27 :
            break
cap.release()
cv.destroyAllWindows()
import numpy as np
import cv2 as cv
import math
rostro = cv.CascadeClassifier('C:\\Users\\edgar\\Emociones\\\haarcascade_frontalface_alt.xml')

cap = cv.VideoCapture(0) #Aqui se podría el link del video
while True:
    ret, frame = cap.read()
    gray = cv.cvtColor(frame, cv.COLOR_BGR2GRAY)
    rostros = rostro.detectMultiScale(gray, 1.3, 5)
    for(x, y, w, h) in rostros:
        frame = cv.rectangle(frame, (x,y), (x+w, y+h), (0, 255, 0), 2)
    cv.imshow('rostros', frame)
    k =cv.waitKey(1)
    if k == 27 :
        break
cap.release()
cv.destroyAllWindows()
```

Podemos guardar el segmentado del rostro encontrado, con esto crearemos nuestro propio DataSet; solo falta acondicionar el tamaño real del pixelaje.

Con la información (frame) ya segmenteado (frame 2) ahora debemos estandarizar el tamaño.

LBPH
Se genera el XML apuntando al directorio del dataSet

```

import cv2 as cv 
import numpy as np 
import os

dataSet = 'C:\\Users\\edgar\\emociones\\emociones'
faces  = os.listdir(dataSet)
print(faces)

labels = []
facesData = []
label = 0 
for face in faces:
    facePath = dataSet+'\\'+face
    for faceName in os.listdir(facePath):
        labels.append(label)
        facesData.append(cv.imread(facePath+'\\'+faceName,0))
    label = label + 1
#print(np.count_nonzero(np.array(labels)==0)) 
faceRecognizer = cv.face.LBPHFaceRecognizer_create()
faceRecognizer.train(facesData, np.array(labels))
faceRecognizer.write('C:\\Users\\edgar\\Emociones\\XML2\\EmocionesLBPHDrei.xml')
['enojo', 'feliz', 'sorpresa']
import cv2 as cv
import os 

faceRecognizer = cv.face.LBPHFaceRecognizer_create()
faceRecognizer.read('C:\\Users\\edgar\\Emociones\\XML2\\EmocionesLBPHDrei.xml')
dataSet = 'C:\\Users\\edgar\\Emociones\\emociones3'
faces  = os.listdir(dataSet)
cap = cv.VideoCapture(0)
rostro = cv.CascadeClassifier('C:\\Users\\edgar\\Emociones\\haarcascade_frontalface_alt.xml')
while True:
    ret, frame = cap.read()
    if ret == False: break
    gray = cv.cvtColor(frame, cv.COLOR_BGR2GRAY)
    cpGray = gray.copy()
    rostros = rostro.detectMultiScale(gray, 1.3, 3)
    for(x, y, w, h) in rostros:
        frame2 = cpGray[y:y+h, x:x+w]
        frame2 = cv.resize(frame2,  (100,100), interpolation=cv.INTER_CUBIC)
        result = faceRecognizer.predict(frame2)
        cv.putText(frame, '{}'.format(result), (x,y-20), 1,3.3, (255,255,0), 1, cv.LINE_AA)
        if result[1] < 100:
            cv.putText(frame,'{}'.format(faces[result[0]]),(x,y-25),2,1.1,(0,255,0),1,cv.LINE_AA)
            cv.rectangle(frame, (x,y),(x+w,y+h),(0,255,0),2)
        else:
            cv.putText(frame,'Desconocido',(x,y-20),2,0.8,(0,0,255),1,cv.LINE_AA)
            cv.rectangle(frame, (x,y),(x+w,y+h),(0,0,255),2) 
    cv.imshow('frame', frame)
    k = cv.waitKey(1)
    if k == 27:
        break
cap.release()
cv.destroyAllWindows()
```
Luego procederemos a leer el XML con el Haraascade y encendemos la cámara

```
import numpy as np
import cv2 as cv
import math
rostro = cv.CascadeClassifier('C:\\Users\\edgar\\Emociones\\haarcascade_frontalface_alt.xml')

cap = cv.VideoCapture(0)
i = 0
while True:
    ret, frame = cap.read()
    gray = cv.cvtColor(frame, cv.COLOR_BGR2GRAY)
    rostros = rostro.detectMultiScale(gray, 1.3, 5)
    for(x, y, w, h) in rostros:
        #frame = cv.rectangle(frame, (x,y), (x+w, y+h), (0, 255, 0), 2)
        frame2 = frame[y:y+h, x:x+w]
        frame2 = cv.resize(frame2, (100, 100), interpolation = cv.INTER_AREA)
        cv.imshow('rostros encontrados', frame2)
        cv.imwrite('C:\\Users\\edgar\\Emociones\\o\\Edgar'+str(i)+'.png', frame2)
    #cv.imshow('rostros', frame) 
    i=i+1
    k =cv.waitKey(1)
    if k == 27 :
        break
cap.release()
cv.destroyAllWindows()

```
En el resto de celdas están la manera de trabajarde con Eigenface y FisherFace, opciones que fueron descartadas.


EIGENFACE
Generar xml
```
import cv2 as cv 
import numpy as np 
import os

dataSet = 'C:\\Users\\edgar\\Emociones\\\\Emociones3'
faces  = os.listdir(dataSet)
print(faces)

labels = []
facesData = []
label = 0 
for face in faces:
    facePath = dataSet+'\\'+face
    for faceName in os.listdir(facePath):
        labels.append(label)
        facesData.append(cv.imread(facePath+'\\'+faceName,0))
    label = label + 1
print(np.count_nonzero(np.array(labels)==0))
faceRecognizer = cv.face.EigenFaceRecognizer_create()
faceRecognizer.train(facesData, np.array(labels))
faceRecognizer.write('C:\\Users\\edgar\\Emociones\\XML2\\Perrites2.xml')
```

A buscar rostros
```
import cv2 as cv
import os 

faceRecognizer = cv.face.EigenFaceRecognizer_create()
faceRecognizer.read('C:\\Users\\edgar\\Emociones\\\\Perrites2.xml')
dataSet = 'C:\\Users\\edgar\\Emociones\\Emociones3'
faces  = os.listdir(dataSet)
cap = cv.VideoCapture(0)
rostro = cv.CascadeClassifier('C:\\Users\\edgar\\Emociones\\haarcascade_frontalface_alt.xml')
while True:
    ret, frame = cap.read()
    if ret == False: break
    gray = cv.cvtColor(frame, cv.COLOR_BGR2GRAY)
    cpGray = gray.copy()
    rostros = rostro.detectMultiScale(gray, 1.3, 3)
    for(x, y, w, h) in rostros:
        frame2 = cpGray[y:y+h, x:x+w]
        frame2 = cv.resize(frame2,  (100,100), interpolation=cv.INTER_CUBIC)
        result = faceRecognizer.predict(frame2)
        #cv.putText(frame, '{}'.format(result), (x,y-20), 1,3.3, (0,0,0), 1, cv.LINE_AA)
        if result[1] > 2800:
            cv.putText(frame,'{}'.format(faces[result[0]]),(x,y-25),2,1.1,(0,255,0),1,cv.LINE_AA)
            cv.rectangle(frame, (x,y),(x+w,y+h),(0,255,0),2)
        else:
            cv.putText(frame,'Desconocido',(x,y-20),2,0.8,(0,0,255),1,cv.LINE_AA)
            cv.rectangle(frame, (x,y),(x+w,y+h),(0,0,255),2)
    cv.imshow('frame', frame)
    k = cv.waitKey(1)
    if k == 27:
        break
cap.release()
cv.destroyAllWindows()
```

FISHERFACE
Siempre hay que tener en cuenta el tamaño de la imagen y que no se mezcle la información

Puedes cambiar el nombre a los archivos, lo que importa es que estén contenidos dentro de una misma carpeta

Debemos crear un Fisher Face
```
import cv2 as cv 
import numpy as np 
import os

dataSet = 'D:\\MuestreoRostros'
faces  = os.listdir(dataSet)
print(faces)

labels = []
facesData = []
label = 0 
for face in faces:
    facePath = dataSet+'\\'+face
    for faceName in os.listdir(facePath):
        labels.append(label)
        facesData.append(cv.imread(facePath+'\\'+faceName,0))
    label = label + 1
#print(np.count_nonzero(np.array(labels)==0)) 
faceRecognizer = cv.face.FisherFaceRecognizer_create()
faceRecognizer.train(facesData, np.array(labels))
faceRecognizer.write('D:\\XML\\FisherFace\\EdgarFisherface.xml')
```

Así mismo, la lectura se hará con Fisher
Creamos el XML
```
import cv2 as cv
import os 

faceRecognizer = cv.face.FisherFaceRecognizer_create()
faceRecognizer.read('D:\\XML\\FisherFace\\EdgarFisherface.xml')
dataSet = 'D:\\MuestreoRostros'
faces  = os.listdir(dataSet)
cap = cv.VideoCapture(0)
rostro = cv.CascadeClassifier('C:\\Users\\edgar\\Emociones\\haarcascade_frontalface_alt.xml')
while True:
    ret, frame = cap.read()
    if ret == False: break
    gray = cv.cvtColor(frame, cv.COLOR_BGR2GRAY)
    cpGray = gray.copy()
    rostros = rostro.detectMultiScale(gray, 1.3, 3)
    for(x, y, w, h) in rostros:
        frame2 = cpGray[y:y+h, x:x+w]
        frame2 = cv.resize(frame2,  (100,100), interpolation=cv.INTER_CUBIC)
        result = faceRecognizer.predict(frame2)
        cv.putText(frame, '{}'.format(result), (x,y-20), 1,3.3, (255,255,0), 1, cv.LINE_AA)
        if result[1] < 500:
            cv.putText(frame,'{}'.format(faces[result[0]]),(x,y-25),2,1.1,(0,255,0),1,cv.LINE_AA)
            cv.rectangle(frame, (x,y),(x+w,y+h),(0,255,0),2)
        else:
            cv.putText(frame,'Desconocido',(x,y-20),2,0.8,(0,0,255),1,cv.LINE_AA)
            cv.rectangle(frame, (x,y),(x+w,y+h),(0,0,255),2)
    cv.imshow('frame', frame)
    k = cv.waitKey(1)
    if k == 27:
        break
cap.release()
cv.destroyAllWindows()
import cv2 as cv
import os 

faceRecognizer = cv.face.LBPHFaceRecognizer_create()
faceRecognizer.read('C:\\Users\\edgar\\Emociones\\XML2\\PersonesLBPH3.xml')
dataSet = 'C:\\Users\\edgar\\Emociones\\persones'
faces  = os.listdir(dataSet)
cap = cv.VideoCapture(0)
rostro = cv.CascadeClassifier('C:\\Users\\edgar\\Emociones\\haarcascade_frontalface_alt.xml')
while True:
    ret, frame = cap.read()
    if ret == False: break
    gray = cv.cvtColor(frame, cv.COLOR_BGR2GRAY)
    cpGray = gray.copy()
    rostros = rostro.detectMultiScale(gray, 1.3, 3)
    for(x, y, w, h) in rostros:
        frame2 = cpGray[y:y+h, x:x+w]
        frame2 = cv.resize(frame2,  (100,100), interpolation=cv.INTER_CUBIC)
        result = faceRecognizer.predict(frame2)
        cv.putText(frame, '{}'.format(result), (x,y-20), 1,3.3, (255,255,0), 1, cv.LINE_AA)
        if result[1] < 100:
            cv.putText(frame,'{}'.format(faces[result[0]]),(x,y-25),2,1.1,(0,255,0),1,cv.LINE_AA)
            cv.rectangle(frame, (x,y),(x+w,y+h),(0,255,0),2)
        else:
            cv.putText(frame,'Desconocido',(x,y-20),2,0.8,(0,0,255),1,cv.LINE_AA)
            cv.rectangle(frame, (x,y),(x+w,y+h),(0,0,255),2) 
    cv.imshow('frame', frame)
    k = cv.waitKey(1)
    if k == 27:
        break
cap.release()
cv.destroyAllWindows()
```

RESIZE

```
# Importar las bibliotecas necesarias
import numpy as np
import cv2 as cv
import os

# Definir la ruta a la carpeta con las imágenes
folder_path = 'D:\\ZWally\\Ejercicios'  # Reemplaza con la ruta correcta
# Definir la función para redimensionar imágenes
def resize_images(folder_path, size=(50, 50)):
    # Verificar si la carpeta existe
    if not os.path.exists(folder_path):
        print(f"La carpeta {folder_path} no existe.")
        return
    
    # Iterar sobre todos los archivos en la carpeta
    for filename in os.listdir(folder_path):
        if filename.endswith(('.png', '.jpg', '.jpeg', '.bmp', '.gif')):
            try:
                img_path = os.path.join(folder_path, filename)
                img = cv.imread(img_path)
                if img is not None:
                    # Redimensionar la imagen
                    img_resized = cv.resize(img, size, interpolation=cv.INTER_AREA)
                    # Sobrescribir la imagen original con la imagen redimensionada
                    cv.imwrite(img_path, img_resized)
                    print(f"Imagen {filename} redimensionada a {size}.")
                else:
                    print(f"No se pudo leer la imagen {filename}.")
            except Exception as e:
                print(f"No se pudo redimensionar la imagen {filename}: {e}")


```
### Llamar a la función para redimensionar imágenes
resize_images(folder_path)

### Conclusión
Con el debido entrenamiento podemos manejar situaciones como aprender a reconocer sentimientos o emociones, con las herramientas actuales cada vez se puede volver más fácil y práctico.
