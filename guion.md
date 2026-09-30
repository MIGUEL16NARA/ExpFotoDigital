# Guion de la exposición — El legado de Sycamore Gap

Tiempo total: 15 min (5 min por expositor). Cambien los nombres en `index.html` (objeto `EQUIPO`).

## Expositor 1

### Portada (0:00 – 0:30)

Buenas. Vamos a presentar el artículo The Legacy of Sycamore Gap, publicado en 2024 en los Archivos Internacionales de la ISPRS. Lo escribió un equipo de la Fundación Bruno Kessler (Italia), el Politécnico de Milán, la Universidad de Newcastle y el National Trust del Reino Unido.

En una frase: cómo usar inteligencia artificial y fotos tomadas por turistas para reconstruir en 3D un árbol famoso que ya no existe.

### La ruta de hoy (0:30 – 1:00)

La exposición tiene tres partes de unos cinco minutos. Yo explico el problema y los datos; Expositor 2 explica la metodología; Expositor 3 presenta los resultados y las conclusiones. Al final respondemos preguntas entre los tres.

### El árbol (1:00 – 2:00)

El Sycamore Gap era un sicómoro de unos 150 años que estaba en un hueco entre dos colinas, junto al Muro de Adriano, en el Parque Nacional de Northumberland, Inglaterra. El muro es Patrimonio Mundial de la UNESCO y el árbol era uno de sus paisajes más reconocidos.

La noche del 27 de septiembre de 2023 alguien lo taló a propósito; la foto muestra cómo quedó. El punto clave es este: ya no se puede ir a medirlo, porque el objeto desapareció.

### Patrimonio perdido (2:00 – 2:45)

Cuando un bien patrimonial se pierde, la única forma de recuperarlo digitalmente es con lo que quedó registrado antes. Ahí entra el crowdsourcing: usar fotos que tomó la gente. Como el árbol era tan famoso, hay muchísimas.

Con fotogrametría esas fotos se pueden convertir en un modelo 3D, que sirve para documentar, hacer réplicas o experiencias de realidad virtual. A la derecha ven además un escaneo láser que alcanzaron a hacer del árbol caído; lo vamos a usar más adelante.

### El problema (2:45 – 3:45)

El problema es que estas fotos son muy distintas entre sí. Miren la figura: invierno con nieve, verano, atardecer, noche, blanco y negro. Cada una con una cámara diferente, con resoluciones desde 480 por 359 píxeles hasta 9504 por 6336. Además el árbol cambia: a veces tiene hojas, a veces no, y el viento lo mueve.

La fotogrametría necesita puntos de enlace: el mismo detalle encontrado en dos fotos. Los métodos clásicos como SIFT fallan cuando los cambios de luz y ángulo son tan fuertes, y como cada foto viene de una cámara distinta, la autocalibración también es poco confiable.

La pregunta del artículo es entonces: ¿cómo orientar y reconstruir en 3D un conjunto de imágenes tan desordenado?

### Objetivos y datos (3:45 – 5:00)

Los autores se plantean dos objetivos: primero, orientar todas las imágenes disponibles con métodos de IA fotogramétrica; segundo, obtener un modelo 3D del árbol con buena exactitud geométrica y de color.

Los datos son tres. Unas 180 fotos de internet y del National Trust. Cuatro videos de dron que aportó el National Trust, porque allí está prohibido volar sin permiso. Y un escaneo láser terrestre que hizo la Universidad de Newcastle el 11 de octubre de 2023, catorce días después de la tala, con un Leica RTC360 y en solo 90 minutos antes de que retiraran el árbol. Ese escaneo es la referencia de escala y de precisión.

Ahora Expositor 2 explica cómo usaron estos datos.

## Expositor 2

### Flujo del método (5:00 – 6:00)

Este es el método completo en seis pasos. Uno: encontrar puntos de enlace entre las fotos con inteligencia artificial. Dos: con esos puntos, calcular dónde estaba cada cámara. Tres: generar vistas sintéticas desde el escáner láser, que tienen posición conocida y dan la escala. Cuatro: producir nubes de puntos densas. Cinco: completar el tronco y las ramas con el escáner. Seis: publicar todo en una plataforma web. Voy a detallar los pasos más importantes.

### Emparejamiento con IA (6:00 – 7:15)

El corazón del método es SuperPoint más LightGlue. SuperPoint es una red neuronal que detecta puntos característicos y los describe; LightGlue es otra red que decide qué punto de una foto corresponde a cuál en la otra. Como están entrenadas con casos extremos, toleran los cambios de luz y de ángulo que confunden a SIFT.

El problema es que estas redes no se diseñaron para imágenes de alta resolución. Por eso usaron la librería deep-image-matching, DIM, creada por el mismo grupo: primero compara todas las fotos reducidas a 1000 píxeles para saber qué pares se parecen, y después extrae los puntos en la imagen completa por teselas, es decir, por pedazos. Los emparejamientos pasan a COLMAP, que calcula la posición de cada cámara. En la figura, cada pirámide roja es una foto ubicada en el espacio.

### Ajuste y punto de control (7:15 – 7:55)

COLMAP hace un ajuste de haces: ajusta al mismo tiempo las posiciones de las cámaras y los puntos 3D para que los rayos se crucen lo mejor posible, parecido al ajuste por mínimos cuadrados de una red que vemos en la carrera.

Pero que el ajuste converja no garantiza que esté bien. Por eso eligieron un punto natural en el tronco, visible en muchas fotos, y midieron el error de reproyección: cuántos píxeles se separa el punto calculado del punto marcado a mano en cada foto.

### Vistas del láser (7:55 – 9:15)

