<h1>Paint:</h1>

<p>El siguiente proyecto consiste en la recreación de un paint sencillo codificado en P5*js. En este paint se podrá: cambiar el grosor del pincel; cambiar el color del pincel; cambiar el color de fondo; limpiar el último trazo dado en el lienzo; limpiar por completo el lienzo.</p>


- Link a la aplicación: [Paint](https://editor.p5js.org/juanse77/present/hBz8EHCeR). 



<h2>Detalles de implementación:</h2>

<p>Para la interacción del usuario se ha creado dos componentes. Uno para la selección de los colores, tanto del pincel como del fondo, y el otro para la elección del grosor del pincel.</p>

<p>El componente para la selección del color se ha diseñado como una rejilla de 2x8. La primera fila está compuesta por los colores saturados primarios y sus combinaciones 2 a 2. La segunda fila también combina las componentes primarias de los colores 2 a 2 pero sin saturar. Esta rejilla servirá tanto para seleccionar el color del pincel como para seleccionar el color del fondo. Para que esto sea posible se ha de pulsar la tecla (c) la cual activará alternativamente el modo de color de fondo o de pincel. En cada modo se presentará una etiqueta distinta indicando el modo actual y se visualizará debajo de la etiquea un recuadro con el color de elemento activado en ese momento.</p>

<p>Para el componente de elección del grosor del pincel se ha utilizado un diseño de barra deslizante con un punto centrado en el valor de grosor del pincel activado en ese momento y con un color de relleno igual al color del pincel que se encuentre seleccionado.</p>

<h3>Elección del color del pincel y del fondo:</h3>

<p>El color del pincel o del fondo se selecciona haciendo click en alguno de los cuadrados de color de la rejilla. La lógica para esta funcionalidad se centra en el evento mouseClicked. Los colores que se visulizan en la rejilla están almacenados en el array 'colores' y los colores activados en cada momento se guardan en las variables 'indexColorP' y 'indexColorBG', que almacenarán el valor del índice que apunte al color dentro del array 'colores'.</p>

```java
[...]

if (mouseX > 110 && mouseX < 385 && mouseY > 15 && mouseY < 80) {
  let posX = int((mouseX - 110) / 34.375);
  let posY = int((mouseY - 15) / 32.5);
  if (modoPincel) {
    indexColorP = posY * 8 + posX;
  } else {
    indexColorBG = posY * 8 + posX;
  }
}

[...]
```

<h3>Elección del grosor del pincel:</h3>

<p>La lógica para esta funcionalidad se encuentra, como en el caso anterior, en el evento mouseClicked. Cuando la pulsación del ratón se detecte dentro del rectángulo de la barra deslizante se calculará el valor entero más próximo a la posición en la que se generó el evento. Este valor se almacenará en la variable 'valGrosor'.</p>

```java
[...]

if (mouseX > 415 && mouseX < 785 && mouseY > 40 && mouseY < 80) {
  valGrosor = round(9 * (mouseX - 415) / (785 - 415)) + 1;
}

[...]
```

<h3>El proceso de pintado del lienzo</h3>

<p>Esta vez la lógica de la funcionalidad se reparte entre dos eventos de ratón: mouseDragged, mouseReleased; y el método 'dibujaLienzo' que se llamará desde el método 'draw' y que será el que utilice los valores recogidos por los eventos del ratón.</p>

<p>MouseDragged se encarga de ir almacenando los puntos del trazo en un array dinámico llamado 'trazo'. Cada punto del trazo contendrá la información del punto en el que se detectó, el grosor y el color del pincel en el momento de la detección, y el valor de un contador llamado 'numTrazo' que se utilizará para la funcionalidad de borrar el último trazo de los que se vayan apilado.</p>

<p>También se utilizarán las variables 'numSubTrazos' y 'numPuntosNuevos'. 'numSubTrazos' se usará para conocer el tamaño de array de puntos de la variable 'trazo'. La variable 'numPuntosNuevos' se usará para detectar si, en el intervalo en el que se presionó el botón del ratón y se liberó, se añadieron nuevos puntos al array 'trazo'. Si no se añadió ningún punto se considera que no se ha realizado ningún trazo por lo que no es necesario avanzar el contador 'numTrazo'.</p>

<p>Hay que resaltar también que para que se considere un punto como capturable debe mantener una distancia de más de un pixel con respecto al anterior punto detectado por el ratón.</p>

```java
function mouseDragged() {

  if (mouseY > 95) {
    let distancia = abs(winMouseX - pwinMouseX);

    if (distancia > 1) {
      append(trazo, [mouseX, mouseY, valGrosor * 2, indexColorP, numTrazo]);
      numSubTrazos++;
      numPuntosNuevos++;
    }
  }
}

function mouseReleased() {
  if (numPuntosNuevos > 0) {
    numTrazo++;
    numPuntosNuevos = 0;
  }
}
```

<p>El método 'dibujaLienzo' se encarga de dibujar los trazos en el lienzo. Este método tiene que dibujar los trazos separados unos de otros, para lo cual no se puede únicamente unir todos los puntos del array, sino que es preciso detectar cuándo se pasa de dibujar un trazo al siguiente y establecer entre ellos una separación. Para lo cual se utiliza la variable local 'pNumTrazo' que será la que se utilizará para detectar cuándo cambia el valor de 'numTrazo' dentro del array 'trazo'.</p>

```java
function dibujaLienzo() {

  fill(colores[indexColorBG]);
  rect(0, 95, 800, 705);

  let pNumTrazo = 0;
  for (let i = 0; i < numSubTrazos - 1; i++) {
    stroke(colores[trazo[i][3]]);
    strokeWeight(trazo[i][2]);
    if (pNumTrazo != trazo[i+1][4]) {
      pNumTrazo = trazo[i+1][4];
    } else {
      line(trazo[i][0], trazo[i][1], trazo[i + 1][0], trazo[i+1][1]);
    }
  }

  strokeWeight(1);
  stroke(0);
}
```

<h3>Borrado del último trazo:</h3>

<p>Esta funcionalidad se activa al pulsar la tecla (d). Su lógica se desarrolla en el método 'borraTrazo'. Para detectar el trazo que se debe borrar se irá recorriendo inversamente el array 'trazo' hasta que se encuentre un valor de 'numTrazo' distinto. Cuando esto suceda se tomará como nuevo array de trazos al subarray de 'trazo' sin los puntos superiores a la posición calculada.</p>

```java
function borraTrazo() {
  if (numSubTrazos > 0) {
    let i = numSubTrazos - 1;
    let aux_numTrazo = trazo[i][4];
    while (i > 0 && trazo[i][4] == aux_numTrazo) {
      i--;
    }

    if (i == 0) {
      trazo = [];
      numSubTrazos = 0;
      numTrazo = 0;
    } else {
      trazo = subset(trazo, 0, i + 1);
      numSubTrazos = i + 1;
      numTrazo -= 1;
    }
  }
}
```

<p>Esta aplicación se ha desarrollado como octava práctica evaluable para la asignatura de "Creando Interfaces de Usuarios" de la mención de Computación del grado de Ingeniería Informática de la Universidad de Las Palmas de Gran Canaria en el curso 2019/20 y en fecha de 2/4/2020 por el alumno Juan Sebastián Ramírez Artiles.</p>

<p>Referencias a los recursos utilizados:</p>

- Modesto Fernando Castrillón Santana, José Daniel Hernández Sosa: [Creando Interfaces de Usuario. Guion de Prácticas](https://cv-aep.ulpgc.es/cv/ulpgctp20/pluginfile.php/126724/mod_resource/content/25/CIU_Pr_cticas.pdf).
- [Página de inicio del framework P5.js](https://p5js.org/).
- [Referencia de la librería P5.js](https://p5js.org/reference/). 
