<script>
  import * as d3 from "d3";
  import jugadores from "/src/data/bdoro.csv"; // Asegúrate de que los datos estén en el formato adecuado

  // Convertir los datos y asegurarse de que los tipos son correctos
  const data = jugadores.map(d => ({
    ...d,
    edad: +d.edad, // Convertir la edad a número
    votos: +d["% de votos"], // Convertir los votos a número
    continente: d.continente,
    posicion: d.posicion,
    ranking: +d.ranking, // Añadir el ranking a cada jugador
    radius: 0 // Inicializar el radio aquí para usar más tarde
  }));

  // Escala para el tamaño basado en los votos
  const minMaxVotos = d3.extent(data, (d) => d.votos);
  const grosorPartic = d3.scaleRadial()
    .domain(minMaxVotos)
    .range([40, 250]); // Tamaños de los círculos (en píxeles)

  // Escala para continentes (categórico > color)
  const colorContinente = d3.scaleOrdinal()
    .domain(["America", "Africa", "Europa"])
    .range(["#41b6c4", "#fe9929", "#78c679"]);

  // Escala para posiciones (categórico > color)
  const colorPosicion = d3.scaleOrdinal()
    .domain(["arquero", "defensor", "mediocampista", "delantero"])
    .range(["#30ff24", "#fe0000", "#ffff00", "#00ffff"]);

  // Asignar el radio a cada jugador
  data.forEach(d => {
    d.radius = grosorPartic(d.votos) / 2; // Usar la mitad del tamaño para el radio
  });

  // Función para determinar el grosor del borde basado en la edad
  function getBorderThickness(edad) {
    if (edad >= 20 && edad < 25) return 1;   // Grosor fino
    if (edad >= 25 && edad < 30) return 5;   // Grosor mediano
    if (edad >= 30 && edad <= 35) return 9;  // Grosor grueso
    return 1; // Grosor predeterminado para otras edades
  }

  // Crear un tooltip (cartel) oculto por defecto
  let tooltip;
  let container;

  // Función para centrar la simulación dentro del contenedor
  function centerSimulation() {
    const rect = container.getBoundingClientRect(); // Obtén las dimensiones del contenedor
    simulation.force("center", d3.forceCenter(rect.width / 2, rect.height / 2)); // Centra en el medio del contenedor
    simulation.alpha(1).restart(); // Reinicia la simulación para aplicar los cambios
  }

  // Configuración de la simulación
  const simulation = d3.forceSimulation(data)
    .force("charge", d3.forceManyBody().strength(50)) // Fuerza de repulsión
    .force("collide", d3.forceCollide(d => d.radius+7)) // Colisión
    .force("center", d3.forceCenter(0, 0)) // Valor temporal hasta obtener las dimensiones del contenedor
    .on("tick", ticked); // Llamar a la función ticked en cada tick

  // Función que se ejecuta cuando cambia el tamaño de la ventana
  window.addEventListener("resize", centerSimulation);

  // Esperar a que el contenedor esté disponible y luego centrar la simulación
  $: if (container) {
    centerSimulation(); // Llama a la función para centrar la simulación
  }

  // Función que actualiza las posiciones de los círculos
  function ticked() {
  const circles = d3.select(container).selectAll(".player-circle")
    .data(data);

  circles.join("div")
    .attr("class", "player-circle")
    .style("width", d => d.radius * 2 + "px")
    .style("height", d => d.radius * 2 + "px")
    .style("background-color", d => colorContinente(d.continente))
    .style("box-shadow", d => `0 0 0 ${getBorderThickness(d.edad)}px ${colorPosicion(d.posicion)}`)
    .style("border-radius", "50%")
    .style("position", "absolute")
    .style("left", d => (d.x - d.radius) + "px")
    .style("top", d => (d.y - d.radius) + "px")
    .attr("title", d => `${d.Nombre} - ${d.Pais}`)
        // Agregar un círculo dorado para Lionel Messi
  circles.each(function(d) {
      if (d.Nombre === "Lionel Messi") {
        d3.select(this)
          .append("div")
          .attr("class", "gold-circle")
          .style("width", d => (d.radius * 0.4) * 2 + "px") // Hacer el círculo dorado más pequeño (40% del radio)
          .style("height", d => (d.radius * 0.4) * 2 + "px") // Hacer el círculo dorado más pequeño (40% del radio)
          .style("background-color", "gold") // Color dorado
          .style("border-radius", "50%") // Hacerlo circular
          .style("position", "absolute")
          .style("left", "30%") // Centrar el círculo dorado dentro del círculo de Messi
          .style("top", "30%"); // Centrar el círculo dorado dentro del círculo de Messi
      }

      // Agregar un círculo plateado para Virgil Van Dijk
      if (d.Nombre === "Virgil Van Dijk") {
        d3.select(this)
          .append("div")
          .attr("class", "silver-circle")
          .style("width", d => (d.radius * 0.4) * 2 + "px") // Hacer el círculo plateado más pequeño (40% del radio)
          .style("height", d => (d.radius * 0.4) * 2 + "px") // Hacer el círculo plateado más pequeño (40% del radio)
          .style("background-color", "silver") // Color plateado
          .style("border-radius", "50%") // Hacerlo circular
          .style("position", "absolute")
          .style("left", "30%") // Centrar el círculo plateado dentro del círculo de Van Dijk
          .style("top", "30%"); // Centrar el círculo plateado dentro del círculo de Van Dijk
      }

      // Agregar un círculo de bronce para Cristiano Ronaldo
      if (d.Nombre === "Cristiano Ronaldo") {
        d3.select(this)
          .append("div")
          .attr("class", "bronze-circle")
          .style("width", d => (d.radius * 0.4) * 2 + "px") // Hacer el círculo de bronce más pequeño (40% del radio)
          .style("height", d => (d.radius * 0.4) * 2 + "px") // Hacer el círculo de bronce más pequeño (40% del radio)
          .style("background-color", "#cd7f32") // Color de bronce
          .style("border-radius", "50%") // Hacerlo circular
          .style("position", "absolute")
          .style("left", "30%") // Centrar el círculo de bronce dentro del círculo de Ronaldo
          .style("top", "30%"); // Centrar el círculo de bronce dentro del círculo de Ronaldo
      }
    })
    .on("mouseover", (event, d) => {
      d3.select(tooltip)
        .style("visibility", "visible")
        .style("left", `${event.pageX + 10}px`)
        .style("top", `${event.pageY - 20}px`)
        .html(`<strong>${d.Nombre}</strong><br/>Ranking: ${d.ranking}`);
    })
    .on("mousemove", (event) => {
      d3.select(tooltip)
        .style("left", `${event.pageX + 10}px`)
        .style("top", `${event.pageY - 20}px`);
    })
    .on("mouseout", () => {
      d3.select(tooltip).style("visibility", "hidden");
    });
}


