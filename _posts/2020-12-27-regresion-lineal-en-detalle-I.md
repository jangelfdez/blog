---
layout: post
title: "Regresión lineal en detalle, los primeros pasos en Machine Learning"
author: "José Ángel Fernández"
categories: blog
tags: [machine learning, regresion lineal]
image: machinelearning.jfif
---

El primer paso para cualquier persona que quiera empezar con el Aprendizaje Automático o *Machine Learning* es bastante probable que sea la regresión lineal. Si realmente su uso entra en la categoría del Aprendizaje Automático o, en su lugar, es simplemente una herramienta de modelado estadístico daría para otra conversación. Yo personalmente me decanto más por la versión del *Machine Learning* como la mezcla entre la estadística y la probabilidad, algo más allá de ese halo místico que parece envolver a todo lo relacionado con la Inteligencia Artificial. Sin embargo, visto así parece que se le quita toda la magia y la mercadotecnia.

Recientemente acabé el [Máster Universatrio en Métodos Análiticos para Datos Masivos: Big Data](https://www.uc3m.es/master/big-data) de la Universidad Carlos III. Una forma de salir del día a día en el mundo de la infraestructura y el despliegue de aplicaciones en el que he estado los últimos diez años, al tan relevante nuevo mundo del dato, la piedra angular de las organizaciones para los próximos años. 

Trabajar y estudiar no es algo sencillo y una de las cosas que me arrepiento es no poder haber disfrutado de más tiempo para bajar al detalle en los conceptos y las técnicas que nos enseñaron. Como por ejemplo, el motivo detrás de este artículo: conocer más en profundidad los detalles que hay detrás de una regresión lineal, más allá de la facilidad que algunas librerías como Scikit-learn nos proporciona con su *split*, *fit* y *predict*. 

En este artículo la idea es conocer qué tipo de problemas son candidatos para aplicar una regresión lineal, qué condiciones tienen que cumplir los datos de los que disponemos para que realmente el resultado del modelo tenga algún valor para sacar conclusiones y las opciones matemáticas a la hora de resolverlo. Es posible que información similar aparezca por la Wikipedia u otros sitios webs; sin embargo, leerlo no implica saberlo por lo que no hay nada mejor que intentar explicarlo para ver hasta qué punto se ha comprendido.

### ¿Qué problemas son candidatos para una regresión lineal?

Los modelos de aprendizaje automático basados en el uso de una regresión lineal se emplean para predecir el valor de una variable respuesta, $Y$, a un conjunto de variables denominadas predictores, $X_i$. También se le conoce respectivamente como variable dependiente y variables independientes. Como bien indica su nombre, la relación entre ellas será una relación lineal en los parámetros y se puede expresar según la ecuación \eqref{eq:rl}.

$$ \begin{equation}  Y = \beta_0 + \beta_1 X + ... + \beta_p X_p + \epsilon = \beta_0 + \sum_{i=1}^p \beta_iX_i + \epsilon  \label{eq:rl}\end{equation}$$

Estos parámetros $\beta_i$ serán los que tendremos que obtener a partir de los datos disponibles de nuestro problema. La linealidad de la regresión va asociado a dichos parámetros, no a las variables predictoras. Es decir, el siguiente caso también es una regresión lineal aunque pudiera parecer lo contrario ya que sus parámetros son lineales.

$$ Y = \beta_0 + \beta_1 \log X + ... + \beta_p X_p^2 + \epsilon $$

En cuanto a $$\epsilon$$, se considera una variable aleatoria que modela la interferencia de otros posibles predictores que afecten a nuestra relación lineal pero que no son considerados en nuestro modelo. Es posible verlo como el *error* de nuestra predicción respecto a la realidad si consideráramos todas las variables que afectarían a nuestro problema; en la mayor parte de los casos esas variables no incluídas ni siquiera son conocidas. Dicho error tendrá una media nula y es independiente de los predictores $X_p$.

### ¿Puedo utilizar una regresión lineal con los datos que tengo?

Una regresión lineal asume que los datos cumplen una serie de propiedades probabilísticas que nos permitan extraer conclusiones válidas a partir de los resultados obtenidos. Es decir, que las predicciones que realicemos con el modelo se ajusten a nuestro escenario real y tengan algún sentido.

Esto se debe a que los datos con los que contamos son una muestra de todos los posibles valores existentes de nuestro escenario completo. Si los datos no cumplen esas propiedades probabilísticas, no podremos extrapolar los resultados obtenidos a partir de la muestra a todo el conjunto completo de datos.

Las propiedades que tienen que cumplirse son: linealidad, homocedasticidad, normalidad e independencia de los errores. Pero, ¿qué implica cada una de ellas?

#### Linealidad

Cuando hablamos de *linealidad*, nos referimos a que la variable que predecimos debe de mantener una relación lineal con cada una de las variables predictoras. Expresado de forma matemática, la esperanza de la variable predicha equivale a una combinación lineal de las predictoras.

$$ \mathbb{E}[Y | X_1 = x_1, ..., X_p = x_p] = \beta_0 + \beta_1 x_i + ... + \beta_p x_p  $$

Esto es así ya que como mencionabamos anteriormente, la media del error es nula

$$ \mathbb{E}[\epsilon |X_1 = x_1, ..., X_p = x_p ] = 0 $$

En el caso de una regresión lineal simple con un solo predictor, la forma más sencillo de comprobarlo es utilizando un diagrama de dispersión (*scatterplot*) en el que se apreciará la relación entre el predictor y la variable respuesta. Sin embargo, en el caso de una regresión lineal múltiple, su forma de validarlo es comparando en un diagrama similar los residuos con el valor predicho. Más detalles de lo que es un residuo se pueden encontrar más adelante.  

#### Homocedasticidad

Probablemente de las palabras más trabalenguas que conozco para decirla bien a la primera. Sin embargo, su explicación es más sencilla. Los residuos deben de tener una varianza constante para cualquier valor de entrada. Expresado de forma matemática.

$$ \mathbb{Var}[\epsilon | X_1 = x_1, ..., X_p = x_p] =\sigma^2  $$

Es posible comprobarlo igual que antes con un diagrama de dispersión de los residuos comparado con los valores predichos. Su imcumplimiento provoca que no se pueda predecir de forma certera los errores de las predicciones que realiza nuestro modelo.

#### Normalidad 

A partir de lo que hemos visto en los dos puntos anteriores tenemos esta tercera condición.

$$ \epsilon \sim \mathcal{N}(0,\sigma^2) $$

Es decir, los errores siguen una distribución normal de media nula y varianza $\sigma^2$. Si esto no sucede, afecta a la hora de calcular los intervalos de confianza y considerar las probabilidades de que el error de una predicción excede un valor particular. 

#### Independencia de los errores

Los residuos obtenidos no deben de tener ninguna correlación entre ellos. Es decir, son independientes. Generalmente esto sucede en casos en los que los datos proceden de algún tipo de serie temporal en el que un valor tiene dependencia o correlación con los anteriores o posteriores. La forma más sencillo de diagnosticarlo es a través de un gráfico con las autocorrelaciones de los residuos. La mayor parte de los valores deberían caer entorno al cero.

### ¿Resolución analítica o numérica?

En este punto disponemos ya de nuestros datos de entrada y hemos verificado que cumplen las propiedades necesarias para que los resultados de la regresión lineal tengan sentido. ¿Cómo obtenemos entonces ahora los coeficientes asociados a nuestros predictores, solución de nuestro modelo de regresión lineal?. Desde el punto de vista matemático, contamos con dos opciones para ello: la *resolución analítica* o la *resolución numérica*.

En el primer escenario, buscamos una ecuación o conjunto de ecuaciones que nos permitan calcular la solución del problema de regresión lineal para cualquier valor de los predictores que empleemos. De esta manera, únicamente será necesario reemplazar los valores de nuestro problema específico en el conjunto de ecuaciones y tendremos la solución. En el segundo, por el contrario, en lugar de buscar una serie de ecuaciones universales nos centraremos en obtener la solución para los valores específicos de nuestros predictores.

Comparando ambas opciones, parece que el camino más adecuado es el primero: obtener una solución analítica que nos permita obtener la solución en cualquier escenario que nos encontremos. Sin embargo, mientras que esto es sencillo para el escenario de una regresión lineal simple donde únicamente tenemos un predictor, la situación se complica cuando necesitamos resolver una regresión lineal múltiple en la que el número de predictores aumenta. En estos casos es posible que el coste computacional para obtener la solución sea tan alto que necesitemos resolverlo numéricamente de forma más eficiente.

Veamos a continuación los detalles de ambos casos:

#### Resolución analítica

En primer lugar obtendremos la resolución analítica para el caso más sencillo: la regresión lineal simple. Como hemos visto anteriormente en el artículo, ésta se puede representar en forma de ecuación de la siguiente manera.

$$ Y = \beta_0 + \beta_1 X + \epsilon $$

Los datos de entrenamiento que tenemos para nuestro modelo podemos representarlos como el conjunto de tuplas $(X_1,Y_1),...,(X_n,Y_n)$ en el que para cada valor del predictor, $X_i$, conocemos el valor real de la respuesta, $Y_i$. De esta manera, necesitaremos encontrar los valores adecuados de los parámetros desconocidos $\beta_0,\beta_1$.

Utilizando el razonamiento geométrico, en la regresión lineal simple la solución es la línea recta que minimiza la distancia entre ella y cada una de las tuplas de entrada. Esta distancia se puede representar como la diferencia entre el valor real conocido de nuestra respuesta, $Y_i$, y el valor predicho por nuestro modelo, $\hat Y_i$.

La diferencia entre ambos valores, $Y_i - \hat Y_i$, se conoce como el residuo. En nuestro caso nos interesará minimizar la suma de todos los residuos obtenidos a partir de los datos de entrada. Al ser una diferencia, nos encontraremos resultados positivos y negativos que tenderán a anularse, por lo que se emplea el cuadrado de la diferencia a la hora de minimizar esa suma. Este concepto se le denomina como la suma de los residuos al cuadrado, o como se encuentra habitualmente en inglés: RSS *(Residual Sum of Squares)*.

Los valores reales de $(\beta_0, \beta_1)$ únicamente los podríamos conocer si tuviéramos los detalles de todas las tuplas asociadas a nuestro problema; es decir, conociéramos toda la población. Sin embargo, este nunca será el caso, únicamente tendremos una muestra de la población por lo que los valores que obtendremos serán una aproximación a ellos. Es por ese motivo que nos referiremos a ellos como $(\hat\beta_0,\hat\beta_1)$.

Representado de forma matemática, nuestros dos parámetros $(\hat\beta_0,\hat\beta_1)$ serán aquellos que minimicen la suma de los residuos al cuadrado de todas las tuplas que disponemos.

$$ (\hat\beta_0,\hat\beta_1) = \underset{(\beta_0,\beta_1)\in\Re} {\operatorname{arg\,min\,RSS}} (\beta_0,\beta_1) := \underset{(\beta_0,\beta_1)\in\Re} {\operatorname{arg\,min}}(\sum_{i=1}^n(Y_i - \hat Y_i)^2)$$

Reemplazando el valor de nuestra estimación, $\hat Y_i$, por su expresión matemática, el problema quedaría planteando de la siguiente manera.

$$ (\hat\beta_0,\hat\beta_1) = \underset{(\beta_0,\beta_1)\in\Re} {\operatorname{arg\,min}}(\sum_{i=1}^n(Y_i - \hat\beta_0 - \hat\beta_1X_i )^2)  $$

Nos encontramos ante un problema de optimización en el que buscamos el valor mínimo de esa expresión. Del cálculo numérico sabemos que el mínimo o máximo de una función se da cuando la derivada de la misma es igual a $0$. Por lo tanto, el mínimo de dicha expresión para cada uno de nuestros dos parámetros $(\hat\beta_0, \hat\beta_1)$ quedará definido como:

$$ \frac \partial{\hat\beta_0} [\sum_{i=1}^n(Y_i - \hat\beta_0 - \hat\beta_1X_i )^2)] = 0$$

