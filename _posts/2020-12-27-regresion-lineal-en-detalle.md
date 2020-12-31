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

Llegamos al punto en el que disponemos de nuestros datos de entrada y hemos verificado que cumplen las propiedades necesarias para que los resultados de la regresión lineal tengan sentido. ¿Cómo obtenemos entonces ahora los coeficientes asociados a nuestros predictores solución de nuestro modelo de regresión lineal?. Desde el punto de vista matemático, contamos con dos opciones para ello: la *resolución analítica* o la *resolución numérica*. 

En el primer escenario, buscamos una ecuación o conjunto de ecuaciones que nos permitan calcular la solución del problema de regresión lineal para cualquier valor de los predictores que empleemos. De esta manera, únicamente será necesario reemplazar los valores de nuestro problema específico en el conjunto de ecuaciones y tendremos la solución. En el segundo, por el contrario, en lugar de buscar una serie de ecuaciones universales nos centraremos en obtener la solución para los valores específicos de nuestros predictores. 

Comparando ambas opciones, parece que el camino más adecuado es el primero, obtener una solución analítica que nos permita obtener la solución en cualquier escenario que nos encontremos. Sin embargo, mientras que esto es sencillo para el escenario de una regresión lineal simple donde únicamente tenemos un predictor, la situación se complica cuando necesitamos resolver una regresión lineal múltiple en la que el número de predictores aumenta. En estos casos es posible que el coste computacional para obtener la solución sea tan alto que necesitemos resolverlo numéricamente de forma más eficiente.

Veamos a continuación los detalles de ambos casos:

#### Resolución analítica

Comenzaremos en primer lugar para el caso de la regresión lineal simple. Como hemos visto anteriormente, la solución se puede representar de la siguiente manera:

$$ Y = \beta_0 + \beta_1 X + \epsilon $$

Si los datos que emplearemos para nuestro modelo se pueden definir como el conjunto de tuplas $$(X_1,Y_1),...(X_n,Y_n)$$ en el que para cada valor de $$X_i$$ conocemos el valor de la respuesta $$Y_i$$, llegamos a un conjunto de ecuaciones de necesitamos resolver para los parámetros desconocidos $$\beta_0,\beta_1$$. Utilizando el razonamiento geométrico, la regresión lineal se puede representar como la línea recta que minimiza la distancia a cada uno de las tuplas de entrada de nuestro problema. Esta distancia se puede representar como la diferencia entre el valor real $$Y_i$$ del que disponemos y el valor predicho $$\hat Y_i$$. Esta diferencia se conoce como la suma de los residuos, al ser una diferencia generalmente se empleará dicha suma al cuadrado dando lugar a su nombre en inglés de RSS *(Residual Sum of Squares)*.

Por lo tanto, nuestra estimación de los valores reales de $$(\beta_0, \beta_1)$$ se representará como $$(\hat\beta_0,\hat\beta_1)$$. Siendo estos los dos valores concretos que minimizan el resultado de *RSS*:


$$ (\hat\beta_0,\hat\beta_1) = \underset{(\beta_0,\beta_1)\in\Re} {\operatorname{arg\,min\,RSS}} (\beta_0,\beta_1) := \underset{(\beta_0,\beta_1)\in\Re} {\operatorname{arg\,min}}(\sum_{i=1}^n(Y_i - \hat Y_i)^2)$$

Si reemplazamos $$\hat Y_i$$ por su definición, nuestro problema quedaría definido como: 

$$ (\hat\beta_0,\hat\beta_1) = \underset{(\beta_0,\beta_1)\in\Re} {\operatorname{arg\,min}}(\sum_{i=1}^n(Y_i - \beta_0 - \beta_1X)^2)  $$

Nos encontramos ante un problema de optimización en el que buscamos el valor mínimo de la ecuación. Del cálculo numérico sabemos que el mínimo o máximo de una función se da cuando la derivada de la misma es igual a $$0$$. Por lo tanto, el mínimo de dicha expresión para cada uno de nuestros dos parámetros $$(\beta_0, \beta_1)$$ será:

$$ \frac \partial \beta_0 [\sum_{i=1}^n(Y_i - \beta_0 - \beta_1X)^2)] = 0$$

$$ \frac \partial \beta_1 [\sum_{i=1}^n(Y_i - \beta_0 - \beta_1X)^2)] = 0$$

El siguiente paso será resolver cada una de estas expresiones por separado. En primer lugar, la derivada en $\beta_0$ será ya que la derivada es una operación lineal por lo que la derivada de una suma es igual a la suma de sus derivadas:
 
$$\frac \partial \beta_0 [\sum_{i=1}^n(Y_i - \beta_0 - \beta_1X)^2)] =  \sum_{i=1}^n [\frac \partial \beta_0 (Y_i - \beta_0 - \beta_1X)^2)] $$

