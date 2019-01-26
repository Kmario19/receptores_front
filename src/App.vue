<template>
  <div id="app">
    <div class="container-fluid">
      <div class="row">
        <div class="col-sm-4">
          <div class="kbox my-3">
            <div class="kbox-title">
              <h5><span v-if="false" v-on:click="chart_pause = !chart_pause" style="cursor: pointer">{{ chart_pause ? '▶️' : '⏸️'}}</span>Tramas recibidas</h5>
              <span class="pull-right">
                <select v-model="trams_frecuency" v-on:change="changeFrecuency()" class="form-control form-control-sm">
                  <option value="second">Por segundo</option>
                  <option value="minute">Por minuto</option>
                  <option value="hour">Por hora</option>
                </select>
              </span>
            </div>
            <div class="kbox-content p-0">
              <v-chart :options="chart_tramas_recibidas"/>
            </div>
          </div>
        </div>
        <div class="col-sm-4">
          <div class="kbox my-3">
            <div class="kbox-title"><h5>Equipos conectados <span class="pull-right">Total <span class="label label-primary">{{ trackers.length }}</span></span></h5></div>
            <div class="kbox-content p-0">
              <v-chart :options="chart_equipos_conectados"/>
            </div>
          </div>
        </div>
        <div class="col-sm-4">
          <div class="kbox my-3">
            <div class="kbox-title"><h5>Puertos</h5></div>
            <div class="kbox-content p-0">
              <v-chart :options="chart_puertos"/>
            </div>
          </div>
        </div>
      </div>
      <div class="row">
        <div class="col-md-6 track-list">
          <div class="kbox">
            <div class="kbox-title">
              <h5>Lista de equipos
                <span class="label pull-right" v-bind:class="[socket.connected ? 'label-primary' : 'label-warning']">{{ socket.connected ? 'Conectado' : 'Desconectado'}}</span>
              </h5>
            </div>
            <div class="kbox-content">
              <div class="row">
                <div class="col-sm-6">
                  <input type="text" class="form-control form-control-sm" placeholder="Buscar..." v-model="search">
                </div>
                <div class="col-sm-3">
                  <select v-model="select_port" class="form-control form-control-sm">
                    <option value="">Puerto [Todos]</option>
                    <option v-for="(port, index) in ports" :key="index" :value="port.name">{{port.name}} ({{port.value}})</option>
                  </select>
                </div>
                <div class="col-sm-3">
                  <select v-model="select_status" class="form-control form-control-sm">
                    <option value="">Estado [Todos]</option>
                    <option v-for="(status, index) in chart_equipos_conectados.series[0].data" :key="index" :value="status.name == 'Conectados' ? 1 : -1">{{status.name}} ({{status.value}})</option>
                  </select>
                </div>
              </div>
            </div>
            <div class="kbox-content tracker-list">
              <ul class="list-group list-group-flush">
                <li class="list-group-item"
                v-for="(track, index) in filterTrackers" :key="index"
                v-on:click="selectTrack(track)"
                v-bind:class="{ 'list-group-item-info': track_select == track }">
                  <span class="label label-info mr-2">{{ index+1 }}</span>
                  <span class="font-weight-bold mr-2">{{ track.imei }}</span>
                  <span class="label">{{ track.tramas[0].receptor }}</span>
                  <span class="pull-right">
                    {{ track.tramas[0].DATETIME }}
                    <span v-if="track.online" class="label label-primary track-status" title="Conectado"></span>
                    <span v-if="!track.online" class="label label-danger track-status" title="Desconectado"></span>
                  </span>
                </li>
                <li class="list-group-item" v-if="!filterTrackers.length">
                  <div class="alert alert-info">Sin Resultados</div>
                </li>
              </ul>
            </div>
          </div>
        </div>
        <div class="col-md-6">
          <div v-if="track_select">
            <div class="kbox">
              <div class="kbox-title">
                <h5>Detalles de Trama
                  <span class="label label-primary" v-if="new_tram" v-on:click="changeTram()">Nueva</span>
                </h5>
                  <div class="pull-right">
                    <select class="form-control form-control-sm" v-model="tram_select" v-on:change="changeTram(true)">
                      <option v-for="(tram, index) in track_select.tramas" :key="index" :value="tram">
                        [{{index+1}}] {{ tram.FECHA }} {{ tram.HORA}}
                      </option>
                    </select>
                  </div>
                </div>
              <div class="kbox-content">
                <div class="row">
                  <div class="col-12">
                    <span v-if="tram_select.ES_TRAMA_LOGIN" class="label label-default">LOGIN</span>
                    <span v-if="tram_select.ES_TRAMA_POSICION" class="label label-info">POSICIÓN</span>
                    <span v-if="tram_select.ES_TRAMA_EVENTO" class="label label-warning">EVENTO</span>
                    <span v-if="tram_select.ES_TRAMA_RESPUESTA" class="label label-success">RESPUESTA</span>
                  </div>
                </div>
                <div class="row mt-2">
                  <div class="col-md-2 tram-param">
                    {{ tram_select.FECHA }}
                    <div class="value">FECHA</div>
                  </div>
                  <div class="col-md-2 tram-param">
                    {{ tram_select.HORA }}
                    <div class="value">HORA</div>
                  </div>
                  <div class="col-md-2 tram-param">
                    {{ tram_select.receptor }}
                    <div class="value">RECEPTOR</div>
                  </div>
                  <div class="col-md-2 tram-param">
                    {{ tram_select.ip }}
                    <div class="value">IP</div>
                  </div>
                  <div class="col-md-2 tram-param">
                    {{ tram_select.puerto }}
                    <div class="value">PUERTO</div>
                  </div>
                  <div class="col-md-2 tram-param">
                    {{ tram_select.modelo }}
                    <div class="value">MODELO</div>
                  </div>
                </div>
                <div class="row mt-2">
                  <div class="col-md-2 tram-param">
                    <span class="label" v-bind:class="[tram_select.SENAL == 'A' ? 'label-primary' : 'label-danger']">{{ tram_select.SENAL }}</span>
                    <div class="value">SEÑAL</div>
                  </div>
                  <div class="col-md-2 tram-param">
                    {{ tram_select.EVENTO }}
                    <div class="value">EVENTO</div>
                  </div>
                  <div class="col-md-2 tram-param">
                    <span class="label" v-bind:class="[tram_select.IGNICION == '1' ? 'label-primary' : 'label-danger']">{{ tram_select.IGNICION == '1' ? 'ON' : 'OFF' }}</span>
                    <div class="value">IGNICIÓN</div>
                  </div>
                  <div class="col-md-2 tram-param">
                    {{ tram_select.VELOCIDAD }}Km/h
                    <div class="value">VELOCIDAD</div>
                  </div>
                  <div class="col-md-2 tram-param">
                    {{ tram_select.ODOMETRO }}Km
                    <div class="value">ODOMETRO</div>
                  </div>
                    <div class="col-md-2 tram-param">
                    {{ tram_select.ODOMETRO_VIRTUAL || 'NA' }}
                    <div class="value">ODO VIRT</div>
                  </div>
                </div>
                <div class="row mt-2">
                  <div class="col-md-2 tram-param">
                    {{ tram_select.LAT }}
                    <div class="value">LATITUD</div>
                  </div>
                  <div class="col-md-2 tram-param">
                    {{ tram_select.LNG }}
                    <div class="value">LONGITUD</div>
                  </div>
                  <div class="col-md-2 tram-param">
                    {{ tram_select.ORIENTACION }}
                    <div class="value">ORIENTACIÓN</div>
                  </div>
                  <div class="col-md-2 tram-param">
                    {{ tram_select.AD1 }}
                    <div class="value">AD1</div>
                  </div>
                  <div class="col-md-2 tram-param">
                    {{ tram_select.AD2 }}
                    <div class="value">AD2</div>
                  </div>
                </div>
                <div class="row mt-2">
                  <googlemaps-map
                    :center.sync="map.center"
                    :zoom.sync="map.zoom"
                    :options="map.options">

                    <googlemaps-marker
                      v-for="marker of map.markers"
                      :key="marker.id"
                      :label="{
                        color: marker === map.currentmarker ? 'white' : 'black',
                        text: (marker.id+1)+'',
                      }"
                      :position="marker.position"
                      @click="selectMarker(marker.id)"
                    />
                  </googlemaps-map>
                </div>
              </div>
              <div class="kbox-footer">
                <div class="row text-center">
                  <div class="col-sm-3">
                    <button class="btn btn-primary" v-on:click="sendCommand(3)">Posición</button>
                  </div>
                  <div class="col-sm-3">
                    <button class="btn btn-danger" v-on:click="sendCommand(2)">Apagar</button>
                  </div>
                  <div class="col-sm-3">
                    <button class="btn btn-success" v-on:click="sendCommand(1)">Encender</button>
                  </div>
                  <div class="col-sm-3">
                    <button class="btn btn-warning" v-on:click="sendCommand(4)">Reiniciar</button>
                  </div>
                </div>
              </div>
            </div>
          </div>
          <div v-else>
            <div class="kbox">
              <div class="kbox-title"><h5>Detalles de trama</h5></div>
              <div class="kbox-content">
                <div class="alert alert-info">Seleccione un dispositivo para ver su información</div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import io from 'socket.io-client'
