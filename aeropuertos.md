# Visualización de despegues de vuelos en Argentina

La [base de datos de movimientos aeroportuarios en Argentina](https://datos.gob.ar/dataset/transporte-aterrizajes-despegues-registrados-por-empresa-argentina-navegacion-aerea-eana) contiene información con despegues y aterrizajes en los aeropuertos nacionales.

A partir de esto, pensé en aplicar herramientas de ETL en Pandas para armar algunas visualizaiones usando Geopandas, Seaborn y Plotl.

## Datasets

* Los datasets se encuentran desmembrados por años, en formato csv, desde el 2014 en adelante. 


* A partir de esa información, generé un dataset desde el 2014 hasta el 2020 (inclusive) del que parto para hacer este trabajo. 


* A esto le sumé información geográfica sobre los aeropuertos para poder hacer gráficos en geopandas. Esta información la tome de [The Global Airport Database](https://www.partow.net/miscellaneous/airportdatabase/)



```python
import pandas as pd
import numpy as np
import geopandas as gpd
import seaborn as sns
import matplotlib.pyplot as plt
import plotly.graph_objects as go
import plotly.express as px
```

## ETL: Ordenando y Limpiando los Datasets

### Location
Ubicación de los aeropuertos. Elimino información no útil, elimino Nan, cambio nombres de columnas


```python
location = pd.read_csv("GlobalAirportDatabase.txt", sep=":")
```


```python
location.reset_index(inplace=True)
location.drop(["3","5","6","7","8","9","10","11","12","13"], axis = 1,inplace = True)
location.columns = ["ICAO","IATA","Aeropuerto","Pais","Lat","Long"]
location.dropna(inplace=True)
```


```python
location.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>ICAO</th>
      <th>IATA</th>
      <th>Aeropuerto</th>
      <th>Pais</th>
      <th>Lat</th>
      <th>Long</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>AYGA</td>
      <td>GKA</td>
      <td>GOROKA</td>
      <td>PAPUA NEW GUINEA</td>
      <td>-6.082</td>
      <td>145.392</td>
    </tr>
    <tr>
      <th>2</th>
      <td>AYMD</td>
      <td>MAG</td>
      <td>MADANG</td>
      <td>PAPUA NEW GUINEA</td>
      <td>-5.207</td>
      <td>145.789</td>
    </tr>
    <tr>
      <th>3</th>
      <td>AYMH</td>
      <td>HGU</td>
      <td>MOUNT HAGEN</td>
      <td>PAPUA NEW GUINEA</td>
      <td>-5.826</td>
      <td>144.296</td>
    </tr>
    <tr>
      <th>4</th>
      <td>AYNZ</td>
      <td>LAE</td>
      <td>NADZAB</td>
      <td>PAPUA NEW GUINEA</td>
      <td>-6.570</td>
      <td>146.726</td>
    </tr>
    <tr>
      <th>5</th>
      <td>AYPY</td>
      <td>POM</td>
      <td>PORT MORESBY JACKSONS INTERNATIONAL</td>
      <td>PAPUA NEW GUINEA</td>
      <td>-9.443</td>
      <td>147.220</td>
    </tr>
  </tbody>
</table>
</div>



### Datos
Información de datos.gob.ar. Concateno los datasets y genero uno nuevo


```python
datos2014 = pd.read_csv("aterrizajes-y-despegues-registrados-por-eana-2014 .csv",sep=";")
datos2015 = pd.read_csv("aterrizajes-y-despegues-registrados-por-eana-2015.csv",sep=";")
datos2016 = pd.read_csv("aterrizajes-y-despegues-registrados-por-eana-2016.csv",sep=";")
datos2017 = pd.read_csv("aterrizajes-y-despegues-registrados-por-eana-2017.csv",sep=";")
datos2018 = pd.read_csv("aterrizajes-y-despegues-registrados-por-eana-2018.csv",sep=";")
datos2019 = pd.read_csv("aterrizajes-y-despegues-registrados-por-eana-2019.csv",sep=";")
datos2020 = pd.read_csv("aterrizajes-y-despegues-registrados-por-eana-2020.csv",sep=";")

datos = pd.concat([datos2014,datos2015,datos2016,datos2017,datos2018,datos2019,datos2020],axis = 0)
```


```python
datos.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Fecha</th>
      <th>Hora</th>
      <th>Clase de Vuelo</th>
      <th>Clasificaci�n Vuelo</th>
      <th>Tipo de Movimiento</th>
      <th>Origen OACI</th>
      <th>Destino OACI</th>
      <th>Aerolinea Nombre</th>
      <th>Aeronave</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>01/01/2014</td>
      <td>1.0</td>
      <td>No Regular</td>
      <td>Cabotaje</td>
      <td>Aterrizaje</td>
      <td>SAME</td>
      <td>SACO</td>
      <td>SOL L�neas A�reas</td>
      <td>SAAB SF34</td>
    </tr>
    <tr>
      <th>1</th>
      <td>01/01/2014</td>
      <td>1.0</td>
      <td>Regular</td>
      <td>Internacional</td>
      <td>Despegue</td>
      <td>SABE</td>
      <td>SBGR</td>
      <td>Gol Transportes A�reo</td>
      <td>BOEING B-737</td>
    </tr>
    <tr>
      <th>2</th>
      <td>01/01/2014</td>
      <td>1.0</td>
      <td>Regular</td>
      <td>Cabotaje</td>
      <td>Aterrizaje</td>
      <td>SASJ</td>
      <td>SABE</td>
      <td>Austral L�neas A�reas</td>
      <td>EMBRAER E-190</td>
    </tr>
    <tr>
      <th>3</th>
      <td>01/01/2014</td>
      <td>1.0</td>
      <td>Regular</td>
      <td>Cabotaje</td>
      <td>Aterrizaje</td>
      <td>SABE</td>
      <td>SACO</td>
      <td>Austral L�neas A�reas</td>
      <td>EMBRAER E-190</td>
    </tr>
    <tr>
      <th>4</th>
      <td>01/01/2014</td>
      <td>1.0</td>
      <td>Regular</td>
      <td>Internacional</td>
      <td>Despegue</td>
      <td>SAEZ</td>
      <td>KIAH</td>
      <td>United Airlines</td>
      <td>BOEING B-767</td>
    </tr>
  </tbody>
</table>
</div>



## Merging
En cada registro de datos, agrego información geográfica de ambos aeropuertos (despegue y aterrizaje). Para eso duplico la tabla location, generando una para despegues y otra para aterrizajes


```python
origen = location
origen.columns=["Origen OACI", "Origen IATA", "Airport Origen", "Country Origen", "Lat Origen", "Long Origen"]
```


```python
origen.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Origen OACI</th>
      <th>Origen IATA</th>
      <th>Airport Origen</th>
      <th>Country Origen</th>
      <th>Lat Origen</th>
      <th>Long Origen</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>AYGA</td>
      <td>GKA</td>
      <td>GOROKA</td>
      <td>PAPUA NEW GUINEA</td>
      <td>-6.082</td>
      <td>145.392</td>
    </tr>
    <tr>
      <th>2</th>
      <td>AYMD</td>
      <td>MAG</td>
      <td>MADANG</td>
      <td>PAPUA NEW GUINEA</td>
      <td>-5.207</td>
      <td>145.789</td>
    </tr>
    <tr>
      <th>3</th>
      <td>AYMH</td>
      <td>HGU</td>
      <td>MOUNT HAGEN</td>
      <td>PAPUA NEW GUINEA</td>
      <td>-5.826</td>
      <td>144.296</td>
    </tr>
    <tr>
      <th>4</th>
      <td>AYNZ</td>
      <td>LAE</td>
      <td>NADZAB</td>
      <td>PAPUA NEW GUINEA</td>
      <td>-6.570</td>
      <td>146.726</td>
    </tr>
    <tr>
      <th>5</th>
      <td>AYPY</td>
      <td>POM</td>
      <td>PORT MORESBY JACKSONS INTERNATIONAL</td>
      <td>PAPUA NEW GUINEA</td>
      <td>-9.443</td>
      <td>147.220</td>
    </tr>
  </tbody>
</table>
</div>




```python
union = pd.merge(datos, origen,left_on="Origen OACI", right_on = "Origen OACI", how = "outer")
```


```python
destino = location
destino.columns = ["Destino OACI", "Destino IATA", "Airport Destino", "Country Destino", "Lat Destino", "Long Destino"]
```


```python
union2 = pd.merge(union, destino,left_on="Destino OACI", right_on = "Destino OACI")
union2.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Fecha</th>
      <th>Hora</th>
      <th>Clase de Vuelo</th>
      <th>Clasificaci�n Vuelo</th>
      <th>Tipo de Movimiento</th>
      <th>Origen OACI</th>
      <th>Destino OACI</th>
      <th>Aerolinea Nombre</th>
      <th>Aeronave</th>
      <th>Origen IATA</th>
      <th>Airport Origen</th>
      <th>Country Origen</th>
      <th>Lat Origen</th>
      <th>Long Origen</th>
      <th>Destino IATA</th>
      <th>Airport Destino</th>
      <th>Country Destino</th>
      <th>Lat Destino</th>
      <th>Long Destino</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>10/01/2019</td>
      <td>7.0</td>
      <td>Regular</td>
      <td>Cabotaje</td>
      <td>Aterrizaje</td>
      <td>SAME</td>
      <td>SABE</td>
      <td>LATAM Argentina</td>
      <td>AIRBUS A-320</td>
      <td>MDZ</td>
      <td>EL PLUMERILLO</td>
      <td>ARGENTINA</td>
      <td>-32.832</td>
      <td>-68.793</td>
      <td>AEP</td>
      <td>AEROPARQUE JORGE NEWBERY</td>
      <td>ARGENTINA</td>
      <td>-34.559</td>
      <td>-58.416</td>
    </tr>
    <tr>
      <th>1</th>
      <td>10/01/2019</td>
      <td>7.0</td>
      <td>Regular</td>
      <td>Cabotaje</td>
      <td>Despegue</td>
      <td>SAME</td>
      <td>SABE</td>
      <td>Norwegian Air Argenti</td>
      <td>BOEING B-737</td>
      <td>MDZ</td>
      <td>EL PLUMERILLO</td>
      <td>ARGENTINA</td>
      <td>-32.832</td>
      <td>-68.793</td>
      <td>AEP</td>
      <td>AEROPARQUE JORGE NEWBERY</td>
      <td>ARGENTINA</td>
      <td>-34.559</td>
      <td>-58.416</td>
    </tr>
    <tr>
      <th>2</th>
      <td>10/01/2019</td>
      <td>15.0</td>
      <td>Regular</td>
      <td>Cabotaje</td>
      <td>Aterrizaje</td>
      <td>SAME</td>
      <td>SABE</td>
      <td>Norwegian Air Argenti</td>
      <td>BOEING B-737</td>
      <td>MDZ</td>
      <td>EL PLUMERILLO</td>
      <td>ARGENTINA</td>
      <td>-32.832</td>
      <td>-68.793</td>
      <td>AEP</td>
      <td>AEROPARQUE JORGE NEWBERY</td>
      <td>ARGENTINA</td>
      <td>-34.559</td>
      <td>-58.416</td>
    </tr>
    <tr>
      <th>3</th>
      <td>10/01/2019</td>
      <td>9.0</td>
      <td>Regular</td>
      <td>Cabotaje</td>
      <td>Despegue</td>
      <td>SAME</td>
      <td>SABE</td>
      <td>Aerol�neas Argentinas</td>
      <td>BOEING B-737</td>
      <td>MDZ</td>
      <td>EL PLUMERILLO</td>
      <td>ARGENTINA</td>
      <td>-32.832</td>
      <td>-68.793</td>
      <td>AEP</td>
      <td>AEROPARQUE JORGE NEWBERY</td>
      <td>ARGENTINA</td>
      <td>-34.559</td>
      <td>-58.416</td>
    </tr>
    <tr>
      <th>4</th>
      <td>10/01/2019</td>
      <td>19.0</td>
      <td>Regular</td>
      <td>Cabotaje</td>
      <td>Despegue</td>
      <td>SAME</td>
      <td>SABE</td>
      <td>Norwegian Air Argenti</td>
      <td>BOEING B-737</td>
      <td>MDZ</td>
      <td>EL PLUMERILLO</td>
      <td>ARGENTINA</td>
      <td>-32.832</td>
      <td>-68.793</td>
      <td>AEP</td>
      <td>AEROPARQUE JORGE NEWBERY</td>
      <td>ARGENTINA</td>
      <td>-34.559</td>
      <td>-58.416</td>
    </tr>
  </tbody>
</table>
</div>




```python
union2.drop(["Hora", "Clase de Vuelo", "Clasificaci�n Vuelo", "Origen OACI", "Destino OACI"], axis = 1, inplace = True)
```

* Dado que el DataSet original listaba los *movimientos* aeroportuarios, cada ruta está duplicada, indicando cuando un avión despega de un aeropuerto nacional y cuando aterriza. Por eso, conservaremos únicamente los aterrizajes.


```python
datosOK = union2[union2["Tipo de Movimiento"]=="Aterrizaje"]
print("Dataset Size: ", datosOK.shape)
```

    Dataset Size:  (178107, 14)


* Los Datos ya están organizados, y pasaremos a hacer algunas visualizaciones


```python
datosOK.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Fecha</th>
      <th>Tipo de Movimiento</th>
      <th>Aerolinea Nombre</th>
      <th>Aeronave</th>
      <th>Origen IATA</th>
      <th>Airport Origen</th>
      <th>Country Origen</th>
      <th>Lat Origen</th>
      <th>Long Origen</th>
      <th>Destino IATA</th>
      <th>Airport Destino</th>
      <th>Country Destino</th>
      <th>Lat Destino</th>
      <th>Long Destino</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>10/01/2019</td>
      <td>Aterrizaje</td>
      <td>LATAM Argentina</td>
      <td>AIRBUS A-320</td>
      <td>MDZ</td>
      <td>EL PLUMERILLO</td>
      <td>ARGENTINA</td>
      <td>-32.832</td>
      <td>-68.793</td>
      <td>AEP</td>
      <td>AEROPARQUE JORGE NEWBERY</td>
      <td>ARGENTINA</td>
      <td>-34.559</td>
      <td>-58.416</td>
    </tr>
    <tr>
      <th>2</th>
      <td>10/01/2019</td>
      <td>Aterrizaje</td>
      <td>Norwegian Air Argenti</td>
      <td>BOEING B-737</td>
      <td>MDZ</td>
      <td>EL PLUMERILLO</td>
      <td>ARGENTINA</td>
      <td>-32.832</td>
      <td>-68.793</td>
      <td>AEP</td>
      <td>AEROPARQUE JORGE NEWBERY</td>
      <td>ARGENTINA</td>
      <td>-34.559</td>
      <td>-58.416</td>
    </tr>
    <tr>
      <th>6</th>
      <td>10/01/2019</td>
      <td>Aterrizaje</td>
      <td>Norwegian Air Argenti</td>
      <td>BOEING B-737</td>
      <td>MDZ</td>
      <td>EL PLUMERILLO</td>
      <td>ARGENTINA</td>
      <td>-32.832</td>
      <td>-68.793</td>
      <td>AEP</td>
      <td>AEROPARQUE JORGE NEWBERY</td>
      <td>ARGENTINA</td>
      <td>-34.559</td>
      <td>-58.416</td>
    </tr>
    <tr>
      <th>9</th>
      <td>10/01/2019</td>
      <td>Aterrizaje</td>
      <td>Norwegian Air Argenti</td>
      <td>BOEING B-737</td>
      <td>MDZ</td>
      <td>EL PLUMERILLO</td>
      <td>ARGENTINA</td>
      <td>-32.832</td>
      <td>-68.793</td>
      <td>AEP</td>
      <td>AEROPARQUE JORGE NEWBERY</td>
      <td>ARGENTINA</td>
      <td>-34.559</td>
      <td>-58.416</td>
    </tr>
    <tr>
      <th>11</th>
      <td>10/01/2019</td>
      <td>Aterrizaje</td>
      <td>Aerol�neas Argentinas</td>
      <td>BOEING B-737</td>
      <td>MDZ</td>
      <td>EL PLUMERILLO</td>
      <td>ARGENTINA</td>
      <td>-32.832</td>
      <td>-68.793</td>
      <td>AEP</td>
      <td>AEROPARQUE JORGE NEWBERY</td>
      <td>ARGENTINA</td>
      <td>-34.559</td>
      <td>-58.416</td>
    </tr>
  </tbody>
</table>
</div>



### Visualización de rutas aréas

Primero, usaremos la librería _geopandas_ para graficar las rutas aéreas en el DataSet. Separaremos esto en dos gráficos, vuelos locales y nacionales. 


```python
locales = datosOK[datosOK["Country Origen"]=="ARGENTINA"]
localesGroup = locales.groupby(["Lat Origen","Long Origen","Lat Destino","Long Destino"]).count()[["Fecha"]].reset_index()
localesGroup.columns = ["Lat Origen","Long Origen","Lat Destino","Long Destino","Vuelos"]

internacionales = datosOK[datosOK["Country Origen"]!="ARGENTINA"]
internacionalesGroup=internacionales.groupby(["Lat Origen","Long Origen","Lat Destino","Long Destino"]).count()[["Fecha"]].reset_index()
internacionalesGroup.columns = ["Lat Origen","Long Origen","Lat Destino","Long Destino","Vuelos"]
```

## Rutas Internacionales


```python
fig = go.Figure()

origen_a_dest = zip(internacionalesGroup["Lat Origen"], internacionalesGroup["Long Origen"],
                     internacionalesGroup["Lat Destino"], internacionalesGroup["Long Destino"],
                     internacionalesGroup["Vuelos"])


## 
for olat,olon, dlat, dlon, vuelos in origen_a_dest:
    fig.add_trace(go.Scattergeo(
                        lat = [olat,dlat],
                        lon = [olon, dlon],
                        mode = 'lines',
                        line = dict(width = 0.05*np.log(vuelos), color="red")
                        ))
    
##
fig.update_layout(
                  height=600, width=800, margin={"t":50,"b":0,"l":0, "r":0, "pad":0},
                  showlegend=False,
                  title={
                        'text': "Vuelos con origen en aeropuertos argentinos y destinos internacionales",
                        'y':0.97,
                        'x':0.5,
                        'xanchor': 'center',
                        'yanchor': 'top'},
                  geo = dict(projection_type = 'orthographic', showland = True),
                )
fig.show()
```



## Rutas Locales


```python
fig = go.Figure()

origen_a_dest = zip(localesGroup["Lat Origen"], localesGroup["Long Origen"],
                     localesGroup["Lat Destino"], localesGroup["Long Destino"],
                     localesGroup["Vuelos"])


## 
for olat,olon, dlat, dlon, vuelos in origen_a_dest:
    fig.add_trace(go.Scattergeo(
                        lat = [olat,dlat],
                        lon = [olon, dlon],
                        mode = 'lines',
                        line = dict(width = 0.05*np.log(vuelos), color="red")
                        ))
    
##
fig.update_layout(
                  height=600, width=800, margin={"t":0,"b":0,"l":0, "r":0, "pad":0},
                  showlegend=False,
                  title={
                        'text': "Vuelos con origen en aeropuertos argentinos y destinos nacionales",
                        'y':0.97,
                        'x':0.5,
                        'xanchor': 'center',
                        'yanchor': 'top'},
                  geo = dict(projection_type = 'mercator',scope = 'south america'),
                )
fig.show()
```



## Vuelos locales: Destinos más frecuentes
Dado que la mayoría de las rutas locales son entre Aeroparque y otros aeropuertos, podemos visualizar en un gráfico tipo _treemap_ esta división


```python
destinosAEP=locales[locales["Origen IATA"]=="AEP"].groupby("Destino IATA").count()[['Fecha']]
destinosAEP["ORIGEN"] = "AEP"
destinosAEP.reset_index(inplace=True)
destinosAEP.columns= ["Destino", "Cantidad", "Origen"]

fig = px.treemap(destinosAEP, path=['Origen', 'Destino'], values='Cantidad',
                  color_continuous_midpoint=np.average(destinosAEP['Cantidad'], weights=destinosAEP['Cantidad']))

fig.update_layout(
                  height=600, width=800, margin={"t":40,"b":0,"l":0, "r":0, "pad":0},

                  title={
                        'text': "Destinos nacionales con origen en Aeroparque",
                        'y':0.97,
                        'x':0.5,
                        'xanchor': 'center',
                        'yanchor': 'top'},
                )
fig.show()
```


## Aeronaves más frecuentes
Por último, podemos ver qué aeronave se utiliza más en todos los vuelos desde argentina


```python
aeronaves=datosOK.groupby("Aeronave").count()[["Fecha"]]
aeronaves.reset_index(inplace=True)
aeronaves.columns = ["Aeronave", "Cantidad de Vuelos"]
aeronaves.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Aeronave</th>
      <th>Cantidad de Vuelos</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>AERO AC50</td>
      <td>13</td>
    </tr>
    <tr>
      <th>1</th>
      <td>AERO AC68</td>
      <td>3</td>
    </tr>
    <tr>
      <th>2</th>
      <td>AERO BOERO AB11</td>
      <td>89</td>
    </tr>
    <tr>
      <th>3</th>
      <td>AERO BOERO AB18</td>
      <td>34</td>
    </tr>
    <tr>
      <th>4</th>
      <td>AERO COMMANDER</td>
      <td>17</td>
    </tr>
  </tbody>
</table>
</div>



Graficaremos únicamente aquellas aviones que aparecen más de 500 veces, dado que hay muchos aviones muy pequeños con muy pocos viajes en todos los años estudiados


```python
plt.figure(figsize=(15, 15))
sns.barplot(y="Aeronave",x="Cantidad de Vuelos", 
                  data = aeronaves[aeronaves["Cantidad de Vuelos"]>500],palette="Blues_d")

```



    
![png](asset/img/aeropuertos-aviones.png)

    

