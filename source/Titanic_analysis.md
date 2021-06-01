---
title: "Limpieza y análisis del dataset TITANIC"
author: "Ander Elkoroaristizabal"
date: "Mayo 2021"
output: 
    html_document:
        highlight: default
        number_sections: no
        toc: yes
        toc_depth: 3
        df_print: kable
        keep_md: true
    pdf_document: 
        toc: yes
editor_options: 
  chunk_output_type: console
---



# 1. Descripción del dataset

El conjunto de datos que estudiaremos es el [*Titanic dataset*](https://www.kaggle.com/c/titanic/data). 
El objetivo primordial de este *dataset* es la creación de un modelo que prediga que pasajeros sobrevivieron al hundimiento del Titanic.

Este dataset resulta especialmente atractivo por lo interesante del tema, 
la variedad en las variables y la cantidad de estudios y discusiones sobre él que podemos encontrar, 
incluso en la propia [página de discusiones de la competición en Kaggle](https://www.kaggle.com/c/titanic/discussion). 

El conjunto de datos "completo" tal y como se incluye en [Kaggle](https://www.kaggle.com) viene ya dividido en dos subconjuntos: 
uno de entrenamiento, denominado "train.csv" y otro de evaluación, denominado "test.csv".
Ambos conjuntos deben pasar por los pasos de preprocesado, 
pero sólo podremos utilizar el primer conjunto para análisis,
dado que al segundo le falta la columna `Survived` que se debe predecir.
Esto se debe a que sobre este segundo conjunto está pensado para que efectuemos nuestras predicciones sobre él y subamos estas predicciones a la [competición](https://www.kaggle.com/c/titanic/overview) a la que pertenece.

La hoja de ruta es la siguiente:

a. Limpiaremos ambos conjuntos de datos de manera conjunta, 
pero distinguiendo el subconjunto a la hora de hacer inputaciones. 

b. Analizaremos el conjunto de entrenamiento. 

c. Crearemos también un modelo de clasificación con el conjunto de entrenamiento, 
y usaremos la validación cruzada para evaluar su rendimiento. 

d. Evaluaremos también el modelo creado sobre el conjunto de test, 
utilizando para ello la plataforma [Kaggle](https://www.kaggle.com).

El conjunto contiene 10 variables además de la variable respuesta `survived`, 
con 891 registros en el conjunto de entrenamiento y 418 registros en el conjunto de evaluación.

A continuación mostramos el diccionario de datos:

| Variable  | Definición    | Claves    |
|-----------|---------------|-----------|
| survival  | Variable binaría indicadora de la supervivencia.   | 1 = Sí, 0 = No.
| pclass    | Clase del billete de embarque.  | 1 = Primera clase, 2 = Segunda clase, 3&nbsp;=&nbsp;Tercera clase
| sex       | Sexo.   |
| Age       | Edad, en años.              |
|sibsp|Número de hermanos/esposas también en el Titanic.|
|parch|Número de padres/hijos también en el Titanic.|
|ticket|Código alfanumérico del billete.|
|fare| Precio del billete. |
|cabin| Código alfanumérico de cabina. |
|embarked| Puerto de embarque. | C = Cherbourg, Q = Queenstown, S&nbsp;=&nbsp;Southampton

# 2. Integración y selección de los datos de interés a analizar.

Como primer paso de la integración unimos ambos conjuntos de datos 
entrenamiento y evaluación en uno sólo, 
sobre el cual efectuaremos la limpieza. 
Dado que los ficheros originales los guardaremos inalterados no hace falta identificar de que fichero viene cada registro, 
por lo que los podemos fundir de manera vertical tras definir la columna `survived` como `NA` en el conjunto de evaluación:


```r
train_df <- read.csv("data/train.csv")
test_df <- read.csv("data/test.csv")
test_df['Survived'] = NA
df = rbind(train_df, test_df)
```

Verificamos la unicidad de los atributos individuales del conjunto unificado, 
es decir todos los atributos excepto el identificador. 
Como podemos ver todos los registros son únicos:


```r
any(duplicated(df[,-1]))
```

```
## [1] FALSE
```

Respecto a la selección de datos:

+ Como no nos interesa restringirnos a un grupo en particular mantendremos en este punto todos los registros. 

+ Nos quedaremos también todas las variables, 
dado que todas parece que puedan estar relacionadas con el objetivo, 
y eliminaremos las que no nos sean útiles más adelante en un proceso de selección de variables.

# 3. Limpieza de datos

## Busqueda de valores nulos

Como primer punto de la limpieza de datos y de cara a identificar los valores desconocidos que pueda haber no identificados como tales (como `NA`), 
analizaremos la estructura del conjunto de datos, 
haremos las conversiones de variables necesarias 
y obtendremos una descripción preliminar de los datos. 

La estructura de los datos tal y como los ha leido `R` es como sigue:

```r
str(df)
```

```
## 'data.frame':	1309 obs. of  12 variables:
##  $ PassengerId: int  1 2 3 4 5 6 7 8 9 10 ...
##  $ Survived   : int  0 1 1 1 0 0 0 0 1 1 ...
##  $ Pclass     : int  3 1 3 1 3 3 1 3 3 2 ...
##  $ Name       : chr  "Braund, Mr. Owen Harris" "Cumings, Mrs. John Bradley (Florence Briggs Thayer)" "Heikkinen, Miss. Laina" "Futrelle, Mrs. Jacques Heath (Lily May Peel)" ...
##  $ Sex        : chr  "male" "female" "female" "female" ...
##  $ Age        : num  22 38 26 35 35 NA 54 2 27 14 ...
##  $ SibSp      : int  1 1 0 1 0 0 0 3 0 1 ...
##  $ Parch      : int  0 0 0 0 0 0 0 1 2 0 ...
##  $ Ticket     : chr  "A/5 21171" "PC 17599" "STON/O2. 3101282" "113803" ...
##  $ Fare       : num  7.25 71.28 7.92 53.1 8.05 ...
##  $ Cabin      : chr  "" "C85" "" "C123" ...
##  $ Embarked   : chr  "S" "C" "S" "S" ...
```

Como podemos ver hay muchas variables cuyo tipo debemos cambiar: 

+ `Survived`, `Sex` y `Cabin` (dado que mucha gente comparte cabina), y `Embarked` deberían ser factores.

+ `Ticket` también debería ser un factor, dado que no es único, 
si no que hay pasajeros que comparten un mismo ticket (y no es porque sean `NA`o `''`):


```r
table(duplicated(df['Ticket']))
```

```
## 
## FALSE  TRUE 
##   929   380
```

```r
head(df[duplicated(df['Ticket']), c('Ticket')])
```

```
## [1] "349909"       "CA 2144"      "19950"        "11668"        "347082"      
## [6] "S.O.C. 14879"
```

+ `Pclass` debería ser un factor ordenado donde el nivel más bajo fuese 3 y el más alto 1.

+ `SibSp` y `Parch` deberían ser enteros, si bien este cambio no suele impactar demasiado en los análisis.

Mantendremos `PassengerId` como número, 
siendo conscientes de que es en realidad la clave primaria de nuestros datos.

Hacemos las conversiones necesarias:


```r
df[, c('Survived', 'Sex', 'Ticket', 'Cabin', 'Embarked')] = 
  lapply(df[, c('Survived', 'Sex', 'Ticket', 'Cabin', 'Embarked')], factor)
df[, 'Pclass'] = factor(df[,'Pclass'], ordered = TRUE, levels = c(3,2,1))
df[, c('SibSp', 'Parch')] = lapply(df[, c('SibSp', 'Parch')], as.integer)
```

Analizamos ahora por encima los datos que tenemos según su tipo.

Comenzamos por las variables numéricas:


```r
summary(Filter(is.numeric, df))
```

```
##   PassengerId        Age            SibSp            Parch      
##  Min.   :   1   Min.   : 0.17   Min.   :0.0000   Min.   :0.000  
##  1st Qu.: 328   1st Qu.:21.00   1st Qu.:0.0000   1st Qu.:0.000  
##  Median : 655   Median :28.00   Median :0.0000   Median :0.000  
##  Mean   : 655   Mean   :29.88   Mean   :0.4989   Mean   :0.385  
##  3rd Qu.: 982   3rd Qu.:39.00   3rd Qu.:1.0000   3rd Qu.:0.000  
##  Max.   :1309   Max.   :80.00   Max.   :8.0000   Max.   :9.000  
##                 NA's   :263                                     
##       Fare        
##  Min.   :  0.000  
##  1st Qu.:  7.896  
##  Median : 14.454  
##  Mean   : 33.295  
##  3rd Qu.: 31.275  
##  Max.   :512.329  
##  NA's   :1
```

Podemos ver que sólo a un registro le falta información de `Fare`. 
Por otra parte también vemos que 263 registros no tienen la edad informada.
Esto sí que es un posible problema que necesitaremos subsanar, 
dado que una posible política de supervivencia prioritaria aplicada durante el hundimiento del Titanic podría ser "Mujeres y niños primero".
Del resto de variables sólo la cantidad de ceros en `Parch` nos hace pensar que puedan ser valores desconocidos, 
pero teniendo en cuenta el significado de la variable parece normal.

Estudiamos también los factores:


```r
summary(Filter(is.factor, df))
```

```
##  Survived   Pclass      Sex           Ticket                 Cabin     
##  0   :549   3:709   female:466   CA. 2343:  11                  :1014  
##  1   :342   2:277   male  :843   1601    :   8   C23 C25 C27    :   6  
##  NA's:418   1:323                CA 2144 :   8   B57 B59 B63 B66:   5  
##                                  3101295 :   7   G6             :   5  
##                                  347077  :   7   B96 B98        :   4  
##                                  347082  :   7   C22 C26        :   4  
##                                  (Other) :1261   (Other)        : 271  
##  Embarked
##   :  2   
##  C:270   
##  Q:123   
##  S:914   
##          
##          
## 
```

Vemos por una parte los 418 valores desconocidos que ya conocíamos en la variable `Survived`. 
También vemos dos valores `NA` encubiertos en las variables `Cabin` y `Embarked` que convertimos en `NA`, tras lo cual eliminamos el nivel que les corresponde:


```r
# Cabin
df[df['Cabin']=='', 'Cabin']  = NA
df[,'Cabin'] = droplevels(df[,'Cabin'])
# Embarked
df[df['Embarked']=='', 'Embarked']  = NA
df[,'Embarked'] = droplevels(df[,'Embarked'])
```

## Reconocimiento de valores extremos

Estudiamos ahora los posibles valores extremos que pueda haber en el conjunto de datos. 
Realizamos para ello un diagrama de caja de cada una de las variables numéricas: 


```r
library(ggplot2)
PrettyBoxplot= function(var1) {
  bp = ggplot(df, aes(y = get(var1))) + geom_boxplot(outlier.colour="blue") + 
    ggtitle(paste("Boxplot de la variable", var1)) + ylab("") +
    theme(plot.title = element_text(hjust = 0.5))
  return(bp)
}

gridExtra::grid.arrange(grobs = lapply(colnames(Filter(is.numeric, df[,-1])), PrettyBoxplot))
```

<img src="Titanic_analysis_files/figure-html/unnamed-chunk-9-1.png" style="display: block; margin: auto;" />

### Variable `Age`

El diagrama de caja de la variable `Age` muestra que hubo personas de edad avanzada en el Titanic, pero no muestra ningún valor que pudiese considerarse extremo.

### Variable `SibSp`

El diagrama de caja de la variable `SibSp` muestra que el 75% de los embarcados en el Titanic venían solos o acompañados de un sólo hermano o hermana. 
Como consecuencia todos los valores más grandes que 3 son numéricamente valores extremos. 
<!-- Para identificar si realmente son valores erróneos que debamos tratar jugamos también con la variable `Parch`, dada la relación entre ambas, especialmente para los valores más elevados: los registros con una cantidad elevada de hermanos es probable que viajen en familia junto a sus padres, por lo que debería haber cierta correspondencia (registro con $n$ hermanos) -> (uno o dos registros con $n$ hijos).  -->
Para identificar si realmente son valores erróneos que debemos tratar podemos estudiar cuanta gente con más de 3 hermanos comparte el mismo apellido:


```r
# Filtramos los valores extremos
ManySiblings = df[df['SibSp']>3,]
# Obtenemos el apellido
ManySiblings[, 'Surname'] = sapply(ManySiblings[, 'Name'],
                                   function(x){strsplit(x, ",")[[1]][1]})
# Obtenemos la tabla:
table(ManySiblings[,c('SibSp', 'Surname')])
```

```
##      Surname
## SibSp Andersson Asplund Goodwin Panula Rice Sage
##     4         7       5       0      5    5    0
##     5         0       0       6      0    0    0
##     8         0       0       0      0    0    9
```
Como podemos ver en todos los grupos excepto en el de los apellidados Andersson hay $n+1$ hermanos, donde $n$ es el número de hermanos índicados por todos ellos. Consideramos por lo tanto que todos estos valores son correctos.
Estudiamos con más detalle la familia la familia Andersson:


```r
ManySiblings[ManySiblings['Surname']=='Andersson',]
```

<div class="kable-table">

|     | PassengerId|Survived |Pclass |Name                                    |Sex    | Age| SibSp| Parch|Ticket  |   Fare|Cabin |Embarked |Surname   |
|:----|-----------:|:--------|:------|:---------------------------------------|:------|---:|-----:|-----:|:-------|------:|:-----|:--------|:---------|
|69   |          69|1        |3      |Andersson, Miss. Erna Alexandra         |female |  17|     4|     2|3101281 |  7.925|NA    |S        |Andersson |
|120  |         120|0        |3      |Andersson, Miss. Ellis Anna Maria       |female |   2|     4|     2|347082  | 31.275|NA    |S        |Andersson |
|542  |         542|0        |3      |Andersson, Miss. Ingeborg Constanzia    |female |   9|     4|     2|347082  | 31.275|NA    |S        |Andersson |
|543  |         543|0        |3      |Andersson, Miss. Sigrid Elisabeth       |female |  11|     4|     2|347082  | 31.275|NA    |S        |Andersson |
|814  |         814|0        |3      |Andersson, Miss. Ebba Iris Alfrida      |female |   6|     4|     2|347082  | 31.275|NA    |S        |Andersson |
|851  |         851|0        |3      |Andersson, Master. Sigvard Harald Elias |male   |   4|     4|     2|347082  | 31.275|NA    |S        |Andersson |
|1106 |        1106|NA       |3      |Andersson, Miss. Ida Augusta Margareta  |female |  38|     4|     2|347091  |  7.775|NA    |S        |Andersson |

</div>

Como podemos ver hay dos registros que destacan por edad y por ticket, 
los registros 69 y 1106, y que además no están casadas, como indica el título 'Miss.'.
Consideramos por lo tanto que el atributo `SibSp` de estos dos registros es erróneo, y lo convertimos a `NA`:


```r
df[c(69, 1106), 'SibSp'] = NA
```

### Variable `Parch`

El diagrama de caja de la variable `Parch` muestra que numéricamente todos los valores no cero son valores extremos. 
Estudiamos más en detalle aquellos registros con más de 4 descendientes en el Titanic, 
y vemos si lo podemos corresponder con las familias de hermanos recién identificadas:


```r
# Filtramos los valores extremos
ManyDescendants = df[df['Parch']>4,]
# Obtenemos el apellido
ManyDescendants[, 'Surname'] = sapply(ManyDescendants[, 'Name'],
                                      function(x){strsplit(x, ",")[[1]][1]})
# Obtenemos la tabla:
table(ManyDescendants[,c('Parch', 'Surname')])
```

```
##      Surname
## Parch Andersson Asplund Goodwin Panula Rice Sage
##     5         2       2       0      1    1    0
##     6         0       0       2      0    0    0
##     9         0       0       0      0    0    2
```
Como podemos ver se corresponde perfectamente con las familias identificadas previamente, una vez corregidas los dos registros 'Andersson' erróneos, 
por lo que no son realmente valores extremos.

### Variable `Fare`

En el diagrama de la variable `Fare` vemos que hay muchos valores que numéricamente son valores extremos 
En este caso nos limitamos a verificar que todos los registros con tarifa muy elevada son de primera clase:


```r
all(na.omit(df[df['Fare']>200,'Pclass'] == 1))
```

```
## [1] TRUE
```

Hemos utilizado la función `na.omit()` debido al registro con `Fare` desconocido que ya conocemos. 
Vemos que obviado este valor todos los registros corresponden efectivamente a personas que viajaron en primera clase, por lo que damos los valores de `Fare` por válidos.

## Tratamiento de valores nulos

Recapitulando tenemos que tratar

+ 263 valores perdidos en la variable `Age`,

+ 2 valores perdidos en la variable `SibSp`

+ 1 valor perdido en la varaible `Fare`,

+ 1014 valores perdidos en la variable `Cabin`, y

+ 2 valores perdidos en la variable `Embarked`. 

Es importante notar que todas las medidas que tomemos en este apartado para tratar los valores perdidos deberán tener en cuenta a que subconjunto pertenece cada registro.
El motivo es que de lo contrario podríamos estar filtrando información sobre datos de evaluación al modelo,
si por ejemplo utilizasemos el conjunto completo para inputar un valor en un registro de entrenamiento.

### Variable `Cabin`

El caso de la variable `Cabin` es el más claro:
tanto la variedad de la variable como la cantidad de registros perdidos (casi 3/4)
nos obligan a eliminarla de ambos subconjuntos:


```r
df = df[, names(df) != 'Cabin']
```

Hecho esto resulta interesante estudiar las posibles combinaciones de valores desconocidos, 
dado que esto dificultara las posibles inputaciones que queramos hacer.
Afortunadamente no hay registros con más de un `NA`:


```r
VIM::aggr(Filter(anyNA, df[,-2]), combine = TRUE, bars = FALSE)
```

<img src="Titanic_analysis_files/figure-html/unnamed-chunk-16-1.png" style="display: block; margin: auto;" />

### Variable `Embarked`

Observemos los registros:


```r
df[is.na(df['Embarked']),]
```

<div class="kable-table">

|    | PassengerId|Survived |Pclass |Name                                      |Sex    | Age| SibSp| Parch|Ticket | Fare|Embarked |
|:---|-----------:|:--------|:------|:-----------------------------------------|:------|---:|-----:|-----:|:------|----:|:--------|
|62  |          62|1        |1      |Icard, Miss. Amelie                       |female |  38|     0|     0|113572 |   80|NA       |
|830 |         830|1        |1      |Stone, Mrs. George Nelson (Martha Evelyn) |female |  62|     0|     0|113572 |   80|NA       |

</div>

Vemos que ambos pertenecen al subconjunto de entrenamiento (dado que tienen la variable `Survived` informada).
También vemos que compartían ticket, pero nadie más lo compartia con ellas:


```r
df[df['Ticket'] == 113572,]
```

<div class="kable-table">

|    | PassengerId|Survived |Pclass |Name                                      |Sex    | Age| SibSp| Parch|Ticket | Fare|Embarked |
|:---|-----------:|:--------|:------|:-----------------------------------------|:------|---:|-----:|-----:|:------|----:|:--------|
|62  |          62|1        |1      |Icard, Miss. Amelie                       |female |  38|     0|     0|113572 |   80|NA       |
|830 |         830|1        |1      |Stone, Mrs. George Nelson (Martha Evelyn) |female |  62|     0|     0|113572 |   80|NA       |

</div>

Como no tenían familia a bordo tampoco parece que utilizar sus apellidos vaya a ser una buena opción, 
por lo que decidimos descartar ambos registros:


```r
df = df[-c(62, 830),]
```

### Variable `Fare`.

Observemos el registro:


```r
df[is.na(df['Fare']),]
```

<div class="kable-table">

|     | PassengerId|Survived |Pclass |Name               |Sex  |  Age| SibSp| Parch|Ticket | Fare|Embarked |
|:----|-----------:|:--------|:------|:------------------|:----|----:|-----:|-----:|:------|----:|:--------|
|1044 |        1044|NA       |3      |Storey, Mr. Thomas |male | 60.5|     0|     0|3701   |   NA|S        |

</div>

Notamos que en este caso el registro correspondende al subconjunto de evaluación.
Como el registro pertenece a la tercera clase y no tiene familia embarcada le inputamos la media de tarifas de este subgrupo dentro del subconjunto de evaluación:


```r
df[is.na(df['Fare']),'Fare']= 
  mean(df[df['Pclass'] == 3 & df['SibSp'] == 0 & df['Parch'] == 0 & is.na(df['Survived']), 'Fare'], na.rm = TRUE)
```

### Variable `Age`

Finalmente llegamos a la variable más complicada, `Age`. 
El número de registros sin edad informada hace inviable eliminarlos todos, 
por lo que lo más sensato parece tratar de inputarles valores.
Utilizaremos para ello una estrategia parecida a la utilizada en [este tutorial de Kaggle](https://www.kaggle.com/allohvk/titanic-missing-age-imputation-tutorial-advanced), 
inputando los valores de `Age` estratificando los registros según su titulo (aún por obtener).
Deberemos tener en cuenta también que los registros sin edad informada pertenecen a ambos subconjuntos, 
con 177 registros de entrenamiento y 86 de evaluación:


```r
table(is.na(df[is.na(df['Age']),'Survived']))
```

```
## 
## FALSE  TRUE 
##   177    86
```

El "Título" lo extraemos del nombre usando expresiones regulares (*RegEx*), y convertimos a factor:


```r
df[, 'Title'] = sapply(df[, 'Name'], function(x){trimws(strsplit(x, "[,.] ")[[1]][2])})
df[, 'Title'] = factor(df[, 'Title'])
```

El código que tenemos encima divide el nombre de cada registro en 3, 
dividiendo tanto por coma como por punto y elimina los espacios a ambos extremos.
La distribución de títulos en los registros con edad informada y aquellos en los que no es como sigue, 
de manera diferenciada entre entrenamiento y evaluación:


```r
# Entrenamiento
table(droplevels(df[!is.na(df['Age']) & !is.na(df['Survived']), 'Title']))
```

```
## 
##         Capt          Col          Don           Dr     Jonkheer         Lady 
##            1            2            1            6            1            1 
##        Major       Master         Miss         Mlle          Mme           Mr 
##            2           36          145            2            1          398 
##          Mrs           Ms          Rev          Sir the Countess 
##          107            1            6            1            1
```

```r
table(droplevels(df[is.na(df['Age']) & !is.na(df['Survived']), 'Title']))
```

```
## 
##     Dr Master   Miss     Mr    Mrs 
##      1      4     36    119     17
```

```r
# Evaluación
table(droplevels(df[!is.na(df['Age']) & is.na(df['Survived']), 'Title']))
```

```
## 
##    Col   Dona     Dr Master   Miss     Mr    Mrs    Rev 
##      2      1      1     17     64    183     62      2
```

```r
table(droplevels(df[is.na(df['Age']) & is.na(df['Survived']), 'Title']))
```

```
## 
## Master   Miss     Mr    Mrs     Ms 
##      4     14     57     10      1
```

Como vemos los títulos de aquellos registros sin edad informada son suficientemente comunes como para ser utilizados en la inputación.
La excepción es el registro con título "Ms" del conjunto de evaluación. 
Tampoco podemos vincularla al una sola categoría de las más pobladas, 
si no que [podría pertencer tanto a la categoría "Miss" como "Mrs",](https://en.wikipedia.org/wiki/Ms.)
por lo que le asignaremos la edad media de la unión de estas categorías.
Este caso especial lo tratamos aparte, para no tener que hacer una excepción en el tratamiento general.
Corresponde al `PassengerId` 980,
como podemos ver a continuación:


```r
df[is.na(df['Age']) & is.na(df['Survived']) & df['Title'] == 'Ms',]
```

<div class="kable-table">

|    | PassengerId|Survived |Pclass |Name                    |Sex    | Age| SibSp| Parch|Ticket | Fare|Embarked |Title |
|:---|-----------:|:--------|:------|:-----------------------|:------|---:|-----:|-----:|:------|----:|:--------|:-----|
|980 |         980|NA       |3      |O'Donoghue, Ms. Bridget |female |  NA|     0|     0|364856 | 7.75|Q        |Ms    |

</div>

Le asignamos el valor acordado de manera manual:


```r
df[is.na(df['Age']) & is.na(df['Survived']) & df['Title'] == 'Ms', 'Age'] = 
  mean(df[!is.na(df['Age']) & is.na(df['Survived']) & (df[,'Title'] %in% c("Mrs", "Miss")), 'Age'])
```

Para inputar los valores definimos la siguiente función, 
que para cada registro mira a que subconjunto pertenece y le asigna la media de edad en este subconjunto del resto de registros de su mismo título:


```r
InputeAge = function(PassId){
  ToInpute = mean(
    df[!is.na(df['Age']) # Filtramos que tengan un valor definido de edad
       & is.na(df['Survived']) == is.na(df[df['PassengerId']==PassId, 'Survived']) # Seleccionamos subconjunto
       & df['Title'] ==  as.character(df[df['PassengerId']==PassId, 'Title']), # Filtramos que tengan el mismo título
       'Age']) # Calculamos la media
  return(ToInpute)
}
df[is.na(df['Age']), 'Age'] = sapply(FUN = InputeAge, X = df[is.na(df['Age']), 'PassengerId'])
```

### Variable `SibSp`

Finalmente tratamos los dos valores con `SibSp` sin informar debido a que los hemos considerado valores erróneos en el paso previo de detección de valores extremos.
Los registros a tratar son los siguientes:


```r
df[is.na(df['SibSp']), ]
```

<div class="kable-table">

|     | PassengerId|Survived |Pclass |Name                                   |Sex    | Age| SibSp| Parch|Ticket  |  Fare|Embarked |Title |
|:----|-----------:|:--------|:------|:--------------------------------------|:------|---:|-----:|-----:|:-------|-----:|:--------|:-----|
|69   |          69|1        |3      |Andersson, Miss. Erna Alexandra        |female |  17|    NA|     2|3101281 | 7.925|S        |Miss  |
|1106 |        1106|NA       |3      |Andersson, Miss. Ida Augusta Margareta |female |  38|    NA|     2|347091  | 7.775|S        |Miss  |

</div>

Como podemos ver cada uno pertenece a un subconjunto.
Para inputarles valores utilizaremos el algoritmo de los $k$ vecinos más cercanos (implementado en la función `kNN()` de la libraría `VIM`) y los atributos `Pclass`, `Age` y `Parch`.
La argumentación es que el número de hermanos presente depende de una combinación de la edad
y la clase del billete (clase social, en realidad).
Realizamos la inputación asegurándonos de que no hay filtrado de datos de test a entrenamiento:


```r
# Entrenamiento
train_inputado = VIM::kNN(df[!is.na(df['Survived']), ],
                          dist_var = c("Pclass", "Age", "Parch"),
                         variable = 'SibSp')
df[df['PassengerId']==69, 'SibSp'] = train_inputado[train_inputado['PassengerId']==69, 'SibSp']
# Evaluación
test_inputado = VIM::kNN(df[is.na(df['Survived']), ],
                          dist_var = c("Pclass", "Age", "Parch"),
                         variable = 'SibSp')
df[df['PassengerId']==1106, 'SibSp'] = test_inputado[test_inputado['PassengerId']==1106, 'SibSp']
```

Los valores inputados son los siguentes, en ambos casos cero como vemos:


```r
df[df[,'PassengerId'] %in% c(1106, 96), ]
```

<div class="kable-table">

|     | PassengerId|Survived |Pclass |Name                                   |Sex    |      Age| SibSp| Parch|Ticket |  Fare|Embarked |Title |
|:----|-----------:|:--------|:------|:--------------------------------------|:------|--------:|-----:|-----:|:------|-----:|:--------|:-----|
|96   |          96|0        |3      |Shorney, Mr. Charles Joseph            |male   | 32.36809|     0|     0|374910 | 8.050|S        |Mr    |
|1106 |        1106|NA       |3      |Andersson, Miss. Ida Augusta Margareta |female | 38.00000|     0|     2|347091 | 7.775|S        |Miss  |

</div>

## Resultado

Vemos como no queda ningún registro no identificado en los datos (más allá de los 418 registros de evaluación sin `Survived`):


```r
summary(df)
```

```
##   PassengerId     Survived   Pclass      Name               Sex     
##  Min.   :   1.0   0   :549   3:709   Length:1307        female:464  
##  1st Qu.: 328.5   1   :340   2:277   Class :character   male  :843  
##  Median : 655.0   NA's:418   1:321   Mode  :character               
##  Mean   : 655.3                                                     
##  3rd Qu.: 982.5                                                     
##  Max.   :1309.0                                                     
##                                                                     
##       Age            SibSp            Parch             Ticket    
##  Min.   : 0.17   Min.   :0.0000   Min.   :0.0000   CA. 2343:  11  
##  1st Qu.:21.66   1st Qu.:0.0000   1st Qu.:0.0000   1601    :   8  
##  Median :30.00   Median :0.0000   Median :0.0000   CA 2144 :   8  
##  Mean   :29.86   Mean   :0.4973   Mean   :0.3856   3101295 :   7  
##  3rd Qu.:36.00   3rd Qu.:1.0000   3rd Qu.:0.0000   347077  :   7  
##  Max.   :80.00   Max.   :8.0000   Max.   :9.0000   347082  :   7  
##                                                    (Other) :1259  
##       Fare         Embarked     Title    
##  Min.   :  0.000   C:270    Mr     :757  
##  1st Qu.:  7.896   Q:123    Miss   :259  
##  Median : 14.454   S:914    Mrs    :196  
##  Mean   : 33.205            Master : 61  
##  3rd Qu.: 31.275            Dr     :  8  
##  Max.   :512.329            Rev    :  8  
##                             (Other): 18
```
