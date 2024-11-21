地図上に検索ボックスを入れたいが地図の下に隠れてしまっている。
z-indexで数字を調整してくれても上に来てくれない。何故なのだろう。
_map.html.erb```
<div id="map" style="height: 600px; position: relative;">
  <div id="map-" style="position: absolute; top: 10px; left: 50%; transform: translateX(-50%); background-color: white; padding: 10px; border-radius: 5px; border: 1px solid #ccc;">
    <input id="map-search-container" type="text" placeholder="場所を検索" style="width: 300px; padding: 5px; border: none; outline: none; z-index; 9999" />
  </div>
</div>

<script>
  function initMap() {
    const mapOptions = {
      center: { lat: 35.6803997, lng: 139.7690174 }, // 初期位置（例として東京）
      zoom: 10 
    };

    const map = new google.maps.Map(document.getElementById("map"), mapOptions);

    if (navigator.geolocation) {
      navigator.geolocation.getCurrentPosition(
        function (position) {
          const userLocation = {
            lat: position.coords.latitude,
            lng: position.coords.longitude,
          };

          map.setCenter(userLocation);

          new google.maps.Marker({
            position: userLocation,
            map: map,
            title: "あなたの今いる場所",
          });
        },
        function (error) {
          alert("現在地の取得に失敗しました: " + error.message);
        }
      );
    } else {
      alert("Geolocationはこのブラウザでサポートされていません");
    }

    const input = document.getElementById("search-box");
    const searchBox = new google.maps.places.SearchBox(input);
    searchBox.addListener("places_changed", function () {
      const places = searchBox.getPlaces();

      if (places.length === 0) {
        return;
      }

      const bounds = new google.maps.LatLngBounds();

      places.forEach(function (place) {
        if (!place.geometry || !place.geometry.location) {
          console.log("Returned place contains no geometry");
          return;
        }

        new google.maps.Marker({
          map: map,
          position: place.geometry.location,
          title: place.name,
        });

        if (place.geometry.viewport) {
          bounds.union(place.geometry.viewport);
        } else {
          bounds.extend(place.geometry.location);
        }
      });

      map.fitBounds(bounds);
    });
  }
</script>

<script src="https://maps.googleapis.com/maps/api/js?key=<%= ENV["GOOGLE_MAPS_API_KEY"] %>&libraries=places&callback=initMap" async defer></script>
```