$$ \frac \partial{\hat\beta_1} [\sum_{i=1}^n(Y_i - \hat\beta_0 - \hat\beta_1X_i )^2)] = 0$$

Por lo tanto, el siguiente paso será resolver cada una de estas expresiones por separado. Comenzaremos primero con la derivada respecto $\beta_0$. Dado que la derivada es una operación lineal, la derivada de una suma se puede representar como la suma de sus derivadas.
 
$$\frac \partial{\hat\beta_0} [\sum_{i=1}^n(Y_i - \hat\beta_0 - \hat\beta_1X_i )^2] =  \sum_{i=1}^n [\frac \partial{\hat\beta_0} (Y_i - \hat\beta_0 - \hat\beta_1X_i )^2] $$

Para simplificar, dado que únicamente nos interesa $\hat\beta_0$, haremos la siguiente sustitución en la ecuación $C=Y_i -\hat\beta_1X_i$.

$$\sum_{i=1}^n [\frac \partial{\hat\beta_0} (C - \hat\beta_0)^2)] $$

Aplicando la propiedad del cuadrado de la diferencia que indica que $(a-b)^2 = a^2 - 2ab + b^2$, nuestra ecuación se transfomará de la siguiente manera.

$$\sum_{i=1}^n [\frac \partial{\hat \beta_0} (C^2 -2C\hat\beta_0 + \hat\beta_0^2)] $$