El segundo aporte es cómo unir las fotos con el escáner láser. Lo tradicional es alinear aproximadamente las dos nubes y luego afinar con ICP. Eso es lento, usa una transformación rígida y solo considera la geometría, no el color.

Siguiendo a Elias y otros (2023), hicieron algo distinto: con la librería PlotOptix, que usa el motor de trazado de rayos NVIDIA OptiX, pusieron una cámara virtual y “tomaron fotos” de la nube del escáner. Generaron 21 vistas: una cenital y veinte oblicuas en círculo, de 1920 por 1080 píxeles, con GSD cercano a 2 cm. Un detalle importante: borraron el árbol caído de la nube, para que la IA no intentara emparejarlo con el árbol en pie de las fotos.

Como ellos mismos pusieron la cámara virtual, conocen exactamente su posición. Esas vistas entran al ajuste como restricciones, y así el modelo queda con escala y georreferenciado.

### Videos y tronco (9:15 – 10:00)

Con los videos se hizo lo mismo: frames de cada video más las vistas sintéticas, emparejados con DIM. Como COLMAP no permite meter posiciones de cámara conocidas en el ajuste, pasaron el bloque a Agisoft Metashape y allí fijaron las posiciones de las vistas sintéticas. Así obtuvieron una nube densa por video.

Para validar, usaron un tramo de muro de piedra que no cambió y compararon cada nube con una malla del escáner de 5 mm en CloudCompare. Y para el tronco, separaron hojas y madera del escáner con LeWoS y alinearon el árbol caído con la nube del video 4 mediante ICP.

Ahora Expositor 3 muestra qué tan bien funcionó todo esto.

## Expositor 3

### Imágenes orientadas (10:00 – 11:00)

Primer resultado: cuántas imágenes se lograron orientar de un total de 337, que incluye las fotos y algunos frames de video que agregaron para reforzar la red de cámaras.

Con el método clásico, RootSIFT en COLMAP, se orientaron 164, menos de la mitad. Con Agisoft Metashape, 215. Con SuperPoint y LightGlue, 299: casi nueve de cada diez. Frente a Metashape son 84 imágenes más, cerca de un 40 % más, que es la cifra que dan los autores en las conclusiones.

### Precisión (11:00 – 11:30)

¿Y qué tan exactas quedaron? En el punto de control del tronco, SuperPoint y LightGlue logró un error medio de reproyección de 2,04 píxeles, medido en 107 imágenes. Metashape tuvo 2,66 píxeles con solo 62 proyecciones. En ambos casos se descartaron las proyecciones con más de 5 píxeles de error. Es decir, la IA no solo orientó más imágenes, también las orientó mejor.

### Videos (11:30 – 12:10)

En los videos pasó algo parecido. DIM consiguió unir las vistas sintéticas del escáner con los frames reales en los cuatro videos; Metashape solo en dos. En la figura se ve arriba Metashape y abajo la IA; el círculo de puntos es la trayectoria del dron y el círculo central son las cámaras virtuales alrededor del tocón.

### Calidad en el muro (12:10 – 13:00)

Para medir la calidad compararon las nubes de cada video contra el muro de piedra del escáner. El error medio estuvo entre 2,4 y 4 centímetros y la desviación estándar entre 1,8 y 3 centímetros. Teniendo en cuenta que el tamaño de píxel en el terreno de los videos es de 2 a 5 centímetros, los autores lo consideran satisfactorio.

En los mapas de colores, azul es poco error y rojo es mucho. La cara vertical del muro sale bien; el error se concentra en la parte de arriba, que es irregular y queda oculta en los videos.

### Nubes del árbol (13:00 – 13:30)

Estas son las nubes densas del árbol, de frente y desde arriba. Los videos 1, 2 y 3 son de primavera y verano y reconstruyen casi toda la copa. El video 4 es de invierno y muestra el tronco y las ramas, pero con ruido, porque las ramas son delgadas y el video está muy comprimido.

### Árbol virtual (13:30 – 14:10)

Como el tronco del video 4 era ruidoso, usaron el escáner del árbol caído. Separaron hojas y madera, “levantaron” virtualmente el árbol y lo alinearon con el árbol en pie del video 4 usando ICP, con un error RMSE de 5,3 centímetros.

El resultado es el árbol virtual completo: la copa sale de los videos y el tronco y las ramas del escáner. Queda un hueco de unos 50 centímetros entre el tocón y el tronco, porque el National Trust cortó y tomó muestras antes de que se hiciera el escaneo.

### Plataforma web (14:10 – 14:30)

Todo se publicó en una plataforma web hecha con Potree, que permite ver nubes de puntos enormes en el navegador, medir y hacer cortes. Muestra tres épocas: el árbol antes de la tala, el árbol caído el 11 de octubre de 2023 y el tocón actual, de un vuelo de dron del 16 de abril de 2024. Las fotos originales aparecen ubicadas donde fueron tomadas.

### Conclusiones (14:30 – 15:00)

En conclusión: la inteligencia artificial permitió orientar cerca de un 40 % más de imágenes y con mejor precisión; las vistas renderizadas del escáner dieron escala y referencia de forma directa y redujeron el tiempo de emparejamiento; y la memoria del árbol quedó disponible para cualquier persona. Como trabajo futuro los autores proponen probar NeRF y Gaussian Splatting.

Para nuestra carrera, la idea importante es que fotos de archivo o tomadas por ciudadanos pueden convertirse en información métrica, útil para patrimonio y también para actualizar información del territorio.

## Cierre

### Preguntas

Muchas gracias. Quedamos atentos a sus preguntas. Cada uno responde las de su parte; repasen las dos diapositivas siguientes antes de exponer.

### Banco de preguntas

Anexo para estudiar. La rúbrica evalúa que todos puedan contestar y participen por igual.

### Glosario

Glosario para repasar antes de la exposición.
