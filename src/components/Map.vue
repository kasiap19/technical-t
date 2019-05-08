<template>
    <div class="map" id="map"></div>
</template>

<script>
import axios from 'axios';

export default {
    name: "Map",
    data () { 
        return {
            results: [],
            prices: [],
            average: 0,
        }
    },

    mounted() {
        this.initMap();
    },

    methods: {

        // init map function
        initMap () {

            L.mapbox.accessToken = 'pk.eyJ1Ijoia2FzaWFwMTkiLCJhIjoiY2p1azdzY254MDIwbDRjbzg5MjI1bTBiMCJ9.cracdOs8Bo5nXt46aWTvGw';

            // set map with view London with zoom of 12 
            var map = L.mapbox.map('map')
                .setView([51.505, -0.09], 12)
                .addLayer(L.mapbox.styleLayer('mapbox://styles/mapbox/outdoors-v11'));

                // when you click on the map get zoopla API with bouding_box to get a range od the properties in a given area 
                map.on('click', function(e) {
                    // console.log('latitude: ' + e.latlng.lat.toFixed(6) + ', longitude: ' + e.latlng.lng.toFixed(6))

                    // create a radius of 0.1 mile
                    var radius = 0.1;
                    // get a lat and log on the clicked map (mapbox)
                    var lat = e.latlng.lat.toFixed(6)
                    var lon = e.latlng.lng.toFixed(6)

                    // asign these values to searching properties 
                    // other parameters to reduce the search volume
                    axios.get('http://api.zoopla.co.uk/api/v1/property_listings.js?api_key=w2qyk4gv6f387ckxgafs63ar&latitude='+lat+'&longitude='+lon + '&radius=' + radius+ '&listing_status=sale&include_sold=0&include_rented=0')
                    .then(response => { 
                    this.results = response.data
                    console.log(this.results)

                    //get bounding_box values 
                    const lat1  = this.results.bounding_box.latitude_max
                    const lat2 = this.results.bounding_box.latitude_min
                    const long1 = this.results.bounding_box.longitude_max
                    const long2 = this.results.bounding_box.longitude_min

                    // // click and fit the map into the boundries 
                    var bbox = [[lat1,long1], [lat2,long2]];
                    map.fitBounds(bbox, {
                        maxZoom: 20
                    });

                    // create a marker 
                    var marker = new L.marker(e.latlng).addTo(map);

                    // popup box 

                    // get the prices of properties in given area 
                    this.prices = this.results.listing
                    
                    var sum = 0;
                    var total=this.results.result_count
                    
                    // for loop array of prices to sum all values no matter how big is the arrray and assign to sum variable
                    for (var idx = 0; idx <= this.prices.length - 1; idx++) {
                        sum += parseInt(this.prices[idx].price);
                    }

                    // calculate the average
                    this.average = sum / total
                    
                    // create popup to display total properties in given area and average price of properties
                    marker.bindPopup(
                        "<div class='custom'><div class='custom__total'><span>"+this.results.result_count+" Properties</span></div><div class='custom__content'><p>Average price: <b>£"+this.average.toFixed(2)+"</b></p></div></div>",
                        {autoClose:false,
                        closeOnClick:false}
                    ).openPopup()

                    
                    // create a radius and draw the circle 
                    var radius = 100;
                    var filterCircle = L.circle(L.latLng(e.latlng.lat, e.latlng.lng), radius, {
                        opacity: 1,
                        weight: 1,
                        fillOpacity: 0.4
                    }).addTo(map);
                
                    }) 
                .catch(e => {
                this.errors.push(e)
                })
            });
        },

    }
}
</script>

<style lang="scss">
    .map {
        height: 100vh;
        height: 100vh;
    }
    .leaflet-popup-content {
        padding: 0 !important;
        width: max-content !important;
    }

    .leaflet-popup-content-wrapper {
        border: 3px solid #1b447c;
        padding: 0 !important;
    }

    .custom {
        &__total {
            background: #1b447c;
            padding: 10px;

            span {
                padding: 20px 0;
                font-weight: bold;
                color: #ddd;
                letter-spacing: 1px;
                font-size: 18px;
                height: 30px;
            }
        }

        &__content {
            padding: 20px 20px;

            p {
                color: #1b447c;
                font-size: 20px;
            }
        }
    }

    .leaflet-popup-close-button {  
        border: 1px solid #ddd;
        color: #ddd !important;
    }
</style>

