## Phaser con 3 balas

### Introducción 
La plataforma Phaser nos ayuda a la creación de videojuegos de plataforma 2D, esto con la ayuda de librerias creadas por ellos mismos haciendonos más rápido el desarrollo de los juegos.

### Objetivo
Con la ayuda de la plataforma Phaser y con el código que se te proporciono se debe crear un vídeojuego que sea capaz de esquivar 3 balas y que aprenda con la ayuda de perceptron.

### Desarrollo
Inicialización del Juego

```
var w=800;
var h=400;
var jugador;
var fondo;
var bala, balaD=false, nave;
var bala2, balaD2=false, nave2;
var bala3, balaD3=false, nave3;
var salto;
var avanza;
var menu;

var juego = new Phaser.Game(w, h, Phaser.CANVAS, '', { preload: preload, create: create, update: update, render:render });
Preload
javascript
Copiar código
function preload() {
    juego.load.image('fondo', 'assets/game/fondo.jpg');
    juego.load.spritesheet('mono', 'assets/sprites/altair.png', 32, 48);
    juego.load.image('nave', 'assets/game/ufo.png');
    juego.load.image('bala', 'assets/sprites/purple_ball.png');
    juego.load.image('menu', 'assets/game/menu.png');
}
```

Create
```
function create() {
    juego.physics.startSystem(Phaser.Physics.ARCADE);
    juego.physics.arcade.gravity.y = 800;
    juego.time.desiredFps = 30;

    fondo = juego.add.tileSprite(0, 0, w, h, 'fondo');
    nave = juego.add.sprite(w-100, h-70, 'nave');
    bala = juego.add.sprite(w-100, h, 'bala');
    nave2 = juego.add.sprite(w-800, h-400, 'nave');
    bala2 = juego.add.sprite(w-760, h-380, 'bala');
    nave3 = juego.add.sprite(w-100, h-400, 'nave');
    bala3 = juego.add.sprite(w-100, h-370, 'bala');
    jugador = juego.add.sprite(50, h, 'mono');

    juego.physics.enable(jugador);
    jugador.body.collideWorldBounds = true;
    var corre = jugador.animations.add('corre',[8,9,10,11]);
    jugador.animations.play('corre', 10, true);

    juego.physics.enable(bala);
    bala.body.collideWorldBounds = true;
    juego.physics.enable(nave);
    nave.body.collideWorldBounds = true;
    juego.physics.enable(bala2);
    bala2.body.collideWorldBounds = true;
    juego.physics.enable(bala3);
    bala3.body.collideWorldBounds = true;

    pausaL = juego.add.text(w - 100, 20, 'Pausa', { font: '20px Arial', fill: '#fff' });
    pausaL.inputEnabled = true;
    pausaL.events.onInputUp.add(pausa, self);
    juego.input.onDown.add(mPausa, self);

    salto = juego.input.keyboard.addKey(Phaser.Keyboard.SPACEBAR);
    avanza = juego.input.keyboard.addKey(Phaser.Keyboard.RIGHT);
    
    nnNetwork = new synaptic.Architect.Perceptron(6, 5, 5, 5, 4);
    nnEntrenamiento = new synaptic.Trainer(nnNetwork);
}
```

Update
```
function update() {
    fondo.tilePosition.x -= 1; 

    juego.physics.arcade.collide(nave, jugador, colisionH, null, this);
    juego.physics.arcade.collide(bala, jugador, colisionH, null, this);
    juego.physics.arcade.collide(bala2, jugador, colisionH, null, this);
    juego.physics.arcade.collide(bala3, jugador, colisionH, null, this);

    estatuSuelo = 1;
    estatusAire = 0;
    estatusDerecha = 0;
    estatusIzquierda = 1;

    if(!jugador.body.onFloor() || jugador.body.velocity.y !=0) {
        estatuSuelo = 0;
        estatusAire = 1;
        console.log("ARRIBA!");
    }
    
    if(jugador.body.velocity.x >= 140) {
        estatusDerecha = 1;
        estatusIzquierda = 0;
        console.log("DERECHA!");
    }
    
    despBala = Math.floor(jugador.position.x - bala.position.x);
    despBala2 = Math.floor(jugador.position.x - bala2.position.x);
    despBalaHorizontal3 = Math.floor(jugador.position.x - bala3.position.x);
    despBalaVertical3 = Math.floor(jugador.position.y - bala3.position.y);
    despBala3 = Math.floor(despBalaHorizontal3 + despBalaVertical3);

    if (modoAuto == false && salto.isDown && !estatusAire) {
        if(jugador.body.velocity.x <= 0) {
            jugador.body.velocity.x = 150;
            saltar();
            correr();
        } else { 
            saltar();
            Detenerse();
        }
    }
    if (modoAuto == false && avanza.isDown && jugador.body.onFloor()) {
        correr();
    }

    if (modoAuto == false && jugador.body.onFloor() && jugador.position.x >= 200) {
        correrAtras();
    }

    if (modoAuto == false && !avanza.isDown && jugador.body.onFloor() && jugador.position.x == 50) {
        Detenerse();
    }

    console.log(jugador.position.x);

    if (modoAuto == true && bala.position.x > 0 && jugador.body.onFloor()) {
        if (EntrenamientoSalto([despBala , velocidadBala, despBala2, velocidadBala2, despBala3, velocidadBala3])) {
            console.log("SALTAR");
            if(jugador.body.velocity.x <= 0) {
                jugador.body.velocity.x = 150;
                saltar();
                correr();
            } else { 
                saltar();
                Detenerse();
            }
        } 
        if (datosDeEntrenamiento([despBala , velocidadBala, despBala2, velocidadBala2, despBala3, velocidadBala3])) {
            console.log("CORRER");
            correr();         
        } else if (jugador.body.onFloor() && jugador.position.x >= 250) {
            Detenerse();
            correrAtras();
        }
    }

    if (balaD == false) {
        disparo();
    }

    if (balaD2 == false) {
        disparo2();
    }

    if (balaD3 == false) {
        disparo3();
    }

    if (bala.position.x <= 0) {
        resetVariables();
    }

    if (bala2.position.y >= 355) {
        resetVariables2();
    }

    if (bala3.position.x <= 0 || bala3.position.y >= 355) {
        resetVariables3();
    }

    if (modoAuto == false && bala.position.x > 0) {
        datosEntrenamiento.push({
            'input': [despBala, velocidadBala, despBala2, velocidadBala2, despBala3, velocidadBala3],
            'output': [estatusAire, estatuSuelo, estatusDerecha, estatusIzquierda]
        });
    }
}
```

