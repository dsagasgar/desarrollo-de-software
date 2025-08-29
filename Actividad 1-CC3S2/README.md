# Actividad 1 - Sergio Quesada
# Tiempo invertido: 02:00

## Devops vs. cascada tradicional 
### Imagen comparativa
![Cuadro](imagenes/devops-vs-cascada.png)
![DevOps](imagenes/devops.png)
![Cascada](imagenes/cascada.png)
### ¿Por qué DevOps acelera y reduce riesgo en software para la nube frente a cascada?
Porque devops usa herramientas de automatización y gestión de contenedores en el que ejecutan los microservicios, la integración de nuevo código y su despliegue (CI/CD) en la nube es adecuado ya que nos brindan servicios basados en tecnologías como kubernetes y docker. Nos permiten hacer pruebas de escalabilidad y rendimiento
con un costo proporcional. En cuanto a la metodología de cascada, tendríamos que esperar a que el equipo de desarrollo termine el trabajo para recien desplegarlo en la nube de forma completa.
### Contexto real donde un enfoque cercano a cascada sigue siendo razonable
Un software para registrar participantes y presentaciones de un evento, no requiere actualización continua, ya que es solo un evento y las funcionalidades son pocas y claramente definidas.
## Ciclo tradicional de dos pasos y silos (limitaciones y anti-patrones)
### Silos organizacionales
![Silos](imagenes/silos-equipos.png)
### Limitaciones
- El equipo de software entrega una gran cantidad de material y se lava las manos, esto dificulta la tarea del equipo de operaciones.
- Al no existir una retroalimentación continua, los errores se acumulan hasta llegar a la siguiente fase provocando retrasos, más trabajo y aumentando los costos. La alta cantidad de problemas provoca un mayor tiempo de recuperación.
### Define dos anti-patrones ("throw over the wall", seguridad como auditoría tardía) y explica cómo agravan incidentes (mayor MTTR, retrabajos, degradaciones repetitivas).
**throw over the wall** se refiere a lanzar el material hacía el otro equipo, como si existiese un muro, una barrera de comunicación en el que si a un equipo le funciona, entonces su trabajo ha terminado y el resto es responsabilidad del otro equipo. Los fallos que se producen en operaciones tardan mucho más en corregirse(mayor MTTR) y devuelven el material al equipo de desarrollo que tiene que volver a trabajar para corregir errores que pudieron haberse detectado de forma temprana.
**seguridad como auditoría tardía**, en el enfoque tradicional los análisis de seguridad se realizan al final del proyecto, esto produce la detección de fallos de seguridad que son más costosos de reparar ya que estamos delante de un proyecto ya integrado, el enfoque devsecops los analisis de seguridad y buenas prácticas se realizan de forma continua.
