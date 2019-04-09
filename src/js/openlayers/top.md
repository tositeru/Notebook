# OpenLayers

## 便利サイト

- [geojson.io](http://geojson.io) 位置情報の編集に
- [epsg.io](https://epsg.io/) 好きな空間参照系の変換のときにproj4.jsと組み合わせて使う

## マーカーの見た目を変える方法

```js
import Map from 'ol/Map.js';
import View from 'ol/View.js';
import Tile from 'ol/source/Tile.js'
import OSM from 'ol/source/OSM.js'
import proj from 'ol/proj.js'

let feature = new Feature({
  geometry: new Point()
})
let map = new ol.Map({
  target: 'map',
  layers: [
	new Tile({
		source: new OSM()
	})
  ],
  view: new View({
	  center: proj.fromLonLat([139.745433,35.658581]),
	  zoom: 16
  })
})

```

## GeoJSONからデータを読み込んで指定した空間参照系に変換するコード

```js
import format from 'ol/format'

const features = (new format.GeoJSON()).readFeatures(loadGeoJSONFileForTest())
for (const f of features) {
  const geom = f.getGeometry()
  if (!geom) continue
  geom.transform(/* from */ 'EPSG:4326', /* to */'EPSG:3857')
}
```

ちなみにある座標値を変換したいときは`ol.proj.transform(coordinate, from, to)`を使うといい

```js
import proj from 'ol/proj'
const coordinate = [123.123, 123.123]
proj.transform(coordinate, /* from */ 'EPSG:4326', /* to */'EPSG:3857')
```
## ドラッグアンドドロップで座標データを読み込ませたいときのコード

```js
import DragAndDrop from 'ol/interaction/DragAndDrop'
import source from 'ol/source'
import layer from 'ol/layer'
import format from 'ol/format'

function initDragAndDrop(map) {
  const source = new source.Vector()
  const layer = new layer.Vector({
    source: source
  })
  map.addLayer(layer)
  map.addInteraction(new DragAndDrop({
    source: source,
    formatConstructors: [format.GeoJSON]
  }))
}
```