import ECharts from 'vue-echarts/components/ECharts'
import 'echarts/lib/chart/line'
import 'echarts/lib/chart/bar'
import 'echarts/lib/chart/pie'
import 'echarts/lib/component/tooltip'
import 'echarts/lib/component/legend'
import 'echarts/lib/component/dataZoom'
import 'vue-googlemaps/dist/vue-googlemaps.css'
import Vue from 'vue'
import VueGoogleMaps from 'vue-googlemaps'

Vue.use(VueGoogleMaps, {
  load: {
    // Google API key
    apiKey: 'AIzaSyAuUgBnqfQUiBNeRQmrrWeoLcjlvTvPbII',
    // Enable more Google Maps libraries here
    //libraries: ['places'],
    // Use new renderer
    useBetaRenderer: false,
  },
})

export default {
  name: 'app',
  components: {
    'v-chart': ECharts
  },
  data() {
    return {
      map: {
        center: {
          lat: 10.3845791,
          lng: -75.4817934,
        },
        zoom: 17,
        options: {},
        markers: [],
        currentmarker: null
      },
      data_tramas_recibidas: [],
      history_limit: 7,
      connected: false,
      new_tram: false,
      user: '',
      search: '',
      select_port: '',
      select_status: '',
      counters: {
        date: (new Date).toLocaleDateString().replace(/-(\d)(?!\d)/g, '-0$1'),
        trams: {
          per_hour: {total: 0, history: []},
          per_minute: {total: 0, history: []},
          per_second: {total: 0, history: []}
        },
        trackers: {
          per_hour: {connected: 0, disconnected: 0, history: []},
        }
      },
      trams_frecuency: 'minute',
      chart_pause: false,
      last_minute: 0,
      last_hour: 0,
      data_counter_limit: 50,
      ports: [],
      track_select: null,
      tram_select: null,
      trackers: [],
      socket : io('https://yeci.online:3006'),
      chart_tramas_recibidas: {
        color: ['#1ab394'],
        tooltip: {
          trigger: 'axis',
          formatter: function (params) {
            params = params[0];
            return params.name.substr(-8, 8) + ' - ' + params.value[1] + ' Tramas';
          },
          axisPointer: {
            animation: false
          }
        },
        xAxis: {
          type: 'time',
          splitLine: {
            show: true
          }
        },
        yAxis: {
          type: 'value',
          splitLine: {
            show: true
          }
        },
        dataZoom: {
          show : true,
          realtime: true,
          start : 0,
          end : 100
        },
        series: [{
          type: 'line',
          itemStyle: {normal: {areaStyle: {type: 'default'}}},
          showSymbol: false,
          hoverAnimation: true,
          data: []
        }],
        animationDuration: 2000
      },
      chart_equipos_conectados: {
        color: ['#1ab394','#ed5565'],
        tooltip: {
            trigger: 'item',
            formatter: function (p) {
              return '<span style="color:' + p.color + ';font-size:1.2em">\u25CF</span> ' + p.name + ' ' + p.value + ' (' + (p.percent - 0).toFixed(0) + '%)';
            }
        },
        legend: {
            orient : 'vertical',
            x : 'left',
            data:['Conectados', 'Desconectados']
        },
        calculable: true,
        series : [
            {
                type: 'pie',
                data:[
                    {value:0, name: 'Conectados'},
                    {value:0, name: 'Desconectados'}
                ],
                itemStyle : {
                  normal : {
                      label : {
                          position : 'inner',
                          formatter : function (params) {                   
                            return (params.value - 0).toFixed(0)
                          },
                          fontWeight: '600'
                          
                      }
                  }
                }
            }
        ]
      },
      chart_puertos: {
        color: ['#1ab394','#ed5565', '#d1dade'],
        tooltip: {
            trigger: 'axis'
        },
        xAxis: [
            {
                type: 'category',
                data: [],
                name: 'Puerto'
            }
        ],
        yAxis: [
            {
                type: 'value',
                name: 'Equipos',
            }
        ],
        series: [
          {type: 'bar', stack: 'vehiculos', name: 'Conectados', data: []},
          {type: 'bar', stack: 'vehiculos', name: 'Desconectados', data: []},
          {type: 'line', name: 'Total', data: []}
        ]
      }
    }
  },
  watch: {
    /*map: {
      currentmarker: function(newMarker, oldMarker) {
        
      }
    }*/
  },
  methods: {
    sendMessage(e) {
      e.preventDefault();
      this.socket.emit('trama', this.last);
      this.message = '';
    },
    selectTrack(track) {
      this.track_select = track;
      this.tram_select = track.tramas[0];
      this.new_tram = false;
      this.loadMarkers();
    },
    changeTram(verify) {
      if (!verify || verify && this.tram_select == this.track_select.tramas[0]) {
        this.tram_select = this.track_select.tramas[0];
        this.new_tram = false;
        this.loadMarkers();
      } else {
        for (let i = 1; i < this.track_select.tramas.length; i++) {
          if (this.track_select.tramas[i] == this.tram_select) {
            this.map.currentmarker = this.map.markers.find(m => m.id == i);
            break;
          }
        }
      }
      this.map.center = this.map.currentmarker ? this.map.currentmarker.position : this.map.center;
    },
    changeFrecuency() {
      switch(this.trams_frecuency) {
        case 'second':
          this.chart_tramas_recibidas.series[0].data = this.counters.trams.per_second.history;
          break;
        case 'minute':
          this.chart_tramas_recibidas.series[0].data = this.counters.trams.per_minute.history;
          break;
        case 'hour':
          this.chart_tramas_recibidas.series[0].data = this.counters.trams.per_hour.history;
          break;
      }
    },
    sendCommand(command) {
      if (this.connected) {
        if (confirm('Seguro?'))
          this.socket.emit('command', this.tram_select, command);
      } else {
        alert('Estas desconectado del servidor, verifica tu conexión e intenta de nuevo');
      }
    },
    updateChartEquiposConectados() {
      this.chart_equipos_conectados.series[0].data[0].value = this.trackers.filter(t => t.online).length;// Conectados
      this.chart_equipos_conectados.series[0].data[1].value = this.trackers.filter(t => !t.online).length;// Conectados
    },
    updateChartPuertos() {
      var puertos = this.trackers.reduce((p, t) => {
        var puerto = t.tramas[0].receptor;
        if (!p.hasOwnProperty(puerto)) {
          p[puerto] = 0;
        }
        p[puerto]++;
        return p;
      }, {});
      this.ports = Object.keys(puertos).map(p => {
        return {
          name: p,
          value: puertos[p],
          conectados: this.trackers.filter(t => t.online && t.tramas[0].receptor == p).length,
          desconectados: this.trackers.filter(t => !t.online && t.tramas[0].receptor == p).length
        };
      });
      this.chart_puertos.xAxis[0].data = [];
      this.chart_puertos.series[0].data = [];
      this.chart_puertos.series[1].data = [];
      this.chart_puertos.series[2].data = [];
      for (let i = 0; i < this.ports.length; i++) {
        var p = this.ports[i];
        this.chart_puertos.xAxis[0].data.push(p.name);
        this.chart_puertos.series[0].data.push(p.conectados);
        this.chart_puertos.series[1].data.push(p.desconectados);
        this.chart_puertos.series[2].data.push(p.value);
      }
    },
    clearMarkers: function () {
      /*for (let i = 0; i < this.map.markers.length; i++) {
        const mark = this.map.markers[i];
        mark.setMap(null);
      }*/
      this.map.markers = [];
    },
    loadMarkers: function () {
      this.clearMarkers();
      for (let i = 0; i < this.track_select.tramas.length; i++) {
        const trama = this.track_select.tramas[i];
        if (trama.LAT && trama.LNG) {
          this.map.markers.push({
            id: i,
            position: {lat: parseFloat(trama.LAT), lng: parseFloat(trama.LNG)}
          });
        }
      }
      this.map.currentmarker = this.map.markers[0] || null;
      this.map.center = this.map.currentmarker ? this.map.currentmarker.position : this.map.center;
    },
    selectMarker: function(id) {
      this.map.currentmarker = this.map.markers.find(m => m.id == id);
      this.tram_select = this.track_select.tramas[id];
      this.map.center = this.map.currentmarker.position;
    }
  },
  mounted() {
    /* eslint-disable */
    this.socket.on('connect', () => {
      this.connected = true;
      this.socket.emit('login', {user: 'server'});
    });
    this.socket.on('tracker_list', (tracker_list) => {
      this.trackers = tracker_list;
      this.updateChartEquiposConectados();
      this.updateChartPuertos();
    });
    this.socket.on('counters', (counters) => {
      this.counters = counters;
    });
    this.socket.on('trama', (data) => {
      let track = this.trackers.find(obj => {
        return obj.imei == data.IMEI
      });
      if (track) {
        track.tramas.unshift(data); // Add start
        if (track.tramas.length > this.history_limit) {
          track.tramas.pop(); // Remove end
        }
        if (this.track_select && !this.new_tram) {
          this.new_tram = track.imei == this.track_select.imei;
        }
        if (!track.online) { // Estaba desconectado
          track.online = true;
          this.updateChartEquiposConectados();
          this.updateChartPuertos();
          console.log('Reconectado', track.imei);
        }
        track.socket = this.socket.id;
      } else {
        this.trackers.push({imei: data.IMEI, online: true, socket: this.socket.id, tramas: [data]});
        this.updateChartEquiposConectados();
        this.updateChartPuertos();
      }
      this.counters.trams.per_hour.total++;
      this.counters.trams.per_minute.total++;
      this.counters.trams.per_second.total++;
      if (data.error) {
        this.counters.trams.errors.total++;
      }
    });
    this.socket.on('track_disconnect', (data) => {
      console.log('Se desconectó: ', data);
      let track = this.trackers.find(obj => {
        return obj.imei == data
      });
      if (track) {
        track.online = false;
      }
      this.updateChartEquiposConectados();
      this.updateChartPuertos();
    });
    this.socket.on('disconect', () => {
      this.connected = false;
    });
    setInterval(() => {
			if (this.counters.trams.per_second.history.length > this.data_counter_limit) {
				this.counters.trams.per_second.history.shift();
			}
			this.counters.trams.per_second.history.push({name: (new Date).toLocaleString(), value: [(new Date).toISOString(), this.counters.trams.per_second.total]});
			this.counters.trams.per_second.total = 0;
			if (this.last_minute >= 60) {
				if (this.counters.trams.per_minute.history.length > this.data_counter_limit) {
					this.counters.trams.per_minute.history.shift();
				}
				this.counters.trams.per_minute.history.push({name: (new Date).toLocaleString(), value: [(new Date).toISOString(), this.counters.trams.per_minute.total]});
				this.counters.trams.per_minute.total = 0;
				this.last_minute = 0;
				this.counters.trackers.per_hour.connected = this.trackers.filter(t => t.online).length;
				this.counters.trackers.per_hour.disconnected = this.trackers.filter(t => !t.online).length;
				this.counters.trackers.per_hour.history.push({name: (new Date).toLocaleString(), connected: this.counters.trackers.per_hour.connected, disconnected: this.counters.trackers.per_hour.disconnected});
				if (this.counters.date != (new Date).toLocaleDateString().replace(/-(\d)(?!\d)/g, '-0$1')) {
					this.counters = {
						date: (new Date).toLocaleDateString().replace(/-(\d)(?!\d)/g, '-0$1'),
						trams: {
							per_hour: {total: 0, history: []},
							per_minute: {total: 0, history: []},
              per_second: {total: 0, history: []},
              errors: {total: 0}
						},
						trackers: {
							per_hour: {connected: 0, disconnected: 0, history: []},
						}
					};
				}
			}
			if (this.last_hour >= 3600) {
				this.counters.trams.per_hour.history.push({name: (new Date).toLocaleString(), value: [(new Date).toISOString(), this.counters.trams.per_hour.total]});
				this.counters.trams.per_hour.total = 0;
				this.last_hour = 0;
			}
			this.last_minute++;
      this.last_hour++;
      if (!this.chart_pause) {
        this.changeFrecuency();
      }
		}, 1000);
  },
  computed: {
    filterTrackers: function() {
      let filtered = this.trackers;
      if (this.search) {
        filtered = this.trackers.filter(
          t => t.imei.toLowerCase().indexOf(this.search) > -1
        );
      }
      if (this.select_port) {
        filtered = filtered.filter(
          t => t.tramas[0].receptor == this.select_port
        );
      }
      if (this.select_status != '') {
        filtered = filtered.filter(
          t => (t.online && this.select_status == 1) || (!t.online && this.select_status == -1)
        );
      }
      return filtered;
    }
  }
}
</script>

<style>
  .tram-param .value {
    font-weight: 500
  }
  .echarts {
    width: 100%;
    height: 200px;
  }
  .track-status {
    border-radius: 30px;
    padding: 5px;
    height: 15px;
    width: 15px;
    display: inline-block;
    vertical-align: text-bottom;
  }
  .tracker-list {
    padding: 0;
    max-height: 440px;
    overflow: auto;
  }
  .vue-google-map {
    width: 100%;
    height: 250px;
  }
</style>