Derivamos cada uno de los términos de la ecuación.

$$\sum_{i=1}^n [\frac \partial{\hat\beta_0} (C^2) - \frac \partial{\hat\beta_0} (2C\hat\beta_0) + \frac \partial{\hat\beta_0}(\hat\beta_0^2)] = \sum_{i=1}^n [ 0 - 2C + 2\hat\beta_0] $$

Deshacemos el cambio anterior de $C=Y_i -\hat\beta_1X_i$

$$-2\sum_{i=1}^n(C) + 2\sum_{i=1}^n\hat\beta_0 = -2\sum_{i=1}^n(Y_i -\hat\beta_1X_i) + 2n\hat\beta_0$$

Obteniendo finalmente la siguiente expresión.

$$2(n\hat\beta_0 -\sum_{i=1}^nY_i + \hat\beta_1\sum_{i=1}^nX_i)$$

Una vez que llegamos a esta expresión de la derivada, nos queda por lo tanto igualar el resultado a $0$ y despejar $\hat\beta_0$.

$$2(n\hat\beta_0 -\sum_{i=1}^nY_i + \hat\beta_1\sum_{i=1}^nX_i) = 0$$

$$\hat\beta_0 = \frac 1 n {\sum_{i=1}^nY_i} - \frac 1 n \hat\beta_1\sum_{i=1}^nX_i $$