</script>


<style>
  *{
    margin: 0;
    padding: 0;
    vertical-align: baseline;
    box-sizing: border-box;
  }
  body {
    background-color: #000000; /* Asegura que todo el fondo sea negro */
    color: white;
  }

  .circle-container {
    position: relative;
    width: 100%;
    height: 100vh ; /* Altura completa de la ventana */
    overflow: hidden; /* Para evitar barras de desplazamiento */
  }

  .player-circle {
    
    border-radius: 50%; /* Asegurar que cada círculo tenga bordes redondeados */
    overflow: hidden; /* Para asegurar que el contenido se mantenga dentro del círculo */
    transform: translateX(-50%);
  }

 /* Leyenda */
 .legend {
  display: flex;
  flex-direction: row; /* Alinea las secciones en fila */
  justify-content: space-between; /* Espacio entre las secciones y la imagen */
  align-items: flex-start; /* Alinear verticalmente al principio */
  width: 100%;
  gap: 70px;
  margin-left: 70px; 

}

.legend-section {
  display: flex;
  flex-direction: column; /* Las secciones (continente, posición, edad) están en columnas */
  gap: 10px; /* Espacio entre las secciones */
  align-items: flex-start; /* Alinea los elementos al inicio */

}

.legend-item {
  display: flex;
  align-items: center;
  gap: 8px;
}

.imagen-referencia {
  width: 80%; /* Ajusta la imagen al 50% de su tamaño original */
  height: auto; /* Mantiene la proporción de la imagen */
  margin-left: 20px; /* Añade espacio a la izquierda para separarla de las secciones */
}


.legend-circle, .legend-border, .border-circle {
  width: 25px;
  height: 25px;
  border-radius: 50%;
  display: inline-block;
}

