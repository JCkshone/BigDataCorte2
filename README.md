
<!-- TABLE OF CONTENTS -->
<details open="open">
  <summary>Table de Contenido</summary>
  <ol>
    <li>
      <a href="#Jupyter Lab">Creacion de proyecto en Jupyter Lab</a>
      <ul>
        <li><a href="#jupyter-lab-web">Jupyter Lab en la web</a></li>
      </ul>
    </li>
    <li>
      <a href="#web-scrapping">Web Scrapping</a>
      <ul>
        <li><a href="#dependency-installation">Instalacion de dependencias</a></li>
        <li><a href="#web-request">Request a la pagina web</a></li>
        <li><a href="#data-build">Construccion de la informacion con BeautifulSoup</a></li>
      </ul>
    </li>
  </ol>
</details>

<!-- ABOUT THE PROJECT -->
## Introduccion
En este Readme se van a demostrar y a describir cada uno de los pasos a seguir para construir un json usable con la informacion de cada uno de los presidentes de Colombia desde los incios es decir 1810.

## Creacion de proyecto en Jupyter Lab
Nos dirigimos a la [Web](https://jupyter.org) y seleccionamos intentar en la web, esto nos iniciara una maquina virtual enlazada a un entorno web donde podremos trabajar sin la necesidad de instalar y configurar Jupyter en un entorno local

![image](https://user-images.githubusercontent.com/19766554/138575387-0135332e-b96a-4129-a6b6-1e6fa42b9bd0.png)

## Web Scrapping
### Instalacion de dependencias
Para instalar las dependencias es muy simple, solo nos dirigimos a unas de las seccion para compilar codigo en Jupyterlab y se ejecuta la instruccion 
  ```sh
  pip install nombre-de-la-dependencia
  ```
![image](https://user-images.githubusercontent.com/19766554/138575493-041b109a-5bfa-4b70-b7cb-b9924f91847c.png)

Para este escenario nos va a pedir reiniciar el kernel, en Jupyter existe una opcion llamada kernel, solo debemos dar clic en esa opcion y luego reiniciar.

![image](https://user-images.githubusercontent.com/19766554/138575483-a3547b41-4466-4087-b2b1-cb4f16e4ae81.png)

### Request a la pagina web
Luego de realizar los pasos mencionados anteriormente inciamos la construccion,
#### Importamos las librerias
  ```sh
  import requests
  from bs4 import BeautifulSoup
  import json
  ```
#### Hacemos una solicitud a la pagina de wikipedia donde se encuenta toda la informacion de los presidentes de colombia e inicializamos BeautifulSoup
  ```sh
  URL = "https://es.wikipedia.org/wiki/Anexo:Presidentes_de_Colombia"
  page = requests.get(URL)
  soup = BeautifulSoup(page.content, "html.parser")
  ```
#### Indicamos que elementos HTML bamos a buscar para posteriormente usar
  ```sh
  tables = soup.find_all("table", class_="wikitable")
  ```
Si observamos en la pagina web mensionada anteriormente, al utilizar el inspector de elementos de cualquier buscador, podemos observar una clase llamada wiki table, la cual contiene toda la informacion que necesitamos.

![image](https://user-images.githubusercontent.com/19766554/138575631-9d40d280-36ad-4925-9aee-7ee2cdd83152.png)

#### Inicializamos las variables que almacenaran la informacion procesada
  ```sh
  presidentData = []
  titleData = []
  final_data = []
  final_titleData = []
  ```
### Construccion de la informacion con BeautifulSoup
#### Vamos a realizar la busqueda de todas las etiquetas que contienen informacion relevante almacenando los resultados en las variables declaradas anteriormente

```sh
  for items in tables:
      for presidentInfo in items.find_all(['tr']):
          titleData.append(presidentInfo.find_all(['th']))
          presidentData.append(presidentInfo)
```

 #### Como queremos construir un JSON para que la informacion sea un poco mas manipulable vamos a construir las llaves de la estructura con los titulos de las tablas
 
```sh
  for (key, item) in enumerate(titleData):
      final_titleData.append({
          "parent": [i.text.rstrip("\n") for i in item],
          "position": key
      })
```

 #### Luego de conbstruir las keys vamos a construir y setear la informacion correspondiente, algo a tener en cuenta es que se creo una llave fija para almacenar la imagen del presidente
 
 ```sh
  for (key, presidentItem) in enumerate(presidentData):
      model = {}
      if len(final_titleData[key-1]["parent"]):
          title = final_titleData[key-1]["parent"]
      for (k, data) in enumerate(presidentItem.find_all(['td'])):
          tag = data.find("a")
          if tag and tag.img:
              tag = tag.find('img')
              model["president_img"] = "https:"+tag["src"]
          if len(data.text) > 1:
              model[title[k-1]] = data.text

      if len(model):
          final_data.append({
              "president_info": model
          })
```

 #### Finalmente le hacemos un formateo al json para mostrarlo con la libresia json
 
 ```sh
  json_formatted_str = json.dumps(final_data, indent=2, ensure_ascii=False)
  print(json_formatted_str)
```

![image](https://user-images.githubusercontent.com/19766554/138575781-effc8f03-df2d-45da-94b3-2e04f4c8de10.png)

## Referencias 
- [Archivo.ipynb](https://github.com/JCkshone/BigDataCorte2/blob/main/Untitled.ipynb) con la ejecucion
