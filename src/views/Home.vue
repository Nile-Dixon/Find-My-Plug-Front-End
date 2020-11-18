<template>
  <div class="home">
  	<v-app>
  		<v-app-bar app>
  			<v-app-bar-nav-icon></v-app-bar-nav-icon>
  			<v-toolbar-title>Find My Plug</v-toolbar-title>
  		</v-app-bar>
  		<v-main>
  			<v-container>
	  			<v-row>
	  				<v-col cols = "12" md = "12">
	  					<v-card min-height = "100px">
	  						<v-row>
	  							<v-container>
		  							<v-col cols = "12" md = "10">
		  								<v-text-field label = "Start Address" v-model = "start_address"></v-text-field>
		  								<v-text-field label = "End Address" v-model = "end_address"></v-text-field>
		  								<v-btn v-on:click = "plot_map">Search for Nearest Charging Station</v-btn>
		  							</v-col>
		  						</v-container>
	  						</v-row>
	  					</v-card>
	  				</v-col>
	  			</v-row>
	  			<v-row>
	  				<v-col cols = "12" md = "6">
	  					<v-card min-height = "400px">
	  						<div id = "map" style="height: 400px; width: 100%;">
	  							
	  						</div>
	  					</v-card>
	  				</v-col>
	  				<v-col cols = "12" md = "6">
	  					<v-card height = "400px"  class = "scroll">
	  						<h2 class = "text-center">Directions</h2>
	  						<v-list>
	  							<v-list-item v-for="direction in directions">
	  								<v-list-item-content>
	  									<v-list-item-title v-html = "direction"></v-list-item-title>
	  								</v-list-item-content>
	  							</v-list-item>
	  						</v-list>
	  					</v-card>
	  				</v-col>
	  			</v-row>
	  		</v-container>
  		</v-main>
  	</v-app>
  </div>
</template>

<script>

export default {
  name: 'Home',
  data : function() {
  	return {
  		'start_address' : '',
  		'end_address' : '',
  		'map' : null,
  		'directions' : []
  	}
  },
  mounted() {
  	this.map = L.map('map').setView([51.505, -0.09], 13);
  	L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
    	attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors'
	}).addTo(this.map);
  },
  methods : {
  	plot_map : function() {
  		var self = this;
  		window.axios.post('http://localhost:5000/api/directions',{
  			'start_address' : self.start_address,
  			'end_address' : self.end_address
  		}).then(function(response){
  			if (response.data.status == "error") {
  				alert("This search did not work. Please try again.")
  			}
  			else {
  				var polyline = L.polyline(response.data.station_polyline, {color: 'blue'}).addTo(self.map);
  				self.map.fitBounds(polyline.getBounds());
  				L.marker([response.data.closest_station.lat, response.data.closest_station.lng])
  				.bindPopup('Name: ' + response.data.closest_station.name + ', <br>Address: ' + response.data.closest_station.address)
  				.addTo(self.map);
  				self.directions = response.data.station_directions;
  			}
  		})
  	}
  }
}
</script>
<style type="text/css">
	.scroll {
		overflow-y: scroll
	}
</style>