Para simplificar los siguiente pasos, haremos la siguiente sustitución en la ecuación $$C=Y_i -\beta_1X_i$$, quedándonos que:

$$\sum_{i=1}^n [\frac \partial \beta_0 (C - \beta_0)^2)] $$

Aplicando la propiedad del cuadrado de la diferencia que indica que $$(a-b)^2 = a^2 - 2ab + b^2$$, nuestra ecuación se transforma en:

$$\sum_{i=1}^n [\frac \partial \beta_0 (C^2 -2C\beta_0 + \beta_0^2)] $$

Derivando cada uno de los términos de la ecuación nos queda que:

$$\sum_{i=1}^n [\frac \partial \beta_0 (C^2) - \frac \partial \beta_0 (2C\beta_0) + \frac \partial \beta_0(\beta_0^2)] = $$
$$\sum_{i=1}^n [ 0 - 2C + 2\beta_0] $$

Deshaciendo el cambio anterior de $$C=Y_i -\beta_1X_i$$, llegamos a que:

$$-2\sum_{i=1}^n(C) + 2\sum_{i=1}^n(\beta_0) = -2\sum_{i=1}^n(Y_i -\beta_1X_i) + 2n\beta_0$$

Obteniendo finalmente que:

$$2(n\beta_0 -\sum_{i=1}^n(Y_i) + \beta_1\sum_{i=1}^n(X_i))$$

Nos queda por lo tanto igualar el resultado de nuestra derivada parcial en $$\beta_0$$ a $$0$$:

$$2(n\beta_0 -\sum_{i=1}^n(Y_i) + \beta_1\sum_{i=1}^n(X_i)) = 0$$

$$\beta_0 = \frac 1 n {\sum_{i=1}^n(Y_i)} - \frac 1 n \beta_1\sum_{i=1}^n(X_i) $$

Siendo $$\frac 1 n {\sum_{i=1}^n(A_i)}$$ la expresión de la media aritmética podemos resumir el resultado en:

$$\beta_0 = \hat Y - \beta_1 \hat X $$

El siguiente paso será hacer lo mismo para resolverlo en $$\beta_1$$

$$\frac \partial \beta_1 [\sum_{i=1}^n(Y_i - \beta_0 - \beta_1 X_i)^2] =  \sum_{i=1}^n [\frac \partial \beta_1 (Y_i - \beta_0 - \beta_1X_i)^2] $$

Si reemplazamos el valor de $$\beta_0$$ nos quedará que:

$$\sum_{i=1}^n [\frac \partial \beta_1 (Y_i - \hat Y - \beta_1 \hat X - \beta_1X_i)^2] = \sum_{i=1}^n [\frac \partial \beta_1 (Y_i - \hat Y - \beta_1 (\hat X - X_i) )^2] $$

Aplicando de nuevo la propiedad del cuadrado de la diferencia nuestra ecuación se transforma en:

$$\sum_{i=1}^n [\frac \partial \beta_1 ((Y_i - \hat Y)^2 - 2(Y_i - \hat Y)\beta_1(\hat X - X_i) + \beta_1^2 (\hat X - X_i)^2) ] $$

Derivando cada uno de los términos de la ecuación nos queda que:

$$\sum_{i=1}^n [0 - 2(Y_i - \hat Y)(\hat X - X_i) + 2\beta_1 (\hat X - X_i)^2) ] $$

Nos queda por lo tanto igualar el resultado de nuestra derivada parcial en $$\beta_1$$ a $$0$$:

$$-2\sum_{i=1}^n [(Y_i - \hat Y)(\hat X - X_i) - \beta_1 (\hat X - X_i)^2] = 0$$

Obteniéndose que

$$ \sum_{i=1}^n [(Y_i - \hat Y)(\hat X - X_i)]  - \sum_{i=1}^n [\beta_1 (\hat X - X_i)^2] = 0$$

Si despejamos respecto a $$\beta_1$$

$$ \beta_1 =  \frac{\sum_{i=1}^n [(Y_i - \hat Y)(\hat X - X_i)]}{\sum_{i=1}^n [(\hat X - X_i)^2]} $$

Multiplicando y dividiendo por $$\frac 1 n$$ nos quedará que

$$ \beta_1 =  \frac{\frac 1 n \sum_{i=1}^n [(Y_i - \hat Y)(\hat X - X_i)]}{\frac 1 n \sum_{i=1}^n [(\hat X - X_i)^2]} = \frac{Cov(X,Y)}{Var(X)} $$


#### Resolución numérica


### ¿Qué conclusiones puedo extraer a partir de los resultados?


### ¿Qué confianza puedo tener en los valores obtenidos?

### ¿Es bueno mi modelo o no?

### De todos los modelos, ¿cuál es el que debo de escoger?