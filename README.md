# repositoriodeprueba
Este es un repositorio de prueba para aprender a realizar cambios y comentarlos.
#________________________________________________________
#REVISAR LA SEGUNDA PARTE DEL SCRIP

#Control+f para remplazar
rm(list=ls())
gc()#Liberar memoria de manera efectiva

#Librerias 1:
library(readxl)
library(writexl)
library(tidyverse)
library(lubridate)
library(ggplot2)
library(dplyr)
library(readr)#Para usar csv

#Librerias 2:
library(httr)#Para conectar con servidor APIs
library(jsonlite)#Para transformar a objetos de r y leer json
library(rvest) #Para scrapear
library(leaflet) #Para mapear


#_____________________PARTE 1__________________________

#Donde queremos guardarlo
setwd()

#MAPA EN TIPO REAL DE LA POSICIÓN DE LOS AVIONES
#pAGINA https://openskynetwork.github.io/opensky-api/rest.html
       
url = "https://opensky-network.org/api/states/all"
datos <- GET(url)
datos #Aquí tenemos la respuesta de la URL (datos de españa)
      #El estatus, es el todo, lo normal es que sea 200, 
      #Si es un estatus 400 es cosa nuestra
      #sí el status es 500 el problema es del servidor

#La respuesta viene en formato json, para convertirlo a objeto de r
datos <- fromJSON(content(datos, type = "text"))
datos <- datos[["states"]] #Esto ya lo convierte en un data frame

#Lo que sigue es scrapiar la tabla pues al convertir en data frame no tenemos títulos
url = "https://openskynetwork.github.io/opensky-api/rest.html"
nombres <- read_html(url) %>% html_nodes("#all-state-vectors") %>% html_nodes("#response") %>% html_nodes('.docutils') %>% html_table()
colnames(datos) <- nombres[[2]]$Property #No me dejo hacer agregar los encabezados, veo que me aparecen 18, uno más
datos <- as.data.frame(datos, stringsAsFactors = FALSE)


#Este paso no viene en el tutorial, pero lo hice para agregar los encabezados
Encabezados <- c("icao24", "callsign", "origin_country", "time_position", "last_contact", "longitude",
                 "latitude", "baro_altitude", "on_ground", "velocity", "true_track", "vertical_rate", 
                 "sensors", "geo_altitude", "squawk", "spi", "position_source", "v18")
colnames(datos) <- Encabezados

str(datos)
str(datos$longitude)
#Convertir longitud y latitud a numerico
datos$longitude <- as.numeric(datos$longitude)
datos$latitude <- as.numeric(datos$longitude)

#Crear un mapa
leaflet() %>%
  addTiles() %>%
  addCircles(lng = datos$longitude, lat = datos$latitude, color = "#ff5733", opacity = 0.3)

#____________________________________________________________________________PARTE 2__________________________

#MAPA EN TIPO REAL DE LA POSICIÓN DE LOS AVIONES
#pAGINA https://openskynetwork.github.io/opensky-api/rest.html

url = "https://opensky-network.org/api/states/all"
datos <- GET(url)
datos #Aquí tenemos la respuesta de la URL (datos de españa)
#El estatus, es el todo, lo normal es que sea 200, 
#Si es un estatus 400 es cosa nuestra
#sí el status es 500 el problema es del servidor

#La respuesta viene en formato json, para convertirlo a objeto de r
datos <- fromJSON(content(datos, type = "text"))
datos <- datos[["states"]] #Esto ya lo convierte en un data frame

#Agregamos el paso de guardarlo
getwd()
setwd("C:/R/Prueba_sky")
write.csv(x = datos, file = "datos_vuelos.csv", row.names = FALSE)
datos2 <- read.csv("datos_vuelos.csv")

#Este paso no viene en el tutorial, pero lo hice para agregar los encabezados
Encabezados <- c("icao24", "callsign", "origin_country", "time_position", "last_contact", "longitude",
                 "latitude", "baro_altitude", "on_ground", "velocity", "true_track", "vertical_rate", 
                 "sensors", "geo_altitude", "squawk", "spi", "position_source", "v18")
#Agregar encabezados
colnames(datos2) <- Encabezados
str(datos2)

#Crear un mapa
leaflet() %>%
  addTiles() %>%
  addCircles(lng = datos2$longitude, lat = datos2$latitude, color = "#ff5733", opacity = 0.3)


#_______________Automatización de Script____________

install.packages("taskschuduleR")
library(taskscheduleR)
#abrir la ventana o bien buscarla en Addins
taskscheduleR:::taskschedulerAddin()

fichero <- "C:\\R\\Prueba_sky\\sky.R"


taskscheduler_create(taskname = "sky2",
                     rscript = fichero,
                     schedule = "MINUTE",
                     starttime = format(Sys.time(), "%12:%06:%10"),
                     startdate = format(Sys.time(), "%10/%05/%2022"))

taskscheduler_delete(sky)


