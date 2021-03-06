```{r, eval=FALSE, include=TRUE}
"Protocolo:

1. Daniel Felipe Villa Rengifo

2. Lenguaje: R

3. Tema: Manejo de archivos xml ,json Y leer tablas incrustadas en archivos HTML.

4. Fuentes:
   https://rsanchezs.gitbooks.io/ciencia-de-datos-con-r/content/importar_datos/jerarquico/archivos_formato_jerarquico.html

  https://www.datanalytics.com/libro_r/json-y-xml.html"
```


# Manejo de tablas incrustadas en archivos HTML:

_Con frecuencia las fuentes de datos están incrustadas en una web y necesitamos adquirir dicha información para trabajar con ella en un dataFrame._

Los datos se leen con un paquete llamado `XML` para utilizar la función `readHTMLTable` asi tenemos una lista de todas las listas dadas en en HTML

```{r}
install.packages("XML")
library(XML)
```


Hay dos formas de leer tabas incrustadas en el HTML:

1. Desde una URL
2. Descargando la pagina web

Las dos fomras las veremos acá

```{r}
# UNO:
"agregamos la url"

link <- "https://en.wikipedia.org/wiki/World_population"
```

La petición anterior ha devuelto un error: `# Warning message: XML content does not seem to be XML:``

Esto error ocurre porque la función `readHTMLTable` no es capaz de leer HTTPS. Para eso usaremos la librería `RCurl`, que nos permite obtener datos con R y guardarlos en un objeto en memoria, aunque el contenido esté en una URL segura.

```{r}

#obtenemos el HTML y lo guardamos en una variable de R ya que no recibe solamente el link:
# Para eso necesitamos otro paquete:
install.packages("RCurl")
library(RCurl)

htmldata <- getURL(link)

#La ponemos dentro dentro de la funcion y listo:
tabla <- readHTMLTable(htmldata)
```


El ejercicio consta de leer una pagina de wikipedia [WorldPopulation](https://en.wikipedia.org/wiki/World_population) y sacar algunas tablas para hacer estadisticas de algunos datos:

```{r}
# Leamos la tablas incrustadas en la pagina descargada:
# imprimimos el directorio:
dir()

#observamos que necesitamos el archivo N°7:
"Asi dejamos el nombre del archivo en una variable caracter"
POPurl <- "WorldPopulation-wiki.html"

#ahora lo convertimos en tabla:
"Entra como una lista de todas las tablas del html dado"
tabla <- readHTMLTable(POPurl)

print(tabla)

crecimientoanual <- tabla[[8]]
print(crecimientoanual)
# Otra forma de leer los archivos es dando valor a which:
crecimientoanual <- readHTMLTable(POPurl, which = 8)
print(crecimientoanual)

"De las dos formas dara el mismo resultado"
```

```{r}
# Ahora veremos si los datos por año estan muy alejados del promedio total de años en la tabla:

# Pero antes transformaremos varios datos y les daremos nombres a sus columnas
# (dejando solamente la que necesitamos):

# Eliminando las columnas inecesarias para el ejercicio:
borrar <- c("V5", "V6", "V7")

crecimientoanual <- crecimientoanual[,!(names(crecimientoanual) %in% borrar)]

# Cambiamos los nombres de las columnas:

names(crecimientoanual) <- c("year","Poblacion", "PorcetajeCrecimiento", "NumeroCrecimiento")

#Ahora con la funcion gsub removeremos el porcentaje:

crecimientoanual$PorcetajeCrecimiento <-  gsub("%", "",
                                               crecimientoanual$PorcetajeCrecimiento)

" Removemos las comas"
crecimientoanual$Poblacion <- gsub(",", "", crecimientoanual$Poblacion)
crecimientoanual$NumeroCrecimiento <- gsub(",", "",
                                           crecimientoanual$NumeroCrecimiento)

# pasamos de factor a numeric las columnas necesarias:
crecimientoanual$Poblacion <- as.numeric(crecimientoanual$Poblacion)

crecimientoanual$PorcetajeCrecimiento <- as.numeric(crecimientoanual$PorcetajeCrecimiento)

crecimientoanual$NumeroCrecimiento <- as.numeric(crecimientoanual$NumeroCrecimiento)

# Ahora el boxplot del promdio de crecimiento por porcentaje anual desde 1951 a 2020:

png(filename = "BoxPLotXCreciemtoPoblacionalXPorcentaje.png")

boxplot(crecimientoanual$PorcetajeCrecimiento, horizontal = T, main = "Promedio anual de crecimento poblacional por porcentaje", xlab = "Porcentaje %", col = "green")

dev.off()
```
Vemos que no tenemos datos atipicos, eso nos muestra que  el creciento anual no se aleja el promedio por mucho, pero como podemos ver su percntil 50 esta descentralizado teniendo una tendencia a crecer la poblacion anual un poco más cada año.



# Lectura de archivos XML

## ¿Qué es XML?

XML, `eXtensible Markup Language`, traducido como “Lenguaje de Marcado Extensible” o “Lenguaje de Marcas Extensible” es un meta-lenguaje que permite definir lenguajes de marcas desarrollado por el World Wide Web Consortium (W3C) utilizado para almacenar datos en forma legible.

## Librería XML en R

En mi caso, vamos a utilizar la librería XML que está disponible en el CRAN (The Comprehensive R Archive Network) que es el repositorio oficial de librerías de R.

```{r}
# Una vez instala la libreria, cargamos el nombre del archivo en caracter:
XML_url <- "cd_catalog.xml"

# Leemos el archivo como XML (descargado)
xmldoc <- xmlParse(XML_url)

# Ahora tomamos el grueso del XML
rootnode <- xmlRoot(xmldoc)

#visualicemos:
print(rootnode)

"Ahora para extraer solamente los datos utlizamos las funciones apply"
# Asi extraemos info como dataframe
cds_data <- xmlSApply(rootnode, function(x) xmlSApply(x, xmlValue) )
print(cds_data)

# Como podemos ver necesitamos trasponer y eliminar el names() de las filas sin trasponer:
cds.catalog <- data.frame(t(cds_data), row.names = NULL)
print(cds.catalog)

"Una vez organizado el archivo esta listo para trabajar"

"El data frame se basa en una venta de discos con:

Titulo del disco

Artista

Pais

Disquera

Precio del disco

Año de publicacion"
```

# Lectura de archivos json:

Desgraciadamente no siempre tendremos un fichero con el que trabajar, si no que tendremos que cargar la información directamente de internet. Hay multitud de formatos, pero sin duda el más extendido es JSON.

Con esto veremos un ejemplo de leer un archivo json en tiempo real, el cual se trata de divisas actuales


+ Para leer archivos json se necesita la libreria `jsonlite`

```{r}
library(jsonlite)
# Damos la url como caracter
url = "http://www.floatrates.com/daily/usd.json"

# Leemos el archivo con la función fromJSON:
"Nos arrojara una lista de listas de las divisas actuales"
j = fromJSON(url)

#veamos las primeras 50 listas
print(j[1:50])

# Podemos convetir en un dataframe para leer la informacion de una manera más entendible una de las lista, cuales quiera:

"En este caso la 3 lista de la lista j"

###Todos estos datos etan respecto al dolar (Moneda Mundial)###

tabla3json <- data.table::as.data.table(j[[3]])
```