---
jupyter:
  kernelspec:
    display_name: Python 3 (ipykernel)
    language: python
    name: python3
  language_info:
    codemirror_mode:
      name: ipython
      version: 3
    file_extension: .py
    mimetype: text/x-python
    name: python
    nbconvert_exporter: python
    pygments_lexer: ipython3
    version: 3.12.3
  nbformat: 4
  nbformat_minor: 5
---

::: {#97310630-2e4a-4a13-bf01-019718368504 .cell .markdown}
# Mapeamento de rotas com OSMnx
:::

::: {#c3ed3f6b-70bd-4e1f-bdb3-482ec6f4d0fe .cell .markdown}
Utilizamos a biblioteca do Python **OSMnx**, desenvolvida e mantida por
Geoff Boeing, professor de Planejamento Urbano e Análise Espacial da
USC - University of Southern California, para calcular e visualizar a
rota mais curta entre pontos de interesse, que fazem parte do circuito
turístico denominado [Museu de Percurso Raphael
Arcuri](https://www.instagram.com/museuraphaelarcuri?igsh=MWRjNWV1cnZnczE5aQ==),
de acordo com o projeto desenvolvido por [Letícia
Rabelo](https://www.instagram.com/leticiarabelo.arq?igsh=dndsYTdsemM4ZWdw).
:::

::: {#d9ab72d0-8bfd-4ba7-ad31-c1fa0571d0ac .cell .markdown}
### Importamos bibliotecas
:::

::: {#756622a0-a10e-4e80-aca8-942a0bff7ce1 .cell .code execution_count="31"}
``` python
import osmnx as ox
import pandas as pd
import numpy as np
#import geopandas as gpd 
import folium
from folium.plugins import Fullscreen
from shapely.geometry import Point
from sklearn.neighbors import NearestNeighbors
```
:::

::: {#7af52b21-f561-4ef7-bd6a-338b32cbd4f4 .cell .markdown}
### Identificamos a versão do pacote OSMnx

Verificamos a versão do **OSMnx** para garantir compatibilidade:
:::

::: {#6bfeb28b-9cf6-420d-b7c5-4eeb1fb6b3af .cell .code execution_count="32"}
``` python
print(f"Versão do OSMnx: {ox.__version__}")
```

::: {.output .stream .stdout}
    Versão do OSMnx: 2.0.3
:::
:::

::: {#1c2868e1-e3d9-4e34-8795-ce2c59b7efbf .cell .markdown}
### Definimos os pontos de interesse

Armazenamos os locais em um dicionário com as coordenadas:
:::

::: {#e71be10c-90cb-4023-b85f-78667cd9cbc5 .cell .code execution_count="33"}
``` python
coordenadas_referencia = {
    "Paço Municipal": (-21.761600, -43.349999),
    "ED. CIAMPI": (-21.761080, -43.349367),
    "Galeria Pio X": (-21.760661, -43.348583),
    "Cine Theatro Central": (-21.761135, -43.347926),
    "Palacete Pinho": (-21.760593, -43.346864),
    "Cia. Dias Cardoso": (-21.759881, -43.344195),
    "Hotel Príncipe": (-21.7599733, -43.3440394),
    "Associação Comercial": (-21.759881, -43.344195),
    "Cia. Pantaleone Arcuri": (-21.762536, -43.342788),
    "Vila Iracema": (-21.763336, -43.344481),
    "Palacete dos Fellet": (-21.763280,	-43.345717),
    "Residência Raphael Arcuri": (-21.763699, -43.342097),
    "Castelinho dos Bracher": (-21.763666, -43.341959),
    "Casa D'Itália": (-21.764455, -43.348467)
}
```
:::

::: {#6adbc2b3-113d-4f88-8b6f-2629f0791fd3 .cell .markdown}
### Criamos o data frame a partir do dicionário

Convertemos as coordenadas para um *data frame*:
:::

::: {#03d2ae7e-a045-493b-bd63-8f10faaf4121 .cell .code execution_count="34"}
``` python
df = pd.DataFrame.from_dict(coordenadas_referencia, 
                            orient='index', 
                            columns=['latitude', 'longitude'])

# Resetar o índice para ter uma coluna com os nomes dos locais
df = df.reset_index().rename(columns={'index': 'obra'})
```
:::

::: {#d8846ea1-8e54-46d4-a6b0-8d4167242497 .cell .markdown}
### Calculamos a rota mais curta

A classe *NearestNeighbors* do módulo *sklearn.neighbors*, junto com o
algoritmo *ball_tree*, fornece uma solução robusta para problemas de
busca por proximidade, como o do roteamento entre pontos geográficos.
:::

::: {#b0f941f3-8b0c-4f6a-b3f9-df2ea828a89f .cell .code execution_count="35"}
``` python
# Calcular a distância para cada par de pontos consecutivos
X = np.array(df[['latitude', 'longitude']])
obras = df['obra']
nbrs = NearestNeighbors(n_neighbors=len(X), algorithm='ball_tree').fit(X)
distances, indices = nbrs.kneighbors(X)

# Encontrar o roteiro mais curto
visited = np.zeros(len(X), dtype=bool)

end_point = 13  # Definindo o ponto final como 13 (Casa d'Itália)

visited[0] = True
tour = [0]
current = 0

# Modificado para parar quando chegar ao ponto 13
while current != end_point and len(tour) < len(X):
    unvisited_mask = np.logical_not(visited[indices[current]])
    if np.any(unvisited_mask):
        nearest = indices[current][unvisited_mask][0].item()
    else:
        # Se todos os vizinhos foram visitados, escolha o próximo não visitado
        unvisited = np.where(visited == False)[0]
        if len(unvisited) > 0:
            nearest = unvisited[0]
        else:
            break
    
    tour.append(nearest)
    visited[nearest] = True
    current = nearest

    # Se chegou ao ponto final, pare
    if current == end_point:
        break

# Nomes dos locais na ordem original
obras = df['obra'].tolist()  

# Ordenar o dicionário conforme a rota
coordenadas_ordenadas = {
    obras[i]: coordenadas_referencia[obras[i]] 
    for i in tour
}

# Resultado
print("Rota mais curta terminando no item 12:")
for i, point in enumerate(tour):
    print(f"{i}. {obras[point]} (Ponto {point})")
```

::: {.output .stream .stdout}
    Rota mais curta terminando no item 12:
    0. Paço Municipal (Ponto 0)
    1. ED. CIAMPI (Ponto 1)
    2. Galeria Pio X (Ponto 2)
    3. Cine Theatro Central (Ponto 3)
    4. Palacete Pinho (Ponto 4)
    5. Cia. Dias Cardoso (Ponto 5)
    6. Associação Comercial (Ponto 7)
    7. Hotel Príncipe (Ponto 6)
    8. Cia. Pantaleone Arcuri (Ponto 8)
    9. Residência Raphael Arcuri (Ponto 11)
    10. Castelinho dos Bracher (Ponto 12)
    11. Vila Iracema (Ponto 9)
    12. Palacete dos Fellet (Ponto 10)
    13. Casa D'Itália (Ponto 13)
:::
:::

::: {#943ed9e4-f123-4f15-be87-bb208618ae02 .cell .markdown}
### Criamos o dicionário da rota mais curta
:::

::: {#a2e6b10e-01ed-497c-b337-45d8a794b596 .cell .code execution_count="36"}
``` python
# Nomes dos locais na ordem original
obras = df['obra'].tolist()  

# Ordenar o dicionário conforme a rota
coordenadas_ordenadas = {
    obras[i]: coordenadas_referencia[obras[i]] 
    for i in tour
}
```
:::

::: {#df0c276e-63d1-4925-b510-5d7697a2f6b6 .cell .markdown}
### Definimos o percurso a pé

Usamos o pacote **OSMnx** para criar um grafo da rede viária para
pedestres. Para cada par de pontos consecutivos, calculamos o caminho
mais curto no grafo:
:::

::: {#3530f68a-a811-499a-8709-28559b921c86 .cell .code execution_count="37"}
``` python

# Transformar os valores do dicionário em uma lista de tuplas
itinerario = list(coordenadas_ordenadas.values())

# Criar grafo de caminhada ao redor do primeiro ponto
G = ox.graph_from_point(itinerario[0], dist=1500, network_type='walk')

# 2. Criar caminho completo conectando todos os pares consecutivos
full_path = []

for i in range(len(itinerario) - 1):
    orig_point = itinerario[i]
    dest_point = itinerario[i + 1]
    
    try:
        orig_node = ox.distance.nearest_nodes(G, orig_point[1], orig_point[0])  
        dest_node = ox.distance.nearest_nodes(G, dest_point[1], dest_point[0])
        
        segment = ox.shortest_path(G, orig_node, dest_node, weight='length')
        
        # Evitar duplicações de nós
        if full_path and full_path[-1] == segment[0]:
            full_path += segment[1:]
        else:
            full_path += segment
    except Exception as e:
        print(f"Erro ao processar trecho entre {orig_point} e {dest_point}: {e}")

# Obter coordenadas (lat, lon) dos nós do caminho
route_coords = [(G.nodes[n]['y'], G.nodes[n]['x']) for n in full_path]
```
:::

::: {#9b81101f-7042-4c90-bdf9-28125af62f8f .cell .markdown}
### Visualizamos o mapa

Criamos um mapa interativo com o pacote **Folium**:
:::

::: {#5b439466-e2cc-4d00-b6aa-63ae7aa589ba .cell .code execution_count="38" scrolled="true"}
``` python
# Criar mapa
mapa = folium.Map(location=itinerario[0], zoom_start=15)

# Adicionar pontos do itinerário como marcadores com ícones personalizados
nomes = list(coordenadas_ordenadas.keys())
for idx, (nome, coord) in enumerate(zip(nomes, coordenadas_ordenadas.values())):
    if idx == 0:
        icon_color = 'blue'
    elif idx == len(coordenadas_ordenadas) - 1:
        icon_color = 'red'
    else:
        icon_color = 'orange'
    
    folium.Marker(
        coord,
        tooltip=nome,
        icon=folium.Icon(icon="monument", color=icon_color, prefix="fa")
    ).add_to(mapa)

# Adicionar a linha da rota em vermelho
folium.PolyLine(route_coords, color='red', weight=4).add_to(mapa)

# Exibir o mapa
mapa
```

::: {.output .execute_result execution_count="38"}
```{=html}
<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><span style="color:#565656">Make this Notebook Trusted to load map: File -> Trust Notebook</span><iframe srcdoc="&lt;!DOCTYPE html&gt;
&lt;html&gt;
&lt;head&gt;
    
    &lt;meta http-equiv=&quot;content-type&quot; content=&quot;text/html; charset=UTF-8&quot; /&gt;
    &lt;script src=&quot;https://cdn.jsdelivr.net/npm/leaflet@1.9.3/dist/leaflet.js&quot;&gt;&lt;/script&gt;
    &lt;script src=&quot;https://code.jquery.com/jquery-3.7.1.min.js&quot;&gt;&lt;/script&gt;
    &lt;script src=&quot;https://cdn.jsdelivr.net/npm/bootstrap@5.2.2/dist/js/bootstrap.bundle.min.js&quot;&gt;&lt;/script&gt;
    &lt;script src=&quot;https://cdnjs.cloudflare.com/ajax/libs/Leaflet.awesome-markers/2.0.2/leaflet.awesome-markers.js&quot;&gt;&lt;/script&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdn.jsdelivr.net/npm/leaflet@1.9.3/dist/leaflet.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdn.jsdelivr.net/npm/bootstrap@5.2.2/dist/css/bootstrap.min.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://netdna.bootstrapcdn.com/bootstrap/3.0.0/css/bootstrap-glyphicons.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdn.jsdelivr.net/npm/@fortawesome/fontawesome-free@6.2.0/css/all.min.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdnjs.cloudflare.com/ajax/libs/Leaflet.awesome-markers/2.0.2/leaflet.awesome-markers.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdn.jsdelivr.net/gh/python-visualization/folium/folium/templates/leaflet.awesome.rotate.min.css&quot;/&gt;
    
            &lt;meta name=&quot;viewport&quot; content=&quot;width=device-width,
                initial-scale=1.0, maximum-scale=1.0, user-scalable=no&quot; /&gt;
            &lt;style&gt;
                #map_2d20ddedc54d0add2a96db7aa99a0a22 {
                    position: relative;
                    width: 100.0%;
                    height: 100.0%;
                    left: 0.0%;
                    top: 0.0%;
                }
                .leaflet-container { font-size: 1rem; }
            &lt;/style&gt;

            &lt;style&gt;html, body {
                width: 100%;
                height: 100%;
                margin: 0;
                padding: 0;
            }
            &lt;/style&gt;

            &lt;style&gt;#map {
                position:absolute;
                top:0;
                bottom:0;
                right:0;
                left:0;
                }
            &lt;/style&gt;

            &lt;script&gt;
                L_NO_TOUCH = false;
                L_DISABLE_3D = false;
            &lt;/script&gt;

        
&lt;/head&gt;
&lt;body&gt;
    
    
            &lt;div class=&quot;folium-map&quot; id=&quot;map_2d20ddedc54d0add2a96db7aa99a0a22&quot; &gt;&lt;/div&gt;
        
&lt;/body&gt;
&lt;script&gt;
    
    
            var map_2d20ddedc54d0add2a96db7aa99a0a22 = L.map(
                &quot;map_2d20ddedc54d0add2a96db7aa99a0a22&quot;,
                {
                    center: [-21.7616, -43.349999],
                    crs: L.CRS.EPSG3857,
                    ...{
  &quot;zoom&quot;: 15,
  &quot;zoomControl&quot;: true,
  &quot;preferCanvas&quot;: false,
}

                }
            );

            

        
    
            var tile_layer_f2abd9686df2ee462fa698062aa8b539 = L.tileLayer(
                &quot;https://tile.openstreetmap.org/{z}/{x}/{y}.png&quot;,
                {
  &quot;minZoom&quot;: 0,
  &quot;maxZoom&quot;: 19,
  &quot;maxNativeZoom&quot;: 19,
  &quot;noWrap&quot;: false,
  &quot;attribution&quot;: &quot;\u0026copy; \u003ca href=\&quot;https://www.openstreetmap.org/copyright\&quot;\u003eOpenStreetMap\u003c/a\u003e contributors&quot;,
  &quot;subdomains&quot;: &quot;abc&quot;,
  &quot;detectRetina&quot;: false,
  &quot;tms&quot;: false,
  &quot;opacity&quot;: 1,
}

            );
        
    
            tile_layer_f2abd9686df2ee462fa698062aa8b539.addTo(map_2d20ddedc54d0add2a96db7aa99a0a22);
        
    
            var marker_9ad2662eb03a64b397983d07f7b5dc99 = L.marker(
                [-21.7616, -43.349999],
                {
}
            ).addTo(map_2d20ddedc54d0add2a96db7aa99a0a22);
        
    
            var icon_025b1d63d9f03fa36b62799619b2a136 = L.AwesomeMarkers.icon(
                {
  &quot;markerColor&quot;: &quot;blue&quot;,
  &quot;iconColor&quot;: &quot;white&quot;,
  &quot;icon&quot;: &quot;monument&quot;,
  &quot;prefix&quot;: &quot;fa&quot;,
  &quot;extraClasses&quot;: &quot;fa-rotate-0&quot;,
}
            );
        
    
            marker_9ad2662eb03a64b397983d07f7b5dc99.bindTooltip(
                `&lt;div&gt;
                     Paço Municipal
                 &lt;/div&gt;`,
                {
  &quot;sticky&quot;: true,
}
            );
        
    
                marker_9ad2662eb03a64b397983d07f7b5dc99.setIcon(icon_025b1d63d9f03fa36b62799619b2a136);
            
    
            var marker_5dbdaa324c52b37836edab8d1b4abcfb = L.marker(
                [-21.76108, -43.349367],
                {
}
            ).addTo(map_2d20ddedc54d0add2a96db7aa99a0a22);
        
    
            var icon_6d3d9ced83de0e441d4e99837eebb5e5 = L.AwesomeMarkers.icon(
                {
  &quot;markerColor&quot;: &quot;orange&quot;,
  &quot;iconColor&quot;: &quot;white&quot;,
  &quot;icon&quot;: &quot;monument&quot;,
  &quot;prefix&quot;: &quot;fa&quot;,
  &quot;extraClasses&quot;: &quot;fa-rotate-0&quot;,
}
            );
        
    
            marker_5dbdaa324c52b37836edab8d1b4abcfb.bindTooltip(
                `&lt;div&gt;
                     ED. CIAMPI
                 &lt;/div&gt;`,
                {
  &quot;sticky&quot;: true,
}
            );
        
    
                marker_5dbdaa324c52b37836edab8d1b4abcfb.setIcon(icon_6d3d9ced83de0e441d4e99837eebb5e5);
            
    
            var marker_78d2560bf081e9ca782cf63a7f0629fa = L.marker(
                [-21.760661, -43.348583],
                {
}
            ).addTo(map_2d20ddedc54d0add2a96db7aa99a0a22);
        
    
            var icon_94b29da9c76cb13c0fcf5d9125dde3a0 = L.AwesomeMarkers.icon(
                {
  &quot;markerColor&quot;: &quot;orange&quot;,
  &quot;iconColor&quot;: &quot;white&quot;,
  &quot;icon&quot;: &quot;monument&quot;,
  &quot;prefix&quot;: &quot;fa&quot;,
  &quot;extraClasses&quot;: &quot;fa-rotate-0&quot;,
}
            );
        
    
            marker_78d2560bf081e9ca782cf63a7f0629fa.bindTooltip(
                `&lt;div&gt;
                     Galeria Pio X
                 &lt;/div&gt;`,
                {
  &quot;sticky&quot;: true,
}
            );
        
    
                marker_78d2560bf081e9ca782cf63a7f0629fa.setIcon(icon_94b29da9c76cb13c0fcf5d9125dde3a0);
            
    
            var marker_cd5e1e377ea54547dffb6ed547ea621f = L.marker(
                [-21.761135, -43.347926],
                {
}
            ).addTo(map_2d20ddedc54d0add2a96db7aa99a0a22);
        
    
            var icon_1dea8a09383d83896cc9e9b434412b6e = L.AwesomeMarkers.icon(
                {
  &quot;markerColor&quot;: &quot;orange&quot;,
  &quot;iconColor&quot;: &quot;white&quot;,
  &quot;icon&quot;: &quot;monument&quot;,
  &quot;prefix&quot;: &quot;fa&quot;,
  &quot;extraClasses&quot;: &quot;fa-rotate-0&quot;,
}
            );
        
    
            marker_cd5e1e377ea54547dffb6ed547ea621f.bindTooltip(
                `&lt;div&gt;
                     Cine Theatro Central
                 &lt;/div&gt;`,
                {
  &quot;sticky&quot;: true,
}
            );
        
    
                marker_cd5e1e377ea54547dffb6ed547ea621f.setIcon(icon_1dea8a09383d83896cc9e9b434412b6e);
            
    
            var marker_11f611f9056ec37183e1bf2ace06fdfb = L.marker(
                [-21.760593, -43.346864],
                {
}
            ).addTo(map_2d20ddedc54d0add2a96db7aa99a0a22);
        
    
            var icon_fa3250e9c3ac21852f49a581c6e62ac6 = L.AwesomeMarkers.icon(
                {
  &quot;markerColor&quot;: &quot;orange&quot;,
  &quot;iconColor&quot;: &quot;white&quot;,
  &quot;icon&quot;: &quot;monument&quot;,
  &quot;prefix&quot;: &quot;fa&quot;,
  &quot;extraClasses&quot;: &quot;fa-rotate-0&quot;,
}
            );
        
    
            marker_11f611f9056ec37183e1bf2ace06fdfb.bindTooltip(
                `&lt;div&gt;
                     Palacete Pinho
                 &lt;/div&gt;`,
                {
  &quot;sticky&quot;: true,
}
            );
        
    
                marker_11f611f9056ec37183e1bf2ace06fdfb.setIcon(icon_fa3250e9c3ac21852f49a581c6e62ac6);
            
    
            var marker_a66cd00eefa900a298b40deb33024866 = L.marker(
                [-21.759881, -43.344195],
                {
}
            ).addTo(map_2d20ddedc54d0add2a96db7aa99a0a22);
        
    
            var icon_cd11eb08ad332c10d0aaaefb5df5cc69 = L.AwesomeMarkers.icon(
                {
  &quot;markerColor&quot;: &quot;orange&quot;,
  &quot;iconColor&quot;: &quot;white&quot;,
  &quot;icon&quot;: &quot;monument&quot;,
  &quot;prefix&quot;: &quot;fa&quot;,
  &quot;extraClasses&quot;: &quot;fa-rotate-0&quot;,
}
            );
        
    
            marker_a66cd00eefa900a298b40deb33024866.bindTooltip(
                `&lt;div&gt;
                     Cia. Dias Cardoso
                 &lt;/div&gt;`,
                {
  &quot;sticky&quot;: true,
}
            );
        
    
                marker_a66cd00eefa900a298b40deb33024866.setIcon(icon_cd11eb08ad332c10d0aaaefb5df5cc69);
            
    
            var marker_b186a86f97fc4e70ae66005169d404f4 = L.marker(
                [-21.759881, -43.344195],
                {
}
            ).addTo(map_2d20ddedc54d0add2a96db7aa99a0a22);
        
    
            var icon_d8fe1f455a032c3ae34624f6014c32d0 = L.AwesomeMarkers.icon(
                {
  &quot;markerColor&quot;: &quot;orange&quot;,
  &quot;iconColor&quot;: &quot;white&quot;,
  &quot;icon&quot;: &quot;monument&quot;,
  &quot;prefix&quot;: &quot;fa&quot;,
  &quot;extraClasses&quot;: &quot;fa-rotate-0&quot;,
}
            );
        
    
            marker_b186a86f97fc4e70ae66005169d404f4.bindTooltip(
                `&lt;div&gt;
                     Associação Comercial
                 &lt;/div&gt;`,
                {
  &quot;sticky&quot;: true,
}
            );
        
    
                marker_b186a86f97fc4e70ae66005169d404f4.setIcon(icon_d8fe1f455a032c3ae34624f6014c32d0);
            
    
            var marker_c610526b4886b258248a2630bf43447c = L.marker(
                [-21.7599733, -43.3440394],
                {
}
            ).addTo(map_2d20ddedc54d0add2a96db7aa99a0a22);
        
    
            var icon_1427d7f0b35fcbbe52188f7447f29f2a = L.AwesomeMarkers.icon(
                {
  &quot;markerColor&quot;: &quot;orange&quot;,
  &quot;iconColor&quot;: &quot;white&quot;,
  &quot;icon&quot;: &quot;monument&quot;,
  &quot;prefix&quot;: &quot;fa&quot;,
  &quot;extraClasses&quot;: &quot;fa-rotate-0&quot;,
}
            );
        
    
            marker_c610526b4886b258248a2630bf43447c.bindTooltip(
                `&lt;div&gt;
                     Hotel Príncipe
                 &lt;/div&gt;`,
                {
  &quot;sticky&quot;: true,
}
            );
        
    
                marker_c610526b4886b258248a2630bf43447c.setIcon(icon_1427d7f0b35fcbbe52188f7447f29f2a);
            
    
            var marker_b3ae5c862dfa50683abac5c92184f15a = L.marker(
                [-21.762536, -43.342788],
                {
}
            ).addTo(map_2d20ddedc54d0add2a96db7aa99a0a22);
        
    
            var icon_ab63d1bb4bc6f42efd73c6ead993b284 = L.AwesomeMarkers.icon(
                {
  &quot;markerColor&quot;: &quot;orange&quot;,
  &quot;iconColor&quot;: &quot;white&quot;,
  &quot;icon&quot;: &quot;monument&quot;,
  &quot;prefix&quot;: &quot;fa&quot;,
  &quot;extraClasses&quot;: &quot;fa-rotate-0&quot;,
}
            );
        
    
            marker_b3ae5c862dfa50683abac5c92184f15a.bindTooltip(
                `&lt;div&gt;
                     Cia. Pantaleone Arcuri
                 &lt;/div&gt;`,
                {
  &quot;sticky&quot;: true,
}
            );
        
    
                marker_b3ae5c862dfa50683abac5c92184f15a.setIcon(icon_ab63d1bb4bc6f42efd73c6ead993b284);
            
    
            var marker_9fceef30264f9dca5d6c4f0cf6611e66 = L.marker(
                [-21.763699, -43.342097],
                {
}
            ).addTo(map_2d20ddedc54d0add2a96db7aa99a0a22);
        
    
            var icon_31d6a97ce9aec5bcfd32226eb6160b7a = L.AwesomeMarkers.icon(
                {
  &quot;markerColor&quot;: &quot;orange&quot;,
  &quot;iconColor&quot;: &quot;white&quot;,
  &quot;icon&quot;: &quot;monument&quot;,
  &quot;prefix&quot;: &quot;fa&quot;,
  &quot;extraClasses&quot;: &quot;fa-rotate-0&quot;,
}
            );
        
    
            marker_9fceef30264f9dca5d6c4f0cf6611e66.bindTooltip(
                `&lt;div&gt;
                     Residência Raphael Arcuri
                 &lt;/div&gt;`,
                {
  &quot;sticky&quot;: true,
}
            );
        
    
                marker_9fceef30264f9dca5d6c4f0cf6611e66.setIcon(icon_31d6a97ce9aec5bcfd32226eb6160b7a);
            
    
            var marker_c64df39adae1b4f7d6fbfd77546404b1 = L.marker(
                [-21.763666, -43.341959],
                {
}
            ).addTo(map_2d20ddedc54d0add2a96db7aa99a0a22);
        
    
            var icon_879c065ac99e9639b03bce63f300e8cd = L.AwesomeMarkers.icon(
                {
  &quot;markerColor&quot;: &quot;orange&quot;,
  &quot;iconColor&quot;: &quot;white&quot;,
  &quot;icon&quot;: &quot;monument&quot;,
  &quot;prefix&quot;: &quot;fa&quot;,
  &quot;extraClasses&quot;: &quot;fa-rotate-0&quot;,
}
            );
        
    
            marker_c64df39adae1b4f7d6fbfd77546404b1.bindTooltip(
                `&lt;div&gt;
                     Castelinho dos Bracher
                 &lt;/div&gt;`,
                {
  &quot;sticky&quot;: true,
}
            );
        
    
                marker_c64df39adae1b4f7d6fbfd77546404b1.setIcon(icon_879c065ac99e9639b03bce63f300e8cd);
            
    
            var marker_bfec4bf0d53275e8e82f74403e5fb3a5 = L.marker(
                [-21.763336, -43.344481],
                {
}
            ).addTo(map_2d20ddedc54d0add2a96db7aa99a0a22);
        
    
            var icon_b8469d12b9ebc925581cd360bd5f2e0c = L.AwesomeMarkers.icon(
                {
  &quot;markerColor&quot;: &quot;orange&quot;,
  &quot;iconColor&quot;: &quot;white&quot;,
  &quot;icon&quot;: &quot;monument&quot;,
  &quot;prefix&quot;: &quot;fa&quot;,
  &quot;extraClasses&quot;: &quot;fa-rotate-0&quot;,
}
            );
        
    
            marker_bfec4bf0d53275e8e82f74403e5fb3a5.bindTooltip(
                `&lt;div&gt;
                     Vila Iracema
                 &lt;/div&gt;`,
                {
  &quot;sticky&quot;: true,
}
            );
        
    
                marker_bfec4bf0d53275e8e82f74403e5fb3a5.setIcon(icon_b8469d12b9ebc925581cd360bd5f2e0c);
            
    
            var marker_e564f79339113a41e66776edde35a382 = L.marker(
                [-21.76328, -43.345717],
                {
}
            ).addTo(map_2d20ddedc54d0add2a96db7aa99a0a22);
        
    
            var icon_5419b82f3a8e5372ba83e4403ce6fe88 = L.AwesomeMarkers.icon(
                {
  &quot;markerColor&quot;: &quot;orange&quot;,
  &quot;iconColor&quot;: &quot;white&quot;,
  &quot;icon&quot;: &quot;monument&quot;,
  &quot;prefix&quot;: &quot;fa&quot;,
  &quot;extraClasses&quot;: &quot;fa-rotate-0&quot;,
}
            );
        
    
            marker_e564f79339113a41e66776edde35a382.bindTooltip(
                `&lt;div&gt;
                     Palacete dos Fellet
                 &lt;/div&gt;`,
                {
  &quot;sticky&quot;: true,
}
            );
        
    
                marker_e564f79339113a41e66776edde35a382.setIcon(icon_5419b82f3a8e5372ba83e4403ce6fe88);
            
    
            var marker_a187a528b09d99543422d21ffefbb10e = L.marker(
                [-21.764455, -43.348467],
                {
}
            ).addTo(map_2d20ddedc54d0add2a96db7aa99a0a22);
        
    
            var icon_8f3cff24416f6cc6d2bdcfe12684664e = L.AwesomeMarkers.icon(
                {
  &quot;markerColor&quot;: &quot;red&quot;,
  &quot;iconColor&quot;: &quot;white&quot;,
  &quot;icon&quot;: &quot;monument&quot;,
  &quot;prefix&quot;: &quot;fa&quot;,
  &quot;extraClasses&quot;: &quot;fa-rotate-0&quot;,
}
            );
        
    
            marker_a187a528b09d99543422d21ffefbb10e.bindTooltip(
                `&lt;div&gt;
                     Casa D&#x27;Itália
                 &lt;/div&gt;`,
                {
  &quot;sticky&quot;: true,
}
            );
        
    
                marker_a187a528b09d99543422d21ffefbb10e.setIcon(icon_8f3cff24416f6cc6d2bdcfe12684664e);
            
    
            var poly_line_20bfea4dfcfd2372cf7f7f012582c445 = L.polyline(
                [[-21.7614517, -43.3499423], [-21.7614302, -43.3498418], [-21.7613976, -43.3497007], [-21.7613666, -43.3495419], [-21.7611727, -43.3487218], [-21.7610793, -43.3483447], [-21.7610104, -43.3480625], [-21.7609892, -43.3479806], [-21.7608374, -43.3473592], [-21.7608237, -43.3473052], [-21.7607335, -43.3469199], [-21.7605254, -43.3460306], [-21.7600207, -43.3439359], [-21.7607605, -43.3437412], [-21.761358, -43.3435896], [-21.7616025, -43.3435138], [-21.7619813, -43.3434555], [-21.7621654, -43.343307], [-21.762344, -43.3434227], [-21.7623855, -43.3435746], [-21.7626141, -43.3432674], [-21.7627769, -43.3428571], [-21.7624821, -43.3413788], [-21.7635133, -43.3403744], [-21.7637102, -43.3411809], [-21.7643484, -43.343896], [-21.7630959, -43.3442741], [-21.7632752, -43.344993], [-21.7634341, -43.3456573], [-21.7635722, -43.3461502], [-21.7636826, -43.3462358], [-21.7636443, -43.3463515], [-21.7636817, -43.3468602], [-21.7644325, -43.3470148], [-21.7645972, -43.347173], [-21.7649258, -43.3486333], [-21.7648029, -43.3486693]],
                {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;red&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: false, &quot;fillColor&quot;: &quot;red&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;noClip&quot;: false, &quot;opacity&quot;: 1.0, &quot;smoothFactor&quot;: 1.0, &quot;stroke&quot;: true, &quot;weight&quot;: 4}
            ).addTo(map_2d20ddedc54d0add2a96db7aa99a0a22);
        
&lt;/script&gt;
&lt;/html&gt;" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>
```
:::
:::

::: {#19508773-4334-4b54-882a-6c397c4c6a6a .cell .code}
``` python
```
:::

::: {#121ed144-4438-4669-a60a-10bf9d1d96e4 .cell .markdown}
**Considerações finais**

Personalizamos os ícones dos marcadores com cores distintas para
identificar cada etapa do percurso.

O código pode ser adaptado para tarefas distintas, como roteiros
turísticos otimizados, planejamento de entregas, análise de
acessibilidade urbana e simulações de evacuação, em casos de desastres
naturais.
:::

::: {#2503d8fd-1d49-4842-a2c1-fa099bd3d693 .cell .code}
``` python
```
:::

::: {#35f05af1-f664-4d5b-8cca-e13bdd05385b .cell .markdown}
**Referências**

Boeing, G. (2025). Modeling and Analyzing Urban Networks and Amenities
with OSMnx. Geographical Analysis, published online ahead of print.
<doi:10.1111/gean.70009>

SCIKIT-LEARN. User Guide: Nearest Neighbors. 2025. Disponível em:
<https://scikit-learn.org/stable/modules/neighbors.html>. Acesso em: 18
JUN 2025.
:::

::: {#d0657ce8-2d7c-47e8-b490-eacee6e6cb10 .cell .code}
``` python
```
:::