Siendo $\frac 1 n {\sum_{i=1}^nA_i}$ la expresión de la media aritmética, podemos reducir la expresión analítica de $\hat\beta_0$ a su versión final.

$$\begin{equation} \hat\beta_0 = \bar Y - \hat\beta_1 \bar X \end{equation} $$

A continuación, tendremos que realizar el mismo proceso para resolver la derivada parcial respecto a $\hat\beta_1$.

$$\frac \partial{\hat\beta_1} [\sum_{i=1}^n(Y_i - \hat\beta_0 - \hat\beta_1 X_i)^2] =  \sum_{i=1}^n [\frac \partial{\hat\beta_1} (Y_i - \hat\beta_0 - \hat\beta_1X_i)^2] $$

Reemplazaremos el valor anterior obtenido de $\hat\beta_0$ en la ecuación.

$$\sum_{i=1}^n [\frac \partial{\hat\beta_1} (Y_i - \bar Y + \hat\beta_1 \bar X - \hat\beta_1X_i)^2] = \sum_{i=1}^n [\frac \partial{\hat\beta_1} (Y_i - \bar Y - \hat\beta_1 ( X_i - \bar X ) )^2] $$

Aplicaremos de nuevo la propiedad del cuadrado de la diferencia para transformar nuestra expresión.

$$\sum_{i=1}^n [\frac \partial{\hat\beta_1} ((Y_i - \bar Y)^2 - 2(Y_i - \bar Y)\hat\beta_1( X_i - \bar X ) + \hat\beta_1^2 ( X_i - \bar X )^2) ] $$

