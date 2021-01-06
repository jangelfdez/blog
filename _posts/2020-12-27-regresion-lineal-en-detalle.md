---
layout: post
title: "Regresión lineal en detalle, los primeros pasos en Machine Learning"
author: "José Ángel Fernández"
categories: blog
tags: [machine learning, regresion lineal]
image: machinelearning.jfif
---

### ¿Qué problemas son candidatos para una regresión lineal?

Y = respuesta
X = predictores

### ¿Puedo utilizar una regresión lineal con los datos que tengo?

Una regresión lineal asume que los datos cumplen una serie de propiedades probabilísticas que nos permitan extraer conclusiones válidas a partir de los resultados obtenidos. Es decir, que las predicciones que realicemos con el modelo se ajusten a nuestro escenario real y tengan algún sentido.

Esto se debe a que los datos con los que contamos son una muestra de todos los posibles valores existentes de nuestro escenario completo. Si los datos no cumplen esas propiedades probabilísticas, no podremos extrapolar los resultados obtenidos a partir de la muestra a todo el conjunto completo de datos.

Las propiedades que tienen que cumplirse son: linealidad, homocedasticidad, normalidad e independencia de los errores. Pero, ¿qué implica cada una de ellas?

#### Linealidad

Cuando hablamos de *linealidad*, nos referimos a que la variable que predecimos debe de mantener una relación lineal con cada una de las variables predictoras. Èxpresado de forma matemática, la esperanza de la variable predicha equivle a una combinación lineal de las predictoras.

$$ E[Y | X_1 = x_1, ..., X_p = x_p] = \beta_0 + \beta_1 x_i + ... + \beta_p x_p ] $$

#### Homocedasticidad

#### Normalidad

#### Independencia de los errores



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


El cálculo de la matriz inversa para matrices de pequeña dimensión es fácil de calcular si cumple las propiedades necesarias para ser invertible. Sin embargo, cuando la dimensión de la matriz aumenta hasta miles, cientos de miles o millones de filas y columnas esta aproximación no es tan sencilla. Es por eso que es muchas veces más fácil resolve el sistema de ecuaciones $\boldsymbol{AX=B}$ que hacer el cálculo de la inversa. 

No entraré en más detalles al respecto pero sí que os recomiendo leer el artículo ["Why Shouldn't I Invert That Matrix?" de Gregory Gundersen ](http://gregorygundersen.com/blog/2020/12/09/matrix-inversion/) y las referencias que se incluyen al artículo ["Don’t invert that matrix"](https://www.johndcook.com/blog/2010/01/19/dont-invert-that-matrix/), al artículo ["Don’t invert that matrix” – why and how"](https://www.r-bloggers.com/2015/07/dont-invert-that-matrix-why-and-how/) o a esta conversación en [Hacker News](https://news.ycombinator.com/item?id=11681893).

#### Resolución numérica



### ¿Qué conclusiones puedo extraer a partir de los resultados?


### ¿Qué confianza puedo tener en los valores obtenidos?

### ¿Es bueno mi modelo o no?

### De todos los modelos, ¿cuál es el que debo de escoger?