## Detección de catastrofes

### Introducción
CNN quiere decir: Convolutional Neural Networks, gracias es este tipo de herramientas podemos aprender a hacer modelos con la capacidas de reconocer diferentes situaciones, nos puede ayudar a detectar robos, incendios con esto incluso podemos llegar a predecir estas situaciones

### Desarrollo
Con la ayuda del archivo red convusional se debe de realizar una red que sea capaz de detectar las siguientes situaciones: 
1. Robo
2. Robo a casa-habitación
3. Inundación
4. Incendios
5. Tornados


Las librerias que necesitamos:
```
<--import numpy as np
import os
import re
import matplotlib.pyplot as plt
%matplotlib inline
from sklearn.model_selection import train_test_split
from sklearn.metrics import classification_report
import keras
import tensorflow as tf
from tensorflow.keras.utils import to_categorical
from keras.models import Sequential,Model
from tensorflow.keras.layers import Input
from keras.layers import Dense, Dropout, Flatten
#from keras.layers import Conv2D, MaxPooling2D
#from tensorflow.keras.models import Sequential
from tensorflow.keras.layers import (
    BatchNormalization, SeparableConv2D, MaxPooling2D, Activation, Flatten, Dropout, Dense, Conv2D
)
from keras.layers import LeakyReLU-->

```


Cargar las imagenes
```
dirname = os.path.join(os.getcwd(),'C:\\Users\\edgar\\CNN\\proyectovideo\\resultados\\resul') #cambiar ruta
imgpath = dirname + os.sep 

images = []
directories = []
dircount = []
prevRoot=''
cant=0

print("leyendo imagenes de ",imgpath)

for root, dirnames, filenames in os.walk(imgpath):
    for filename in filenames:
        if re.search("\.(jpg|jpeg|png|bmp|tiff)$", filename):
            cant=cant+1
            filepath = os.path.join(root, filename)
            image = plt.imread(filepath)
            if(len(image.shape)==3):
                
                images.append(image)
            b = "Leyendo..." + str(cant)
            print (b, end="\r")
            if prevRoot !=root:
                print(root, cant)
                prevRoot=root
                directories.append(root)
                dircount.append(cant)
                cant=0
dircount.append(cant)

dircount = dircount[1:]
dircount[0]=dircount[0]+1
print('Directorios leidos:',len(directories))
print("Imagenes en cada directorio", dircount)
print('suma Total de imagenes en subdirs:',sum(dircount))
```


Creación de etiquetas
```
labels=[]
indice=0
for cantidad in dircount:
    for i in range(cantidad):
        labels.append(indice)
    indice=indice+1
print("Cantidad etiquetas creadas: ",len(labels))

desastres=[]
indice=0
for directorio in directories:
    name = directorio.split(os.sep)
    print(indice , name[len(name)-1])
    desastres.append(name[len(name)-1])
    indice=indice+1
```

Creación de sets, entrenamiento de estos y sus test
```
train_X,test_X,train_Y,test_Y = train_test_split(X,y,test_size=0.2)
print('Training data shape : ', train_X.shape, train_Y.shape)
print('Testing data shape : ', test_X.shape, test_Y.shape)
plt.figure(figsize=[5,5])

# Display the first image in training data
plt.subplot(121)
plt.imshow(train_X[0,:,:], cmap='gray')
plt.title("Ground Truth : {}".format(train_Y[0]))

# Display the first image in testing data
plt.subplot(122)
plt.imshow(test_X[0,:,:], cmap='gray')
plt.title("Ground Truth : {}".format(test_Y[0]))
```

Preprocesamiento de imagenes
```
train_X = train_X.astype('float32')
test_X = test_X.astype('float32')
train_X = train_X/255.
test_X = test_X/255.
plt.imshow(test_X[0,:,:])
```

Hacemos el One-hot Encoding para la red
```
# Change the labels from categorical to one-hot encoding
train_Y_one_hot = to_categorical(train_Y)
test_Y_one_hot = to_categorical(test_Y)

# Display the change for category label using one-hot encoding
print('Original label:', train_Y[0])
print('After conversion to one-hot:', train_Y_one_hot[0])
```