Derivamos cada uno de los términos de la ecuación.

$$\sum_{i=1}^n [0 - 2(Y_i - \bar Y)( X_i - \bar X ) + 2\hat\beta_1 ( X_i - \bar X )^2) ] $$

El último paso es igualar el resultado de nuestra derivada parcial en $\hat\beta_1$ a $0$.

$$-2\sum_{i=1}^n [(Y_i - \bar Y)( X_i - \bar X ) - \hat\beta_1 ( X_i - \bar X )^2] = 0$$

$$ \sum_{i=1}^n (Y_i - \bar Y)( X_i - \bar X )  - \sum_{i=1}^n \hat\beta_1 ( X_i - \bar X )^2 = 0$$

Si despejamos respecto a $\hat\beta_1$

$$ \hat\beta_1 =  \frac{\sum_{i=1}^n (Y_i - \bar Y)( X_i - \bar X )}{\sum_{i=1}^n ( X_i - \bar X )^2} $$

Multiplicando y dividiendo por $\frac 1 n$, las expresiones del numerador y denominador coinciden con las de la $Cov(X,Y)$ y la $Var(X)$ respectivamente.

$$\begin{equation} \hat\beta_1 =  \frac{\frac 1 n \sum_{i=1}^n (Y_i - \bar Y)( X_i - \bar X )}{\frac 1 n \sum_{i=1}^n ( X_i - \bar X )^2} = \frac{Cov(X,Y)}{Var(X)} \end{equation} $$

Por lo tanto, para la regresión lineal simple, los dos parámetros se pueden obtener como:

$$ \begin{equation}
\hat\beta_0 = \bar Y - \hat\beta_1 \bar X  \\
\hat\beta_1 = \frac{Cov(X,Y)}{Var(X)}
\end{equation} $$

Tras obtener el resultado para una regresión lineal, es el momento ahora de obtener la solución analítica en el caso de una regresión lineal múltiple. En este caso, en lugar de un único predictor, tendremos $n$ predictores.

$$ Y = \beta_0 + \beta_1 X + ... + \beta_p X_p + \epsilon = \beta_0 + \sum_{i=1}^p \beta_iX_i + \epsilon  $$

Los datos de entrenamiento en este caso serán $(X_{11}, X_{12}, \dots,X_{1p},Y_1), \dots ,(X_{n1}, X_{n2}, \dots,X_{np},Y_n)$, por lo que necesitaremos encontrar los valores adecuados de los parámetros desconocidos $(\beta_0, \dots, \beta_p)$.

Siguiendo el mismo razonamiento geométrico, en la regresión lineal múltiple la solución es la intersección de los $p$ hiperplanos definidos por cada uno de los predictores. Mientras que en tres dimensiones es interpretable gráficamente, para valores de $p$ mayores no es posible. Aún así, siguendo siendo aplicable la teoría de los residuos anterior en la que buscaremos minimizar la diferencia entre el valor real conocido de nuestra respuesta, $Y_i$, y el valor predicho por nuestro modelo, $\hat Y_i$.

$$ (\hat\beta_0, \dots, \hat\beta_p) = \underset{(\beta_0, \dots, \beta_p)\in\Re} {\operatorname{arg\,min\,RSS}} (\beta_0,\dots,\beta_p) := \underset{(\beta_0, \dots, \beta_p)\in\Re} {\operatorname{arg\,min}}(\sum_{i=1}^n(Y_i - \hat Y_i)^2)$$

Si consideramos que $r_i = Y_i - \hat Y_i$, siendo $r_i$ el residuo para la tupla $i$. 

$$ (\hat\beta_0, \dots, \hat\beta_p) = \underset{(\beta_0, \dots, \beta_p)\in\Re} {\operatorname{arg\,min\,RSS}} (\beta_0,\dots,\beta_p) := \underset{(\beta_0, \dots, \beta_p)\in\Re} {\operatorname{arg\,min}}(\sum_{i=1}^n r_i^2)$$

Podemos comprobar que $\sum_{i=1}^n r_i^2 = r^Tr$ ya que:

