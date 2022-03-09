<script>
import minneapolis from './data/minneapolis_neighborhoods.json';
import stpaul from './data/stpaul_neighborhoods.json';
import { onMount } from 'svelte';
import * as jq from 'jquery';
import * as mapboxgl from 'mapbox-gl';
import * as MapboxGeocoder from '@mapbox/mapbox-gl-geocoder';
import '@mapbox/mapbox-gl-geocoder/dist/mapbox-gl-geocoder.css';
import * as turf from '@turf/turf';

let center = [-93.183629, 44.970970];
let mcenter = [-93.183629, 44.970970];
let metrocenter = [-93.183629, 44.970970];
let zoom = 11;
let mzoom = 9.7;
let metrozoom = 10.7;

mapboxgl.accessToken = 'pk.eyJ1Ijoic3RhcnRyaWJ1bmUiLCJhIjoiY2sxYjRnNjdqMGtjOTNjcGY1cHJmZDBoMiJ9.St9lE8qlWR5jIjkPYd3Wqw';

function makeMap() {
/********** MAKE MAP **********/

const map = new mapboxgl.Map({
  container: 'map',
  style: 'mapbox://styles/startribune/ck1b7427307bv1dsaq4f8aa5h',
  center: metrocenter,
  zoom: metrozoom,
  minZoom: 9.7,
  maxZoom: 15,
  maxBounds: [-107.2,40.88,-78.92,51.62],
  scrollZoom: false
});

//geocoder
const geocoder = new MapboxGeocoder({
      accessToken: mapboxgl.accessToken,     
      marker: { color: '#5bbf48' },
      placeholder: 'Search for an address or location',
      countries: 'us',
      bbox: [-97.24,43.5,-89.48,49.38],
      zoom: 13,
      mapboxgl: mapboxgl
    });

document.getElementById('geocoder').appendChild(geocoder.onAdd(map));


/********** GEOCODER DISTRICT LOCATION RETURN **********/
  var geoPoint;
  var neighborhood;

  function searchWithin(layer, point) {
    var polygon;
    for (var i=0; i < layer.features.length; i++) {
      polygon = turf.multiPolygon(layer.features[i].geometry.coordinates);
      var test = turf.booleanPointInPolygon(point, polygon);
      if (test == true) { return layer.features[i].properties.Name; }
    }
    return 'X';
  }

  geocoder.on('result', (event) => {
    if (event.result.center[0] != null) {
      geoPoint = [event.result.center[0], event.result.center[1]];
      neighborhood = searchWithin(minneapolis, geoPoint);
      if (neighborhood == "X") { 
        neighborhood = searchWithin(stpaul, geoPoint); 
      }
      if (neighborhood != "X") { 
        jq("#resultC").html('That location is centered within the <strong>' + neighborhood + '</strong> neighborhood');
        jq("#resultC").css('visibility','visible');
      } else {
        jq("#resultC").html("Location not found within Minneapolis or St. Paul");
        jq("#resultC").css('visibility','visible');
      }
    } 
  });
  geocoder.on('clear', (event) => {
    jq("#resultC").css('visibility','hidden');
  });

/********** SPECIAL RESET BUTTON **********/
class MetroReset {
  onAdd(map){
    this.map = map;
    this.container = document.createElement('div');
    this.container.className = 'mapboxgl-ctrl my-custom-control mapboxgl-ctrl-group metroreset';

    const button = this._createButton('mapboxgl-ctrl-icon StateFace monitor_button')
    this.container.appendChild(button);
    return this.container;
  }
  onRemove(){
    this.container.parentNode.removeChild(this.container);
    this.map = undefined;
  }
  _createButton(className) {
    const el = window.document.createElement('button')
    el.className = className;
    el.innerHTML = '&#8962;';
    el.addEventListener('click',(e)=>{
     // e.style.display = 'none'
     // e.stopPropagation()
    },false )
    return el;
  }
}
const toggleControlM = new MetroReset();

var scale = new mapboxgl.ScaleControl({
  maxWidth: 80,
  unit: 'imperial'
  });
  map.addControl(scale)

// Setup basic map controls
if (/Android|webOS|iPhone|iPad|iPod|BlackBerry|IEMobile|Opera Mini/i.test(navigator.userAgent)) {
  map.dragPan.disable();
  map.keyboard.disable();
  map.dragRotate.disable();
  map.touchZoomRotate.disableRotation();
  map.scrollZoom.disable();
  map.addControl(toggleControlM,'bottom-right');
} else {

  map.getCanvas().style.cursor = 'pointer';
  map.addControl(new mapboxgl.NavigationControl({ showCompass: false }),'top-left');
  map.addControl(toggleControlM,'top-left');

jq('#map .metroreset').on('click', function(){
  map.jumpTo({
    center: metrocenter,
    zoom: metrozoom,
  });
});

}

/********** MAP BEHAVIORS **********/

map.on('load', function() {

  map.setPaintProperty(
    'water',
    'fill-color','#ededed' 
  );


      map.addSource('minneapolis', {
        type: 'geojson',
        data: minneapolis
      });

      map.addLayer({
            'id': 'minneapolis',
            'interactive': true,
            'source': 'minneapolis',
            'layout': {},
            'type': 'fill',
            'paint': {
              'fill-color': '#000000',
              'fill-opacity': 0,
              'fill-outline-color': '#ffffff'
            }
        }, "settlement-subdivision-label");

      map.addLayer({
            'id': 'minneapolis-l',
            'interactive': true,
            'source': 'minneapolis',
            'layout': {},
            'type': 'line',
            'paint': {
              'line-width': 0.5,
              'line-color': '#9e003f'
            }
        }, "settlement-subdivision-label");

      map.addSource('stpaul', {
        type: 'geojson',
        data: stpaul
      });

      map.addLayer({
            'id': 'stpaul',
            'interactive': true,
            'source': 'stpaul',
            'layout': {},
            'type': 'fill',
            'paint': {
              'fill-color': '#000000',
              'fill-opacity': 0,
              'fill-outline-color': '#ffffff'
            }
        }, "settlement-subdivision-label");

      map.addLayer({
            'id': 'stpaul-l',
            'interactive': true,
            'source': 'stpaul',
            'layout': {},
            'type': 'line',
            'paint': {
              'line-width': 0.5,
              'line-color': '#9e003f'
            }
        }, "settlement-subdivision-label");

    function tooltips(layer) {
          const popup = new mapboxgl.Popup({
          closeButton: false,
          closeOnClick: false
          });
          
          map.on('mousemove', layer, function(e) {
                map.getCanvas().style.cursor = 'pointer';

                var feature = e.features[0];

                popup.setLngLat(e.lngLat)
                    .setText(feature.properties.Name)
                    .addTo(map);
            });

            map.on('mouseleave', layer, function() {
                map.getCanvas().style.cursor = '';
                popup.remove();
            });
    }
    tooltips('minneapolis');
    tooltips('stpaul');
});

jq(document).ready(function() {
  if ((jq("#map").width() < 520)) {
      map.flyTo({
          center: mcenter,
          zoom: mzoom
      });
  }
  jq(window).resize(function() {
      if ((jq("#map").width() < 520)){
          map.flyTo({
              center: mcenter,
              zoom: mzoom
          });
      } else {
          map.flyTo({
              center: center,
              zoom: metrozoom
          });
      }
  });
});
}

    onMount(() => {
        makeMap();
    });
</script>

<div id="geocoder" class="geocoder"></div>

<div class="results" id="resultC">&nbsp;</div>

<div class="map" id="map"></div>