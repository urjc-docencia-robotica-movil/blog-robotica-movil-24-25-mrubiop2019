# P2 Follow Line
## Alrgoritmo

El algoritmo implementado es relativamente simple. Primero, la cámara proporciona una imagen a la que se le pasa un filtro de color con el que se detecta la línea.
Con la imagen filtrada, se calcula el offset de la línea respecto al centro de la pantalla, y con ese offset como referencia, usamos un controlador PID que nos dará un output.
Ese output estará directamente relacionado con los inputs de velocidad lineal y angular del robot.

## Filtro de color

El filtro de color es simple, pero eficaz. Usando la imagen BGR original, usando la biblioteca de OpenCV la pasamos al espacio de color HSV, donde podemos aplicar un filtro más consistente.
Notablemente, el filtro rojo requiere aplicar dos máscaras, una de H = 330 a H = 359 y otra de H = 0 a H = 30. Una vez aplicadas, usamos la operación bitwise_or de OpenCV para obtener la imagen completa que deseamos.

Aqí se puede observar el resultado del fitro en funcionamiento:
![image](https://github.com/user-attachments/assets/0676d934-9c0a-48b6-adf3-3df539544107)

## Calculo del error

Para calcular el error, nos fijamos en las 10 primeras lineas de pixeles de la imagen filtrada, empezando por la mitad de la pantalla. Buscamos pixeles con valor distinto de 0, aquellos pertenecientes a la linea roja.
Por cada pixel de la linea que encontremos, guardamos su valor x y calculamos la media. Restando el centro de la imagen a esa media, obtenemos el error en pixeles, que pasamos a un valor unitario dividiento entre la mitad de columnas de la imagen.
De esta forma, si la curva está en alguno de los extremos, el valor máximo del error es 1 o -1. Tener un error (maximo) unitario facilita la visualización y los cálculos posteriores.

También cabe resaltar que, para casos en los que la linea haya desaparecido del rango visible, el error será de 1 o -1, usando el signo del error anterior para encontrar la direccion de giro optima.
De esta forma, es posible que el algoritmo funcione incluso si la linea no es visible.

## Implementacion del PID
### Errores derivativo e integral

Un controlador PID tiene un funcionamiento bastante simple, convirtiendo un error proporcional en un output usando los errores derivativo e integral.
Para calcular el error derivativo, usamos la diferencia entre el error anterior y el actual. En el caso del error integral, hay diferentes formas de implementarlo. 

En mi caso, uso una implementacion simple, sumando los valores del error proporcional hasta llegar a 10 iteraciones, en cuyo caso el error integral vuelve a cero. 
Una implementacion más elegante usaría un array en el que almacenaría estos valores, y sumaría el array en cada iteración para calcular el error integral. 
En mi experciencia, iterar este array aumentaba el tiempo de computo y reducía la reactividad del coche.

### PID de rectas vs curvas

Una optimización eficaz para optimizar la velocidad del coche es usar diferentes PIDs para diferentas partes del circuito. Para ello, debemos considerar los requisitos del comportamiento de cada PID.

En las rectas, el coche debe girar de forma suave, y tratar de usar velocidades lineales maximas.
Para ello, las componentes proporcional y derivativa del PID serán bajas, dando más prioridad a la integral, que se encargará de corregir si la trayectoria del coche es paralela a la línea pero no la alcanza.

Por contraste, en las curvas, el coche deberá ser capaz de ajustar rápidamente a errores mayores.
Para conseguir esto se aumenta el peso del error derivativo y proporcional, que proporcionarán al coche la capacidad de cambiar su trayectoria de forma más drástica.

## Velocidad

Finalmente, tras obtener el output del PID, tenemos que determinar la velocidad angular y lineal del coche. La implementacion que he escogido usa el output del PID para calcular ambas velocidades.

En el caso de la velocidad angular, he determinado una velocidad angular máxima, que se alcanzaría en casos de outputs de 1 o -1. 
Debido al comportamiento del PID, el mayor output tras el procesamiento los mayores outputs son más cercanos a ±0.5, así que la mayor velocidad angular suele acabar en la mitad de la velocidad angular máxima en los giros más cerrados.

Por el contrario, la velocidad lineal debe disminuir en las curvas, pero mantenerse alta en las rectas. La velocidad lineal es inversamente proporcional a la velocidad angular. Sin embargo, es importante tratar que esta nunca se vuelva cero.
Volviendo a un error maximo (ahora en valor absoluto) de 0.5, una proporcionalidad inversa simple nos resultaría en que la velocidad en las rectas sería el doble que en las curvas. Sin embargo, en mi experiencia, este rango no era óptimo.
Es por ello que incluyo en el cálculo la proporcion entre velocidad linear y angular máxima. De esta forma, el robot puede hacer giros más cerrados en caso de necesidad.

Concretando más, con una velocidad lineal maxima de 20(m/s), y angular de 15(rad/s), el giro más cerrado que puede realizar el coche es de radio 1m a 7.5m/s.
Esto es muy útil para compensar el tiempo de reacción al entrar en curvas a altas velocidades (20m/s), a menudo requiriendo un giro brusco en la entrada que se estabiliza en la salida o si la curva es larga.

## Dificultades

La mayor dificultad que he encontrado es la necesidad de una velocidad de procesamiento rápida, que me ha impedido implementar un cálculo más optimo del error integral.
Además, debido a limitaciones del hardware, el framerate en el que trabajaba era relativamente bajo, y a menudo inconsitente cuando el procesador se sobrecalentaba. 
Es por ello que observaba comportamientos inconsistentes en mi testeo, y afinar el PID para un comportamiento óptimo resultó imposible, ya que en pruebas sucesivas el framrate se volvía excesivamente bajo y el algoritmo dejaba de funcionar.

Una mejora que es posible merezca la pena implementar es un tercer PID para diferenciar entre curvas más suaves y más cerradas.
Esto permitiría al coche transicionar mejor a la hora de coger una curva, y tener un comportamiento consistente en ambos escenarios.

## Videos
A continuación vinculo los videos del coche en diferentes escenarios:

https://www.youtube.com/watch?v=lFFoVya-p7I

https://www.youtube.com/watch?v=ogNaEkUyFRc


