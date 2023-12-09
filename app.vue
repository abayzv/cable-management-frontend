<template>
  <div class="grid grid-cols-2">
    <GoogleMap
      api-key="AIzaSyBHiGcY7tohl9P47ZOYtY5gH4IKdwjndU0"
      style="width: 100%; height: 700px"
      :center="center"
      :zoom="4"
      ref="map"
    />
    <div class="m-4">
      <pre>
        {{ pathOptions }}
      </pre>
    </div>
  </div>
</template>
<script>
import { defineComponent } from "vue";
import { GoogleMap, Polyline, InfoWindow, Marker } from "vue3-google-map";

export default defineComponent({
  components: {
    GoogleMap,
    Polyline,
    InfoWindow,
    Marker
  },
  setup() {
    const center = ref({ lat: -0.789275, lng: 113.921327 });
    const newPath = ref({
      isStartPointDefined: false,
      path : []
    })

    const isCreatable = ref(true);

    const openedMarkerID = ref(1);

    const infoRef = ref([])
    const map = ref(null)
    
    const pathOptions = ref([
      {
        path: [
          { lat: -6.1751, lng: 106.865 }, // Jakarta
          { lat: -7.7956, lng: 110.3695 }, // Yogyakarta
        ],
        geodesic: true,
        strokeColor: "#FF0000",
        strokeOpacity: 1.0,
        strokeWeight: 2,
        editable: true,
      },
      {
        path: [
          { lat: -8.3405, lng: 115.092 }, // Bali
          { lat: -0.5897, lng: 117.0918 }, // Kalimantan
        ],
        geodesic: true,
        strokeColor: "#0000FF",
        strokeOpacity: 1.0,
        strokeWeight: 2,
        editable: true,
      },
    ]);

    const polylineRef = ref()

    const randomColor = () => {
      return "#" + Math.floor(Math.random() * 16777215).toString(16);
    };

    const createMapPoint = (event) => {
      
      if (!isCreatable.value) return; 
      console.log(event.latLng.lat(), event.latLng.lng());

      if (!newPath.value.isStartPointDefined) {
        newPath.value.path.push({
          lat: event.latLng.lat(),
          lng: event.latLng.lng()
        });
        newPath.value.isStartPointDefined = true;

        const newPathOptions = {
          path: newPath.value.path,
          geodesic: true,
          strokeColor: randomColor(),
          strokeOpacity: 1.0,
          strokeWeight: 2,
          editable: true,
        }
        pathOptions.value.push(newPathOptions);
      } else {
        newPath.value.path.push({
          lat: event.latLng.lat(),
          lng: event.latLng.lng()
        });
        pathOptions.value.pop();
        
        const newPathOptions = {
          path: newPath.value.path,
          geodesic: true,
          strokeColor: randomColor(),
          strokeOpacity: 1.0,
          strokeWeight: 2,
          editable: true,
        }
        pathOptions.value.push(newPathOptions);
        newPath.value = {
          isStartPointDefined: false,
          path : []
        }
      }
    }

    const updatePathOptions = (path, index) => {
      path.forEach((item, indexPath) => {
        pathOptions.value[index].path[indexPath] = {
          lat: item.lat(),
          lng: item.lng()
        }
      })
    };
    
    onMounted(() => {
      watch(() => map.value.ready, (isReady) => {
       if (!isReady) return;

        const mapInstance = map.value.map;
        pathOptions.value.forEach((item, index) => {
          const polyline = new google.maps.Polyline(item);
          const path = polyline.getPath();
          const infoWindow = new google.maps.InfoWindow({
              content: "<p class='text-black'>test</p>",
            });

          const markerStart = new google.maps.Marker({
            position: path.getAt(0),
            map: mapInstance,
            title: "Start Point",
          });

          const markerEnd = new google.maps.Marker({
            position: path.getAt(path.getLength() - 1),
            map: mapInstance,
            title: "End Point",
          });

          markerStart.addListener("click", () => {
            infoWindow.open(mapInstance, markerStart);
          });

          markerEnd.addListener("click", () => {
            infoWindow.open(mapInstance, markerEnd);
          });
          

          polyline.setMap(mapInstance);

          path.addListener('insert_at', function(vertex) {
            updatePathOptions(this, index)
          });

          path.addListener('set_at', function(vertex) {
            updatePathOptions(this, index)
            markerStart.setPosition(path.getAt(0));
            markerEnd.setPosition(path.getAt(path.getLength() - 1));
          });

          path.addListener('remove_at', function(vertex) {
            updatePathOptions(this, index)
          });

          google.maps.event.addListener(polyline, "dragend", (event) => {
            console.log(event);
          });
          
        });

        mapInstance.addListener("click", (event) => {
          createMapPoint(event);
        });

      }, { immediate: true, deep: true})

      // watch pathOptions
      watch(() => pathOptions.value, (newPathOptions, oldPathOptions) => {
        if (newPathOptions.length === oldPathOptions.length) return;
        if (newPathOptions.length > oldPathOptions.length) {
          const newPolyline = new google.maps.Polyline(newPathOptions[newPathOptions.length - 1]);
          const newPath = newPolyline.getPath();
          newPath.addListener('insert_at', function(vertex) {
            updatePathOptions(this, newPathOptions.length - 1)
          });

          newPath.addListener('set_at', function(vertex) {
            updatePathOptions(this, newPathOptions.length - 1)
          });

          newPath.addListener('remove_at', function(vertex) {
            updatePathOptions(this, newPathOptions.length - 1)
          });
        }
      }, { deep: true })
    })

    return { center, pathOptions, updatePathOptions, createMapPoint, infoRef, openedMarkerID, map, polylineRef };
  },
});
</script>
