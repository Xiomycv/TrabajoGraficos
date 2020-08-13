# Análisis de Datos 

Análisis de datos referentes a web scrapping hecho en la página de [Yapo](https://www.yapo.cl/region_metropolitana?ca=15_s&o=) en donde se obtuvieron datos sobre categorías, comunas, precios, tipo de negocio, año del usuario Yapo y url de donde se sacó la información particular. 

## Variables que llaman la atención

Se separaron 2 variables, estas variables son: *comunas* y *categorías* 

### Comunas 

En la Región Metropolitana tenemos 32 comunas, las cuales se ven representadas en el gráfico. 

![Comunas](https://raw.githubusercontent.com/Xiomycv/TrabajoGraficos/master/Gráfico_comunas.png)

(Fuente: elaboración propia en RStudio)

En este gráfico podemos observar que, la comuna en donde más se vende es la comuna de Santiago, siguiéndola la comuna de la Cisterna y Maipú.

Para entender de mejor manera el comportamiento de las comunas ante los numeros de venta, podemos proponer una hipótesis en la que estipulamos que a mayor cantidad de habitantes por comuna, tenemos mayor cantidad de ventas por la misma. 

![Habitantes por comuna](https://github.com/Xiomycv/TrabajoGraficos/blob/master/Tabladecomunasyhabitantes.png)

(Fuente: Tabla elaborada a partir de datos Censo 2017)

En esta tabla, podemos observar algunas de las comunas de la Región Metropolitana y las cantidades de habitantes de cada una, elaborada a partir del Censo 2017.

De esta tabla podemos tomar las siguientes conclusiones: 

#### Comunas con mayor cantidad de habitantes

| Comuna | Cantidad de habitantes |
| ------------- | ------------- |
| Puente Alto  | 568.106  |
| Maipú  | 521.627  |
| Santiago  | 404.495 |


Entonces, teniendo los datos obtenidos de la página [Censo 2017](http://www.censo2017.cl/descargue-aqui-resultados-de-comunas/), podemos comparar las comunas con mayor cantidad de habitantes con las comunas con mayores ventas en la página de Yapo. 

+ La comuna de **Puente Alto** es la que mayor cantidad de habitantes tiene, y en ámbitos de ventas por Yapo ocupa el **7º lugar**.
+ La comuna de **Maipú** le sigue, y en ámbitos de ventas por Yapo ocupa el **3º lugar**.
+ La comuna de **Santiago** le sigue, y en ámbitos de ventas por Yapo ocupa el **1º lugar**. 

Gracias a este análisis podemos llegar a la conclusión que la hipótesis planteada es correcta, ya que podemos ver que las comunas con más habitantes suelen ocupar las primeras posiciones en cuanto a ventas en la página de Yapo. (de un total de 32 posiciones)

### Categorías 

En la página de Yapo podemos encontrar un total de 31 categorías de venta en la Región Metropolitana. 

![Categorias](https://raw.githubusercontent.com/Xiomycv/TrabajoGraficos/master/Gráfico_categorias.png)

(Fuente: Elaboración propia en RStudio)

En este gráfico podemos observar que la categoría en la que más productos se venden es la de Accesorios y piezas para vehículos.

Entonces, podemos proponer la hipótesis de que los accesorios de vehículos son más baratos en la página de Yapo y son más caros nuevos en otros retail. 
Para esto, vamos a sacar información nuevamente de la página de Yapo para comprar precios. 

| Producto | Precio |
| ------------- | ------------- |
| Parachoque Suzuki sx4 scross (Yapo) | [$55.000](https://www.yapo.cl/region_metropolitana/accesorios_vehiculos/parachoque_suzuki_sx4_scross_envio_regiones_73763604.htm?ca=15_s&oa=73763604&xsp=31)  |
| Parachoque Suzuki sx4 scross (Repuesto original) | [$75.000](https://repuestosoriginales.cl/parachoque/5307-parachoque-delantero-suzuki-sx4-s-cross-08-11.html)  |

Con este producto, podemos ver que la diferencia es de $20.000 por un producto usado y uno nuevo. Entonces, podríamos confirmar la hipótesis de que existen productos para vehículos en la página de Yapo que son más baratos y es por esto que más gente prefiere este sitio. 


# Valores N/A

Junto a los valores encontrados en la tabla representada por el data frame obtenido en el web scrapping de la página de Yapo, encontramos valores "vacíos" o "N/A".
Estos se representan de esta manera a partir de este código:

**Precios**
```r
sacandoPrecio <- function(htmlSeg){
  nodoBread <- html_nodes(htmlSeg, ".offer")
  if(length(nodoBread)>0){
    precio <- html_text(nodoBread)
    precio <- gsub("\\t","",precio)
    precio <- gsub("\\n","",precio)
    precio <- gsub("\\$","",precio)
    precio <- gsub("[.]","",precio)
    precio <- as.numeric(precio)
  }else{
    precio = NA
```
Al analizar este script, podemos observar que los precios sacados de la página deben pasar por una limpieza, en la cual se le eliminan de su naturaleza carácteres como signos de peso y puntos ("$", "."). 
Luego de que los precios hayan pasado por esta limpieza, retornan como números fáciles de recopilar, pero si uno de esos precios que pasó por este proceso no consta con ninguno de esos carácteres o el precio no está enlistado en la publicación, este retornará como **"N/A"**

**Tipo de negocio**
```r
obtenerTipoNegocio <- function(htmlSeg){
  nodoBread <- html_nodes(htmlSeg, ".details")
  nodoBread <- html_nodes(nodoBread, "table")
  tabla <- html_table(nodoBread)[[1]]
  subsettn <- unlist(subset(tabla, X1 == 'Tipo')[2])
  if(length(subsettn)>0){
    return(subsettn)
  }else{
    return(NA)
  }
}
```
Al analizar este script, podemos observar que se crea una variable llamada "subsettn", en donde guardamos la tabla de tipos de negocios no enlistada (unlist). Luego, si el largo de los elementos del subsettn es mayor a 0 (osea contiene carácteres), este retornará con los resultados, pero si los elementos del subsettn son iguales a 0 (osea que está vacío y no tiene carácteres), este retornará como **"NA"**.

# Ejercicio Optativo 

```r
  fullDatos <- data.frame() 
#se crea un dataframe vacío con el nombre de fullDatos
  for(numeroPagina in 1:3){ 
    readHtml <- read_html(paste("https://www.yapo.cl/region_metropolitana?ca=15_s&o=",numeroPagina,sep = ""))
    print(paste("Descargando pagina nro:",numeroPagina))
    nodeTabla <- html_nodes(readHtml, ".listing_thumbs")
    nodeTabla <- html_nodes(nodeTabla, ".title")
    linksProductos <- html_attr(nodeTabla,"href")
#se lee el html de la pagina de yapo seleccionada y se extrae un array de elementos desde el html leido.
#todo se guarda en la variable linksProductos
    for (urlYapo in linksProductos) {
      htmlSeg <- read_html(urlYapo)
      
      print(paste("Descargando URL ==> ",urlYapo))
      
      textoTipoAviso <- obtenerCategoria(htmlSeg)
      precio <- sacandoPrecio(htmlSeg)
      comuna <- obtenerComuna(htmlSeg)
      tipoNegocio <- obtenerTipoNegocio(htmlSeg)
      anioUsuarioyapo <- obtenerAnioUsuarioYapo(htmlSeg)
      publicacionesactivasusuarioyapo <- obtenerPublicacionesActivasUsuarioYapo(htmlSeg)
      publicacionestotalesusuarioyapo <- obtenerPublicacionesTotalesUsuarioYapo(htmlSeg)
      
      fullDatos <- rbind(fullDatos,data.frame(comuna = comuna, categoria = textoTipoAviso, precio = precio,
                                              tiponegocio = tipoNegocio, aniousuarioyapo = anioUsuarioyapo,
                                              urlyapo = urlYapo ))
    }
  }
#urlYapo recorre la variable linksProductos, se crea variable htmlSeg que contiene la lectura de urlYapo
#se crean variables "textoTipoAviso","precio","comuna","tipoNegocio","anioUsuarioyapo",
#"publicacionesactivasusuarioyapo" y "publicacionestotalesusuarioyapo".
#luego, se utiliza rbind para pegar los argumentos por filas en un data.frame, y asi obetenemos la tabla.
```      
                                             
                                       



