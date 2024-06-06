## Phaser de Rebote

### Introducción 
La programación de videojuegos siempre ha sido tema de tiempos ya que siempre el desarrollo puede ser tardado, para eso existen plataformas como phaser que nos ayudan con sus librerías y archivos disponibles podemos hacer un videojuego bastante rápido.

### Objetivo
Con las herramientas dadas del anterior proyecto desarrollar un videjuego que sea capaz de esquivar una sola bala, esta bala tendrá un rebote aleatorio alrededor del personaje

### Desarrollo

Inicialización del Juego
Configura el tamaño de la pantalla, los objetos principales y las variables necesarias para la lógica del juego.

```
var ancho = 600;
var alto = 423;
var personaje;
var fondoJuego;
var proyectil;
var teclas;
var menuPausa;
var moverArriba;
var moverAbajo;
var moverIzquierda;
var moverDerecha;
var quieto;
var posicionProyectilX;
var posicionProyectilY;
var velocidadProyectilX;
var velocidadProyectilY;

var redNeuronal, entrenadorNeuronal, salidaNeuronal, datosEntrenamiento = [];
var modoAutomatico = false, entrenamientoCompleto = false;

var juego = new Phaser.Game(ancho, alto, Phaser.CANVAS, '', { preload: preload, create: create, update: update, render: render });
```
Preload
La función preload carga todos los recursos necesarios para el juego, como imágenes y sprites.

```
function preload() {
    juego.load.image('fondoJuego', 'assets/game/galaxia.jpg');
    juego.load.spritesheet('personaje', 'assets/sprites/altair.png', 32, 48);
    juego.load.image('menu', 'assets/game/menu.png');
    juego.load.image('proyectil', 'assets/sprites/purple_ball.png');
}
```

Create
La función create inicializa los objetos del juego, configura la física y establece las animaciones y controles del personaje.
```
function create() {
    juego.physics.startSystem(Phaser.Physics.ARCADE);
    juego.physics.arcade.gravity.y = 0; 
    juego.time.desiredFps = 30;

    fondoJuego = juego.add.tileSprite(0, 0, ancho, alto, 'fondoJuego');
    personaje = juego.add.sprite(ancho / 2, alto / 2, 'personaje');

    juego.physics.enable(personaje);
    personaje.body.collideWorldBounds = true; 
    var correrAnimacion = personaje.animations.add('correr', [8, 9, 10, 11]);
    personaje.animations.play('correr', 10, true);

    proyectil = juego.add.sprite(150, 240, 'proyectil');
    juego.physics.enable(proyectil);
    proyectil.body.collideWorldBounds = true; 
    proyectil.body.bounce.set(1); 
    resetearVelocidadProyectil();

    pausaTexto = juego.add.text(ancho - 100, 20, 'Pausa', { font: '20px Arial', fill: '#fff' });
    pausaTexto.inputEnabled = true;
    pausaTexto.events.onInputUp.add(pausarJuego, self);
    juego.input.onDown.add(manejarPausa, self);

    teclas = juego.input.keyboard.createCursorKeys();

    redNeuronal = new synaptic.Architect.Perceptron(3, 6, 6, 6, 5);
    entrenadorNeuronal = new synaptic.Trainer(redNeuronal);
}
```
Normalización de Datos
Una función auxiliar para normalizar los valores de entrada a la red neuronal.
```
function normalizarDatos(valor, maximo) {
    return valor / maximo;
}
```
Entrenamiento de la Red Neuronal
Esta función entrena la red neuronal con los datos recopilados durante el juego.
```
function entrenarRedNeuronal() {
    if (datosEntrenamiento.length > 0) {
        var resultados = entrenadorNeuronal.train(datosEntrenamiento, { rate: 0.01, iterations: 20000, shuffle: true });
        console.log("Resultados del entrenamiento:", resultados);
        entrenamientoCompleto = true;
    } else {
        console.log("No hay datos suficientes para entrenar la red neuronal.");
    }
}
```
Decisión de Movimiento
Esta función activa la red neuronal para decidir el movimiento del personaje basado en los datos de entrada.
```
function decidirMovimiento(parametrosEntrada) {
    salidaNeuronal = redNeuronal.activate(parametrosEntrada);

    var moverIzquierda = salidaNeuronal[0] > 0.5;
    var moverDerecha = salidaNeuronal[1] > 0.5;
    var moverArriba = salidaNeuronal[2] > 0.5;
    var moverAbajo = salidaNeuronal[3] > 0.5;
    var quieto = salidaNeuronal[4] > 0.5;

    return { moverIzquierda: moverIzquierda, moverDerecha: moverDerecha, moverArriba: moverArriba, moverAbajo: moverAbajo, quieto: quieto };
}
```
Pausa del Juego
La función pausarJuego pausa el juego y muestra un menú de pausa.
```
function pausarJuego() {
    juego.paused = true; 
    menuPausa = juego.add.sprite(ancho / 2, alto / 2, 'menu'); 
    menuPausa.anchor.setTo(0.5, 0.5);
}
```