Creamos el Set de Entrenamiento y Validación

```
#Mezclar todo y crear los grupos de entrenamiento y testing
train_X,valid_X,train_label,valid_label = train_test_split(train_X, train_Y_one_hot, test_size=0.2, random_state=13)
print(train_X.shape,valid_X.shape,train_label.shape,valid_label.shape)
```

Creamos el modelo de CNN
```
#declaramos variables con los parámetros de configuración de la red
INIT_LR = 1e-3 # Valor inicial de learning rate. El valor 1e-3 corresponde con 0.001
epochs = 20 # Cantidad de iteraciones completas al conjunto de imagenes de entrenamiento
batch_size = 64 # cantidad de imágenes que se toman a la vez en memoria
disaster_model = Sequential()
disaster_model.add(Conv2D(32, kernel_size=(3, 3),activation='linear',padding='same',input_shape=(28,28,3)))
disaster_model.add(LeakyReLU(alpha=0.1))
disaster_model.add(MaxPooling2D((2, 2),padding='same'))
disaster_model.add(Dropout(0.5))


disaster_model.add(Flatten())
disaster_model.add(Dense(32, activation='linear'))
disaster_model.add(LeakyReLU(alpha=0.1))
disaster_model.add(Dropout(0.5))
disaster_model.add(Dense(nClasses, activation='softmax'))
disaster_model.summary()
#disaster_model.compile(loss=keras.losses.categorical_crossentropy, optimizer=tf.keras.optimizers.legacy.SGD(learning_rate=INIT_LR, decay=INIT_LR / 100),metrics=['accuracy'])
disaster_model.compile(loss=keras.losses.categorical_crossentropy, optimizer=tf.keras.optimizers.SGD(learning_rate=INIT_LR), metrics=['accuracy'])
```

Entrenamos el modelo: Aprende a clasificar imágenes
```
# este paso puede tomar varios minutos, dependiendo de tu ordenador, cpu y memoria ram libre
#sport_train = disaster_model.fit(train_X, train_label, batch_size=batch_size,epochs=epochs,verbose=1,validation_data=(valid_X, valid_label))
sport_train = disaster_model.fit(train_X, train_label, batch_size=batch_size,epochs=epochs,verbose=1,validation_data=(valid_X, valid_label))
# guardamos la red, para reutilizarla en el futuro, sin tener que volver a entrenar
disaster_model.save("C:\\Users\\edgar\\CNN\\proyectovideo\\red\\nuevo.h5")
```

Evaluamos la red 
```
test_eval = disaster_model.evaluate(test_X, test_Y_one_hot, verbose=1)
print('Test loss:', test_eval[0])
print('Test accuracy:', test_eval[1])
sport_train.history
accuracy = sport_train.history['accuracy']
val_accuracy = sport_train.history['val_accuracy']
loss = sport_train.history['loss']
val_loss = sport_train.history['val_loss']
epochs = range(len(accuracy))
plt.plot(epochs, accuracy, 'bo', label='Training accuracy')
plt.plot(epochs, val_accuracy, 'b', label='Validation accuracy')
plt.title('Training and validation accuracy')
plt.legend()
plt.figure()
plt.plot(epochs, loss, 'bo', label='Training loss')
plt.plot(epochs, val_loss, 'b', label='Validation loss')
plt.title('Training and validation loss')
plt.legend()
plt.show()
predicted_classes2 = disaster_model.predict(test_X)
predicted_classes=[]
for predicted_sport in predicted_classes2:
    predicted_classes.append(predicted_sport.tolist().index(max(predicted_sport)))
predicted_classes=np.array(predicted_classes)
predicted_classes.shape, test_Y.shape
```