h5 {
  white-space:nowrap;
  color: #D3C674;
  margin-right: 20px;
}


  .legend-border {
    width: 30px;
    height: 30px;
    border-radius: 50%;
    display: inline-block;
    background-color: transparent; /* Color de fondo para el círculo */
  }

  .border-circle {
    display: inline-block;
    border-radius: 50%;
    width: 30px;
    height: 30px;
    background-color: transparent;
  }

  /* Tooltip */
  .tooltip {
    position: absolute;
    background-color: white;
    color: black;
    border: 1px solid black;
    padding: 5px;
    border-radius: 5px;
    visibility: hidden; /* Oculto por defecto */
    pointer-events: none; /* Deshabilitar interacción con el tooltip */
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
    z-index: 10; /* Asegura que el tooltip esté por delante de otros elementos */
  }
  .premio {
  width: 40%; /* Ajusta el tamaño según necesites */
  height: 40%; /* Ajusta el tamaño según necesites */
  border-radius: 50%;
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%); /* Centrando el círculo */
  pointer-events: none; /* Para que no interfiera con el tooltip */
}

.premio-oro {
  background-color: gold;
}

.premio-plata {
  background-color: silver;
}

.premio-bronce {
  background-color: #cd7f32;
}


  h5 {
    color: #D3C674;
  }

  .titulo-container {
  display: flex; /* Alinear los elementos en una línea */
  align-items: center; /* Centrar verticalmente la imagen y el texto */
  justify-content: center; /* Centrar horizontalmente el contenido */
  gap: 20px; /* Espacio entre la imagen y el título */
}

.titulo-imagen {
  width: 70px; /* Ajusta el tamaño de la imagen según necesites */
  height: auto; /* Mantiene la proporción de la imagen */
  margin: 0px;
}

h1 {
  color: #D3C674;
  /*font-size: 2rem;  Ajusta el tamaño de la fuente según sea necesario */
  margin: 0; /* Elimina los márgenes alrededor del título */
}
.footer {
    text-align: center;
    margin-top: 150px;
    padding: 20px;
    color: #D3C674;
  }

  .footer-image {
    width: 250px; /* Ajusta el tamaño de la imagen */
    margin-top: 10px;
  }

</style>

<!-- Contenedor para la imagen y el título -->
<div class="titulo-container">
  <h1>Ranking Balón de Oro 2019</h1>
  <img src="/images/titulo2.jpg" alt="Título" class="titulo-imagen" />
  
</div>
<h5>La dominancia defensiva de Van Dijk no alcanzó para superar a la grandeza de <span style="color: #41b6c4;">Messi</span>, que gana su 6to Balón de Oro.</h5>


<!-- Contenedor principal con la visualización -->
<div class="circle-container" bind:this={container}></div>

<!-- Tooltip -->
<div class="tooltip" bind:this={tooltip}></div>

<div class="legend">
  <div class="legend-section">
    <h5>Podio</h5>
    <div class="legend-item"><div class="legend-circle" style="background-color: gold;"></div><span>1ero</span></div>
    <div class="legend-item"><div class="legend-circle" style="background-color: silver;"></div><span>2do </span></div>
    <div class="legend-item"><div class="legend-circle" style="background-color: #cd7f32;"></div><span>3ero </span></div>
  </div>
  <div class="legend-section">
    <h5>Continente</h5>
    <div class="legend-item"><div class="legend-circle" style="background-color: #41b6c4;"></div><span>América</span></div>
    <div class="legend-item"><div class="legend-circle" style="background-color: #fe9929;"></div><span>África</span></div>
    <div class="legend-item"><div class="legend-circle" style="background-color: #78c679;"></div><span>Europa</span></div>
  </div>
  <div class="legend-section">
    <h5>Posición</h5>
    <div class="legend-item"><div class="legend-border" style="border: 3px solid #30ff24;"></div><span>Arquero</span></div>
    <div class="legend-item"><div class="legend-border" style="border: 3px solid #fe0000;"></div><span>Defensor</span></div>
    <div class="legend-item"><div class="legend-border" style="border: 3px solid #ffff00;"></div><span>Mediocampista</span></div>
    <div class="legend-item"><div class="legend-border" style="border: 3px solid #00ffff;"></div><span>Delantero</span></div>
  </div>
  <div class="legend-section">
    <h5>Edad (años)</h5>
    <div class="legend-item"><div class="border-circle" style="border: 1px solid white;"></div><span>20-24</span></div>
    <div class="legend-item"><div class="border-circle" style="border: 5px solid white;"></div><span>25-29</span></div>
    <div class="legend-item"><div class="border-circle" style="border: 9px solid white;"></div><span>30-35</span></div>
  </div>

  <div class="legend-section">
    <h5>% de votos</h5>
    <img src="/images/tamano.png" alt="Tamaño" class="imagen-referencia" />
  </div>

</div>
<div class="footer">
  <p>Visualización creada por Benjamín Barceló</p>
  <img src="/images/ditella.jpg" alt="Logo Di Tella" class="footer-image" />
</div>