Funciones Auxiliares
Disparar
```
function disparo() {
    velocidadBala = -1 * velocidadRandom(300,700);
    bala.body.velocity.y = 0;
    bala.body.velocity.x = velocidadBala;
    balaD = true;
}

function disparo2() {
    velocidadBala2 = -1 * velocidadRandom(300,800);
    bala2.body.velocity.y = 0;
    balaD2 = true;
}

function disparo3() {
    velocidadBala3 = -1 * velocidadRandom(400,500);
    bala3.body.velocity.y = 0;
    bala3.body.velocity.x = 1.60 * velocidadBala3;
    balaD3 = true;
}
```

Reset Variables
```
function disparo() {
    velocidadBala = -1 * velocidadRandom(300,700);
    bala.body.velocity.y = 0;
    bala.body.velocity.x = velocidadBala;
    balaD = true;
}

function disparo2() {
    velocidadBala2 = -1 * velocidadRandom(300,800);
    bala2.body.velocity.y = 0;
    balaD2 = true;
}

function disparo3() {
    velocidadBala3 = -1 * velocidadRandom(400,500);
    bala3.body.velocity.y = 0;
    bala3.body.velocity.x = 1.60 * velocidadBala3;
    balaD3 = true;
}
```

Movimiento del Jugador

```
function saltar() {
    jugador.body.velocity.y = -270;
}

function correr() {
    jugador.body.velocity.x = 150;
}

function correrAtras() {
    jugador.body.velocity.x = -150;
}

function Detenerse() {
    jugador.body.velocity.x = 0;
}
```

Neural Network
```
function enRedNeural() {
    nnEntrenamiento.train(datosEntrenamiento, { rate: 0.0003, iterations: 10000, shuffle: true });
}

function datosDeEntrenamiento(param_entrada) {
    nnSalida = nnNetwork.activate(param_entrada);
    return nnSalida[2] >= nnSalida[3];
}

function EntrenamientoSalto(param_entrada) {
    nnSalida = nnNetwork.activate(param_entrada);
    return nnSalida[0] >= nnSalida[1];
}
```

Pausa
```
function pausa() {
    juego.paused = true;
    menu = juego.add.sprite(w/2, h/2, 'menu');
    menu.anchor.setTo(0.5, 0.5);
}

function mPausa(event) {
    if (juego.paused) {
        var menu_x1 = w/2 - 270/2, menu_x2 = w/2 + 270/2,
            menu_y1 = h/2 - 180/2, menu_y2 = h/2 + 180/2;

        var mouse_x = event.x, mouse_y = event.y;

        if (mouse_x > menu_x1 && mouse_x < menu_x2 && mouse_y > menu_y1 && mouse_y < menu_y2) {
            if (mouse_x >= menu_x1 && mouse_x <= menu_x2 && mouse_y >= menu_y1 && mouse_y <= menu_y1 + 90) {
                eCompleto = false;
                datosEntrenamiento = [];
                modoAuto = false;
            } else if (mouse_x >= menu_x1 && mouse_x <= menu_x2 && mouse_y >= menu_y1 + 90 && mouse_y <= menu_y2) {
                if (!eCompleto) {
                    enRedNeural();
                    eCompleto = true;
                }
                modoAuto = true;
            }

            resetVariables();
            resetVariables2();
            resetVariables3();
            resetPlayer();
            juego.paused = false;
            menu.destroy();
        }
    }
}
```

### Conclusión 
Las herramientas de creación de videojuegos nos ayudan a poder avanzar exponencialmente en el desarrollo y sumado a esto si sabemos desarrollar funciones de aprendizaje automatico se pueden crear una creación como estas. 