Volvemos a aprender de los errores obtenidos
```
correct = np.where(predicted_classes==test_Y)[0]
print("Found %d correct labels" % len(correct))
for i, correct in enumerate(correct[0:9]):
    plt.subplot(3,3,i+1)
    plt.imshow(test_X[correct].reshape(28,28,3), cmap='gray', interpolation='none')
    plt.title("{}, {}".format(desastres[predicted_classes[correct]],
                                                    desastres[test_Y[correct]]))

    plt.tight_layout()
    incorrect = np.where(predicted_classes!=test_Y)[0]
print("Found %d incorrect labels" % len(incorrect))
for i, incorrect in enumerate(incorrect[0:9]):
    plt.subplot(3,3,i+1)
    plt.imshow(test_X[incorrect].reshape(28,28,3), cmap='gray', interpolation='none')
    plt.title("{}, {}".format(desastres[predicted_classes[incorrect]],
                                                    desastres[test_Y[incorrect]]))
    plt.tight_layout()
```

Resultados de las clases y sus precisiones
```
target_names = ["Class {}".format(i) for i in range(nClasses)]
print(classification_report(test_Y, predicted_classes, target_names=target_names))
```

Como corremos la imagen:
```
import numpy as np
from skimage.transform import resize
import matplotlib.pyplot as plt
import cv2 as cv

images = []

# AQUI ESPECIFICAMOS UNAS IMAGENES
filenames = [r'C:\Users\edgar\tornade.jpeg']

for filepath in filenames:
    image = plt.imread(filepath, 0)
    image_resized = resize(image, (28, 28), anti_aliasing=True, clip=False, preserve_range=True)
    images.append(image_resized)

X = np.array(images, dtype=np.uint8)  # Convierto de lista a numpy
test_X = X.astype('float32')
test_X = test_X / 255.

# Asumiendo que disaster_model y desastres están definidos
predicted_classes = disaster_model.predict(test_X)

# Especificar nuevas dimensiones (ancho, alto)
nuevo_ancho = 600
nuevo_alto = 400
dimensiones = (nuevo_ancho, nuevo_alto)

# Configuraciones de posición y fuente
escala_fuente = 1
color_texto = (0, 0, 0)  # Negro
color_fondo = (0, 0, 255)  # Rojo (OpenCV usa formato BGR)
grosor_texto = 1
fuente = cv.FONT_HERSHEY_DUPLEX

for i, img_tagged in enumerate(predicted_classes):
    print(filenames[i], desastres[img_tagged.tolist().index(max(img_tagged))])

    # Leer la imagen
    imagen = cv.imread(filenames[i])
    # Redimensionar la imagen
    imagen_redimensionada = cv.resize(imagen, dimensiones)

    # Especificar el texto y su posición
    texto = desastres[img_tagged.tolist().index(max(img_tagged))]

    # Obtener el tamaño del texto
    (tamaño_texto, _) = cv.getTextSize(texto, fuente, escala_fuente, grosor_texto)

    # Calcular coordenadas para el rectángulo de fondo
    posicion_inferior_izquierda = (dimensiones[0] // 2 - tamaño_texto[0] // 2, dimensiones[1] - tamaño_texto[1] - 10)
    posicion_superior_derecha = (dimensiones[0] // 2 + tamaño_texto[0] // 2, dimensiones[1] - 10)

    # Dibujar el rectángulo de fondo
    cv.rectangle(imagen_redimensionada, posicion_inferior_izquierda, posicion_superior_derecha, color_fondo, cv.FILLED)

    # Calcular la posición para el texto centrado
    posicion_texto = (dimensiones[0] // 2 - tamaño_texto[0] // 2, dimensiones[1] - 15)

    # Dibujar el texto sobre el rectángulo
    cv.putText(imagen_redimensionada, texto, posicion_texto, fuente, escala_fuente, color_texto, grosor_texto)

    # Mostrar la imagen con el texto
    cv.imshow('Imagen con Texto y Fondo', imagen_redimensionada)

    # Esperar a que se presione una tecla para cerrar la ventana
    cv.waitKey(0)
    cv.destroyAllWindows()
```

### Conclusión
La ayuda de herramientas como la anterior es importante para el desarrollo de nuevas herramientas que beneficien y optimizen procesos que son tardados y complejos.


