# Phaser con diferentes entradas

El juego tiene una diferencia muy radical porque ahora va a recibir balas de cada punto cardinal, esta
nueva caracteristica supone un cambio en las entradas y salidas en las tablas



## Entradas y Salidas

![](https://static7.depositphotos.com/1064545/762/i/950/depositphotos_7629911-stock-photo-supernatural-man-dodging-bullets.jpg){width='100px'}


Tenemos nuevos movimientos, arriba, abajo, izquierda, derecha, en diagonal recta, inversa, gracias a esto tenemos 8 nuevos movimientos. 

| Movimiento       | Valores | Valores | Valores |
|------------------|---------|---------|---------|
| Arriba           |         |         |         |
| Abajo            |         |         |         |
| Derecha          |         |         |         |
| Izquierda        |         |         |         |
| Derecha-Arriba   |         |         |         |
| Derecha-centro   |         |         |         |
| Izquierda-Abajo  |         |         |         |
| Izquierda-centro |         |         |         |

Ejemplo de lo que puede ser una tabla con valores reales
 
![](WhatsApp%20Image%202024-05-13%20at%2010.52.33%20AM.jpeg)

Como se ve tenemos valores acertados o similares al phaser orignal, sin embargo podemos llegar a tener errores a la hora de la medición, en este caso lo que afecta al aprendizaje puede llegar a verse reflejado en la por las etapas y multicapas que le agreguemos al código.