$$ r^Tr = 
\begin{bmatrix}
r_1 & r_2 & \dots & r_n \\
\end{bmatrix}	
\begin{bmatrix}
r_1 \\
r_2 \\
\vdots \\
r_n 
\end{bmatrix}	= \sum_{i=1}^n r_i^2$$

Por lo tanto, igual que en el caso de la regresión lineal simple tendremos que minimizar la expresión respecto a $\hat\beta$. Sin embargo, no resulta práctico esta expresión a la hora de operar matemáticamente con ello. Trabajaremos en su lugar con su representación matricial.

Podemos representar las ecuaciones del sumatorio en $i$ de la siguiente manera.

$$
\begin{bmatrix}
\hat Y_1 \\
\hat Y_2 \\
\vdots \\
\hat Y_n
\end{bmatrix} =

\begin{bmatrix}
1 & X_{11} & X_{12} & \dots & X_{1p}\\
1 & X_{21} & X_{22} & \dots & X_{2p}\\
\vdots & \vdots & \vdots & \ddots & \vdots \\
1 & X_{n1} & X_{n2} & \dots & X_{np}
\end{bmatrix}
\begin{bmatrix}
\hat\beta_0 \\
\hat\beta_1 \\
\vdots \\
\hat\beta_p
\end{bmatrix} +

\begin{bmatrix}
\epsilon_1 \\
\epsilon_2 \\
\vdots \\
\epsilon_n
\end{bmatrix}
$$

En versión más compacta.

$$ \boldsymbol{\hat Y = X \hat\beta + \epsilon} $$

Por lo tanto, la expresión anterior puede reescribirse en forma de matrices.

$$ (\hat\beta_0, \dots, \hat\beta_p) = \underset{(\beta_0, \dots, \beta_p)\in\Re} {\operatorname{arg\,min}}( \boldsymbol{(Y_i - \hat Y_i)^T(Y_i - \hat Y_i)})$$ 

$$ \frac \partial{\hat\beta} [\boldsymbol{(Y_i - \hat Y_i)^T(Y_i - \hat Y_i)}] = \frac \partial{\hat\beta} [\boldsymbol{(Y_i - X\hat\beta)^T(Y_i - X\hat\beta)}] = 0$$

Aplicando cálculo matricial desarrollaremos la expresión a derivar.

$$ \boldsymbol{(Y_i - X\hat\beta)^T(Y_i - X\hat\beta)=(Y_i^T - \hat\beta^TX^T)(Y_i - X\hat\beta)=} $$

$$\boldsymbol{Y_i^TY_i - Y_i^TX\hat\beta - \hat\beta^TX^TY_i + \hat\beta^TX^TX\hat\beta} $$

Teniendo en cuenta que $\boldsymbol{(Y_i^TX\hat\beta) = (\hat\beta^TX^TY_i)^T}$ y que su dimensión es igual a $(1 \times n)(n \times  p)(p\times1) = 1$, llegamos a la conclusión de que el valor de la matriz y su traspuesta es el mismo por lo que entonces se pueden suman entre sí. 

$$\boldsymbol{Y_i^TY_i - 2\hat\beta^TX^TY_i + \hat\beta^TX^TX\hat\beta} $$

Finalmente.

$$ \frac \partial{\hat\beta} [\boldsymbol{Y_i^TY_i - 2\hat\beta^TX^TY_i + \hat\beta^TX^TX\hat\beta}] = \boldsymbol{0 - 2X^TY_i + 2X^TX\hat\beta} = 0 $$

Asumiendo que la matrix $X$ es invertible, podemos calcular los parámetros $\hat\beta$ de la siguiente manera.

$$ \begin{equation} \boldsymbol{\hat\beta = (X^TX)^{-1}X^TY_i}  \label{eq:betamlr} \end{equation} $$

El cálculo de la matriz inversa para matrices de pequeña dimensión es fácil de calcular si cumple las propiedades necesarias para ser invertible. Sin embargo, cuando la dimensión de la matriz aumenta hasta miles, cientos de miles o millones de filas y columnas esta aproximación no es tan sencilla. Es por eso que es muchas veces más fácil resolve el sistema de ecuaciones $\boldsymbol{Ax=b}$ que hacer el cálculo de la inversa. 

