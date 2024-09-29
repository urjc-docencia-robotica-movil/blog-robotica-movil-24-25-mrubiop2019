# P1 vacuum Cleaner
## Algoritmo
El algoritmo implementado es simple, el robot comienza en un movimiento espiral capaz de cubrir la mayor superficie posible hasta que el bumper se active. 
Con la activacion del bumper, el robot inicia un giro aleatorio y un movimiento hacia adelante.
Si tras el movimiento hacia adelante no se detecta ningun obstaculo, el robot retorna a su movimiento espiral. Por el contrario, si sigue habiendo contacto en el bumper, el robot repite el giro y el movimiento hacia adelante.

El siguiente diagrama de estados muestra el algoritmo descrito:

![fsm_p1](https://github.com/user-attachments/assets/6250bd9b-7331-4878-a307-71fc63c3bd3b)

## Dificultades y soluciones

La primera dificultad fue encontrar las velocidades y duraciones adecuadas para los distintos estados. 
El movimiento espiral debe ser exhaustivo, pero no demasiado lento, mientras que el giro y el movimiento hacia adelante se pueden permitir mayor velocidad.
En el caso de las duraciones, quería que tanto el giro como el movimiento hacia adelante tuvieran duraciones aleatorias para evitar que el robot acabe en una secuencia determinada que le impida desenvolverse en un entorno desconocido.

En el caso del giro, acabe determinando una duracione entre 0.5 y 1.5 segundos, proporcionando giros de aproximadamente 45 a 135 grados. 
Sin embargo, el movimiento recto presentaba una dificultad adicional. Si era demasiado pequeño, el robot no avanzaba a nuevas regiones de la casa, mientras que si era demasiado grande, el robot se atascaba con mayor facilidad en espacios cerrados sin entrar en movimiento espiral.

Para solucionarlo, añadí una variable "bumps" que cuenta las veces que el robot se ha chocado sin entrar en el estado de espiral. 
La distancia que el robot avanza tras girar disminuye cuanto mayor sea esta variable, haciendo así que los movimientos hacia adelante se vuelvan más cortos en espacios cerrados.

## Video demostrativo

El siguiente video muestra el compertamiento del algoritmo en un lapso de 20 minutos a velocidad x8

https://urjc-my.sharepoint.com/:v:/g/personal/m_rubiop_2019_alumnos_urjc_es/EVEanxZ-P-hLuUZy0Dy1pYkBny49IRpD8B7IMd7QD6lr3A?nav=eyJyZWZlcnJhbEluZm8iOnsicmVmZXJyYWxBcHAiOiJPbmVEcml2ZUZvckJ1c2luZXNzIiwicmVmZXJyYWxBcHBQbGF0Zm9ybSI6IldlYiIsInJlZmVycmFsTW9kZSI6InZpZXciLCJyZWZlcnJhbFZpZXciOiJNeUZpbGVzTGlua0NvcHkifX0&e=pvxdCr