Manejo de la Pausa
Esta función maneja los eventos de pausa, incluyendo el entrenamiento de la red neuronal y el cambio al modo automático.
```
function manejarPausa(evento) {
    if (juego.paused) {
        var menuX1 = ancho / 2 - 270 / 2, menuX2 = ancho / 2 + 270 / 2,
            menuY1 = alto / 2 - 180 / 2, menuY2 = alto / 2 + 180 / 2;

        var ratonX = evento.x,
            ratonY = evento.y;

        if (ratonX > menuX1 && ratonX < menuX2 && ratonY > menuY1 && ratonY < menuY2) {
            if (ratonX >= menuX1 && ratonX <= menuX2 && ratonY >= menuY1 && ratonY <= menuY1 + 90) {
                entrenamientoCompleto = false;
                datosEntrenamiento = [];
                modoAutomatico = false;
            } else if (ratonX >= menuX1 && ratonX <= menuX2 && ratonY >= menuY1 + 90 && ratonY <= menuY2) {
                if (!entrenamientoCompleto) {
                    console.log("Entrenando con " + datosEntrenamiento.length + " valores");
                    entrenarRedNeuronal();
                    console.log("Listo para jugar automáticamente");
                    entrenamientoCompleto = true;
                }
                modoAutomatico = true;
            }
            menuPausa.destroy();
            reiniciarJuego(); 
            juego.paused = false;
        }
    }
}
```
Reinicio del Juego
Esta función reinicia la posición del personaje y del proyectil.
```
function reiniciarJuego() {
    personaje.x = ancho / 2;
    personaje.y = alto / 2;
    personaje.body.velocity.x = 0;
    personaje.body.velocity.y = 0;

    proyectil.x = 0;
    proyectil.y = 0;
    resetearVelocidadProyectil();
}
```

Resetear Velocidad del Proyectil
Esta función establece una nueva velocidad aleatoria para el proyectil.
```
function resetearVelocidadProyectil() {
    velocidadProyectilX = -1 * Math.floor(Math.random() * (300 - 500 + 1) + 500);
    velocidadProyectilY = -1 * Math.floor(Math.random() * (300 - 500 + 1) + 500);
    proyectil.body.velocity.x = velocidadProyectilX;
    proyectil.body.velocity.y = velocidadProyectilY;
}
```
Update
La función update se llama en cada frame del juego. Aquí se actualiza la lógica del juego, como la posición del personaje y del proyectil, y se decide el movimiento basado en la entrada del usuario o de la red neuronal.
```
function update() {
    fondoJuego.tilePosition.x -= 1; 

    personaje.body.velocity.x = 0;
    personaje.body.velocity.y = 0;

    moverArriba = 0;
    moverAbajo = 0;
    moverIzquierda = 0;
    moverDerecha = 0;
    quieto = 0;

    posicionProyectilX = proyectil.position.x - personaje.position.x;
    posicionProyectilY = proyectil.position.y - personaje.position.y;
    var distanciaEuclidiana = Math.sqrt(posicionProyectilX * posicionProyectilX + posicionProyectilY * posicionProyectilY);

    var entradaNormalizada = [
        normalizarDatos(posicionProyectilX, ancho),
        normalizarDatos(posicionProyectilY, alto),
        normalizarDatos(distanciaEuclidiana, Math.sqrt(ancho * ancho + alto * alto))
    ];

    if (!modoAutomatico && teclas.left.isDown) {
        personaje.body.velocity.x = -300;
        moverIzquierda = 1;
        console.log("Izquierda");
    } else if (!modoAutomatico && teclas.right.isDown) {
        personaje.body.velocity.x = 300;
        moverDerecha = 1;
        console.log("Derecha");
    }
```

### Conclusión
Phaser nos ayuda demasiado al desarrollo del videojuego, sin embargo el desarrollo de hacer una funcion que sea capaz de aprender a jugar en modo automatico conllevo más tiempo.
   