No entraré en más detalles al respecto pero sí que os recomiendo leer el artículo ["Why Shouldn't I Invert That Matrix?" de Gregory Gundersen ](http://gregorygundersen.com/blog/2020/12/09/matrix-inversion/) y las referencias que se incluyen al artículo ["Don’t invert that matrix"](https://www.johndcook.com/blog/2010/01/19/dont-invert-that-matrix/), al artículo ["Don’t invert that matrix” – why and how"](https://www.r-bloggers.com/2015/07/dont-invert-that-matrix-why-and-how/) o a esta conversación en [Hacker News](https://news.ycombinator.com/item?id=11681893).

#### Resolución numérica

Dado que la aproximación analítica prácticamente es solo útil en el caso de la regresión lineal simple, es necesario buscar una alternativa para problemas de regresión lineal más complejos. Partiendo de la expresión $\eqref{eq:betamlr}$ que obtuvimos en el apartado anterior, podemos representarla en la forma general $\boldsymbol{Ax=b}$. De esta manera, nuestro problema quedaría enunciado de la siguiente manera.

$$ \boldsymbol{(X^TX)\hat\beta = X^TY_i} $$

Existen diferentes técnicas a la hora de resolver este sistema de ecuaciones. No entraremos en detalle de todas ellas en este artículo ya que nos saldríamos del objetivo principal y entraríamos en el territorio del álgebra lineal. Sin embargo, es interesante hacer una revisión de cómo lo resuelve uno de los frameworks más utilizados en el área del aprendizaje automático: [Scikit-learn](https://scikit-learn.org/stable/modules/generated/sklearn.linear_model.LinearRegression.html#sklearn.linear_model.LinearRegression).

En este caso concreto, el modelo que expone Scikit-learn envuelve el método de cálculo del problema *Ordinary Least Squares* dentro del paquete de álgebra lineal de [SciPy](https://docs.scipy.org/doc/scipy/reference/generated/scipy.linalg.lstsq.html) (scipy.linalg.lstsq). Si revisamos la documentación de SciPy de éste método, podemos ver que su objetivo es calcular la solución de mínimos cuadrados de la ecuación $\boldsymbol{Ax=b}$ obteniendo el vector $x$ y minimizando su norma euclídea, $\|\|b-Ax\|\|_2$.

Para realizar esta minimización, este método de SciPy recubre a su vez la implementación en Fortran 90 de la librería [LAPACK](http://performance.netlib.org/lapack/) para [problemas líneales de mínimos cuadradros](https://www.netlib.org/lapack/lug/node27.html#tabdrivellsq). El método *scipy.linalg.lstsq* permite escoger entre tres posibles opciones a la hora de realizar el cálculo:

- ***[gelss](http://www.netlib.org/lapack/explore-html/d7/d3b/group__double_g_esolve_gaa6ed601d0622edcecb90de08d7a218ec.html#gaa6ed601d0622edcecb90de08d7a218ec)***: emplea la descomposición en valores singulares de la matriz $A$.
- ***[gelsd](http://www.netlib.org/lapack/explore-html/d7/d3b/group__double_g_esolve_ga94bd4a63a6dacf523e25ff617719f752.html#ga94bd4a63a6dacf523e25ff617719f752)***: emplea también la descomposición en valores singulares de la matriz $A$ pero en este caso con un algoritmo de divide y vencerás.
- ***[gelsy](http://www.netlib.org/lapack/explore-html/d7/d3b/group__double_g_esolve_ga385713b8bcdf85663ff9a45926fac423.html#ga385713b8bcdf85663ff9a45926fac423)***: emplea una factorización ortogonal completa de la matriz $A$

Si queréis seguir profundizando más en los detalles específicos os recomiendo revisar cada uno de los artículos de la documentación enlazados anteriormente. Por cada función incluyen una breve introducción donde explican a alto nivel la implementación. Si no es suficiente y existe alguna duda, el siguiente paso sería revisar un buen libro de álgebra lineal.

