PAV - P4: reconocimiento y verificación del locutor
===================================================

Obtenga su copia del repositorio de la práctica accediendo a [Práctica 4](https://github.com/albino-pav/P4)
y pulsando sobre el botón `Fork` situado en la esquina superior derecha. A continuación, siga las
instrucciones de la [Práctica 2](https://github.com/albino-pav/P2) para crear una rama con el apellido de
los integrantes del grupo de prácticas, dar de alta al resto de integrantes como colaboradores del proyecto
y crear la copias locales del repositorio.

También debe descomprimir, en el directorio `PAV/P4`, el fichero [db_8mu.tgz](https://atenea.upc.edu/pluginfile.php/3145524/mod_assign/introattachment/0/spk_8mu.tgz?forcedownload=1)
con la base de datos oral que se utilizará en la parte experimental de la práctica.

Como entrega deberá realizar un *pull request* con el contenido de su copia del repositorio. Recuerde
que los ficheros entregados deberán estar en condiciones de ser ejecutados con sólo ejecutar:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~.sh
  make release
  run_spkid mfcc train test classerr verify verifyerr
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Recuerde que, además de los trabajos indicados en esta parte básica, también deberá realizar un proyecto
de ampliación, del cual deberá subir una memoria explicativa a Atenea y los ficheros correspondientes al
repositorio de la práctica.

A modo de memoria de la parte básica, complete, en este mismo documento y usando el formato *markdown*, los
ejercicios indicados.

## Ejercicios.

### SPTK, Sox y los scripts de extracción de características.

- Analice el script `wav2lp.sh` y explique la misión de los distintos comandos involucrados en el *pipeline*
  principal (`sox`, `$X2X`, `$FRAME`, `$WINDOW` y `$LPC`). Explique el significado de cada una de las 
  opciones empleadas y de sus valores.
  
  sox: Sirve para cambiar el formato de una señal de entrada a uno de salida que nos convenga. Para saber las características de sox, escribimos sox -h en el terminal. En nuestro caso, al utilizar -t raw estamos diciendo que queremos un fichero tipo raw. Con -e signed definimos una codificación signed integer. Por último, -b 16 nos dice que queremos 16 bits.

  X2X: Programa de sptk que sirve para transformar datos input a otro formato output. La manera de utilizar este comando en el terminal es la siguiente: x2x [+type1 [+type2][–r] [–o] [%format]. En nuestro caso, +sf transforma los datos a short int format.

  FRAME: Extrae el frame de la secuencia de datos. -l indica la longitud del frame, y -p indica el periodo del frame.

  WINDOW: Enventanado de ventana. -l indica la longitud de frames del input. -L indica la longitud de frames del output.

  LPC: Calcula los coeficientes de predicción lineal. -l indica la longitud de frames, y -m indica el orden de coeficientes LPC.
  
  
  

- Explique el procedimiento seguido para obtener un fichero de formato *fmatrix* a partir de los ficheros de
  salida de SPTK (líneas 45 a 47 del script `wav2lp.sh`).
  
  Primero de todo, se obtiene el fichero $base.lp con los coeficientes LPC, encadenando los comandos descritos en el apartado anterior.

Segundo, se fija una cabecera para el archivo de salida con el número de filas y columnas de la matriz. El número de columnas será el orden del LPC + 1, puesto que en la primera columna se encuentra el factor de ganancia. El número de filas será el número total de tramas a las que se les ha calculado los coeficientes LPC. Se extrae del fichero .lp convirtiendo el contenido a ASCII con X2X +fa y contando el número de líneas con el comando wc -l.
  
  
  

  * ¿Por qué es conveniente usar este formato (u otro parecido)? Tenga en cuenta cuál es el formato de
    entrada y cuál es el de resultado.
    
    Porque se tiene un fácil y rápido acceso a todos los datos almacenados con una correspondencia directa entre la posición en la matriz y el orden del coeficiente y número de trama, por lo que simplifica mucho su manipulación a la hora de trabajar. También nos ofrece información directa en la cabecera sobre el número de tramas y número de coeficientes calculados.



- Escriba el *pipeline* principal usado para calcular los coeficientes cepstrales de predicción lineal
  (LPCC) en su fichero <code>scripts/wav2lpcc.sh</code>:

- Escriba el *pipeline* principal usado para calcular los coeficientes cepstrales en escala Mel (MFCC) en su
  fichero <code>scripts/wav2mfcc.sh</code>:

### Extracción de características.

- Inserte una imagen mostrando la dependencia entre los coeficientes 2 y 3 de las tres parametrizaciones
  para todas las señales de un locutor.
  
![image](https://user-images.githubusercontent.com/92084115/147713081-2a1ef078-5c7d-4286-917c-1f6aa704ca29.png)

![image](https://user-images.githubusercontent.com/92084115/147713108-5be0beaa-61d5-400c-bc04-f27cff8aada0.png)

![image](https://user-images.githubusercontent.com/92084115/147713122-38c3cf38-8b37-4a73-9e70-7842b6a82bfc.png)


  
  + Indique **todas** las órdenes necesarias para obtener las gráficas a partir de las señales 
    parametrizadas.
    
    fmatrix_show work/lp/BLOCK01/SES010/*.lp | egrep '^\[' | cut -f4,5 > lp_2_3.txt 
    
    fmatrix_show work/lpcc/BLOCK01/SES010/*.lpcc | egrep '^\[' | cut -f4,5 > lpcc_2_3.txt 
    
    fmatrix_show work/mfcc/BLOCK01/SES010/*.mfcc | egrep '^\[' | cut -f4,5 > mfcc_2_3.txt
    
  + ¿Cuál de ellas le parece que contiene más información?

- Usando el programa <code>pearson</code>, obtenga los coeficientes de correlación normalizada entre los
  parámetros 2 y 3 para un locutor, y rellene la tabla siguiente con los valores obtenidos.
  
  pearson work/lp/BLOCK01/SES010/*.lp     
  
  ![image](https://user-images.githubusercontent.com/92084115/147713415-8bbc3357-972c-47b2-885b-65c650ac60ed.png)

  pearson work/lpcc/BLOCK01/SES010/*.lpcc 
  
  ![image](https://user-images.githubusercontent.com/92084115/147713480-2aec616f-3d29-4871-8fdd-f7baceddddc0.png)

  pearson work/mfcc/BLOCK01/SES010/*.mfcc
   
  ![image](https://user-images.githubusercontent.com/92084115/147713510-d45b17c6-4ad2-4254-926f-517d77198d4e.png)


  

  |                        | LP         | LPCC         | MFCC        |
  |------------------------|:----:      |:----:        |:----:       |
  | &rho;<sub>x</sub>[2,3] | -0.664123  | 0.292243     | 0.067202    |
  
  + Compare los resultados de <code>pearson</code> con los obtenidos gráficamente.
  
- Según la teoría, ¿qué parámetros considera adecuados para el cálculo de los coeficientes LPCC y MFCC?

### Entrenamiento y visualización de los GMM.

Complete el código necesario para entrenar modelos GMM.

- Inserte una gráfica que muestre la función de densidad de probabilidad modelada por el GMM de un locutor
  para sus dos primeros coeficientes de MFCC.
  
  <img width="642" alt="función de densidad _mfcc" src="https://user-images.githubusercontent.com/92084115/147713553-6098383e-6e4b-4390-9cf9-2a0ccae33f5a.png">

  
- Inserte una gráfica que permita comparar los modelos y poblaciones de dos locutores distintos (la gŕafica
  de la página 20 del enunciado puede servirle de referencia del resultado deseado). Analice la capacidad
  del modelado GMM para diferenciar las señales de uno y otro.
  
  
  <img width="858" alt="C" src="https://user-images.githubusercontent.com/92084115/147713618-57d6d55c-1c7a-4dc9-b63a-646b531c4d2f.png">
  
  <img width="643" alt="CORRESPONDE" src="https://user-images.githubusercontent.com/92084115/147713679-0084f496-8f93-4f0d-a4a7-c4240940f5bc.png">


  <img width="860" alt="NC" src="https://user-images.githubusercontent.com/92084115/147713640-21b08cdb-9387-4d71-bb28-57eafb879ab4.png">
  
  <img width="641" alt="NO CORRESPONDE" src="https://user-images.githubusercontent.com/92084115/147713650-1823e842-189c-4eca-b861-ef8aaad2f00f.png">
  
  
  <img width="855" alt="C_2" src="https://user-images.githubusercontent.com/92084115/147713690-0f17a782-4713-4c82-bf15-f5dfb624e9cd.png">
  
  <img width="643" alt="CORRESPONDE 2" src="https://user-images.githubusercontent.com/92084115/147713695-5288839f-8cd9-4ecd-bcdc-66d764debf47.png">
  
  
  <img width="854" alt="NC_2" src="https://user-images.githubusercontent.com/92084115/147713697-ee86717d-8ad1-40ed-b2a2-467ce25a1034.png">
  
  <img width="636" alt="NO CORRESPONDE 2" src="https://user-images.githubusercontent.com/92084115/147713706-bc442b43-f613-49a3-adc6-31181f16800e.png">



### Reconocimiento del locutor.

Complete el código necesario para realizar reconociminto del locutor y optimice sus parámetros.

- Inserte una tabla con la tasa de error obtenida en el reconocimiento de los locutores de la base de datos
  SPEECON usando su mejor sistema de reconocimiento para los parámetros LP, LPCC y MFCC.
  
  LP
  
  <img width="500" alt="error_rate_lp" src="https://user-images.githubusercontent.com/92084115/147713744-d201c237-0441-479b-8c01-ab6873c84a63.png">
  
  LPC
  
  <img width="498" alt="error_rate_lpcc" src="https://user-images.githubusercontent.com/92084115/147713747-c901bcea-80bc-4c15-9d03-9a45515163e1.png">
  
  MFCC
  
  <img width="509" alt="error_rate_mfcc" src="https://user-images.githubusercontent.com/92084115/147713759-ffafb3a2-1fbc-4a2d-b235-eeaf7055c52c.png">
  

  |           | LP     | LPCC  | MFCC  |
  |-----------|:----:  |:----: |:----: |
  | Error Rate| 10.83% | 3.44% | 1.15% |


### Verificación del locutor.

Complete el código necesario para realizar verificación del locutor y optimice sus parámetros.

- Inserte una tabla con el *score* obtenido con su mejor sistema de verificación del locutor en la tarea
  de verificación de SPEECON. La tabla debe incluir el umbral óptimo, el número de falsas alarmas y de
  pérdidas, y el score obtenido usando la parametrización que mejor resultado le hubiera dado en la tarea
  de reconocimiento.
  
  <img width="424" alt="mfcc_th" src="https://user-images.githubusercontent.com/92084115/147713869-abb104c3-b003-407c-b534-d8f4c90314de.png">

 
### Test final

- Adjunte, en el repositorio de la práctica, los ficheros `class_test.log` y `verif_test.log` 
  correspondientes a la evaluación *ciega* final.

### Trabajo de ampliación.

- Recuerde enviar a Atenea un fichero en formato zip o tgz con la memoria (en formato PDF) con el trabajo 
  realizado como ampliación, así como los ficheros `class_ampl.log` y/o `verif_ampl.log`, obtenidos como 
  resultado del mismo.
