<template>
  <div id="app">
    <div class="container-fluid">
      <div class="row">
        <div class="col-sm-4">
          <div class="kbox my-3">
            <div class="kbox-title"><h5>Tramas recibidas cada minuto</h5></div>
            <div class="kbox-content p-0">
              <v-chart :options="polar"/>
            </div>
          </div>
        </div>
        <div class="col-sm-4">
          <div class="kbox my-3">
            <div class="kbox-title"><h5>Equipos conectados</h5></div>
            <div class="kbox-content">
            </div>
          </div>
        </div>
        <div class="col-sm-4">
          <div class="kbox my-3">
            <div class="kbox-title"><h5>Puertos</h5></div>
            <div class="kbox-content">
            </div>
          </div>
        </div>
      </div>
      <div class="row">
        <div class="col-md-6 track-list">
          <div class="kbox">
            <div class="kbox-title">
              <h5>Track List
                <span class="label" v-bind:class="[socket.connected ? 'label-primary' : 'label-warning']">{{ socket.connected ? 'Conectado' : 'Desconectado'}}</span>
                <span class="label label-primary pull-right">{{ trackers.length }}</span>
              </h5>
            </div>
            <div class="kbox-content">
              <div class="row">
                <div class="col-sm-8">
                  <input type="text" class="form-control form-control-sm" placeholder="Buscar..." v-model="search">
                </div>
                <div class="col-sm-4">
                  <select v-model="select_port" class="form-control form-control-sm">
                    <option value="">Todos los puertos</option>
                    <option v-for="(port, index) in ports" :key="index" :value="port">Puerto {{port}}</option>
                  </select>
                </div>
              </div>
            </div>
            <div class="kbox-content p-0">
              <ul class="list-group list-group-flush">
                <li class="list-group-item"
                v-for="(track, index) in filterTrackers" :key="index"
                v-on:click="selectTrack(track)"
                v-bind:class="{ 'list-group-item-info': track_select == track }">
                  <span class="label label-info mr-2">{{ index+1 }}</span>
                  <span class="font-weight-bold mr-2">{{ track.imei }}</span>
                  <span class="label">{{ track.tramas[0].receptor }}</span>
                  <span class="pull-right">{{ track.tramas[0].DATETIME }}</span>
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
                  <span class="label label-primary" v-if="new_tram" v-on:click="viewNewTram()">Nueva</span>
                </h5>
                  <div class="pull-right">
                    <select class="form-control form-control-sm" v-model="tram_select" v-on:change="viewNewTram(true)">
                      <option v-for="(tram, index) in track_select.tramas" :key="index" :value="tram">
                        {{ tram.FECHA }} {{ tram.HORA}}
                      </option>
                    </select>
                  </div>
                </div>
              <div class="kbox-content">
                <div class="row">
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
                    {{ tram_select.SENAL }}
                    <div class="value">SEÑAL</div>
                  </div>
                  <div class="col-md-2 tram-param">
                    {{ tram_select.EVENTO }}
                    <div class="value">EVENTO</div>
                  </div>
                  <div class="col-md-2 tram-param">
                    {{ tram_select.IGNICION }}
                    <div class="value">IGNICIÓN</div>
                  </div>
                  <div class="col-md-2 tram-param">
                    {{ tram_select.VELOCIDAD }}
                    <div class="value">VELOCIDAD</div>
                  </div>
                  <div class="col-md-2 tram-param">
                    {{ tram_select.ODOMETRO }}
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
                </div>
              </div>
              <div class="kbox-footer">
                <div class="row text-center">
                  <div class="col-sm-3">
                    <button class="btn btn-primary" v-on:click="sendCommand(1)">Posición</button>
                  </div>
                  <div class="col-sm-3">
                    <button class="btn btn-danger" v-on:click="sendCommand(3)">Apagar</button>
                  </div>
                  <div class="col-sm-3">
                    <button class="btn btn-success" v-on:click="sendCommand(2)">Encender</button>
                  </div>
                  <div class="col-sm-3">
                    <button class="btn btn-warning" v-on:click="sendCommand(4)">Reiniciar</button>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import io from 'socket.io-client';
import ECharts from 'vue-echarts/components/ECharts';
import 'echarts/lib/chart/line';
import 'echarts/lib/component/tooltip'

export default {
  name: 'app',
  components: {
    'v-chart': ECharts
  },
  data() {
    let data = [];

    return {
      data: data,
      history_limit: 7,
      connected: false,
      new_tram: false,
      user: '',
      search: '',
      select_port: '',
      num_trams_now: 0,
      limit_data_graph: 100,
      ports: ['8500','8501','8502','8503','8504','8505','8506','8507','8508','8508','8509','8510','8511','8512','8513','8514'],
      track_select: null,
      tram_select: null,
      trackers: [],//[{"imei":"63977031440261","tramas":[{"trama":"242400636397703144026199553034303330392C412C313032332E343236322C4E2C30373532362E363834392C572C3030302C3333392C3033303131397C31322E327C3139347C303230307C303030302C303445357C30303031393637313697030D0A","puerto":12798,"ip":"127.0.0.1","receptor":8512,"ES_TRAMA_POSICION":1,"ES_TRAMA_EVENTO":0,"ES_TRAMA_LOGIN":0,"ES_TRAMA_RESPUESTA":0,"IGNICION":"0","LONGITUD":"","IMEI":"63977031440261","EVENTO":"9955","EVENTOS":"8,0,19","COD_EVENTO":null,"HORA":"4:03:09","SENAL":"A","LAT":"10.3904367","LNG":"-75.4447483","CARD_LAT":"N","CARD_LNG":"W","VELOCIDAD":0,"ORIENTACION":"339","FECHA":"2019-01-03","HDOP":"12.2","ALTITUD":"194","IN_OUTS":"0000001000000000","AD1":0,"AD2":7.341796875,"AD3":"","AD4":"","AD5":"","ODOMETRO":196716,"ODOMETRO_VIRTUAL":"","RFID":"","BAT_INTERNA":"","BAT_EXTERNA":"","NUM_SATELITES":"","DATETIME":"2019-01-03 4:03:09"}]},{"imei":"63158020465516","tramas":[{"trama":"242400676315802046551699553034303331312E30302C412C313032322E38353239382C4E2C30373532382E36363136302C572C302E3030302C2C3033303131397C31322E327C3139347C303430307C303030302C303030307C30313330373630313148060D0A","puerto":12798,"ip":"127.0.0.1","receptor":8512,"ES_TRAMA_POSICION":1,"ES_TRAMA_EVENTO":0,"ES_TRAMA_LOGIN":0,"ES_TRAMA_RESPUESTA":0,"IGNICION":"0","LONGITUD":"","IMEI":"63158020465516","EVENTO":"9955","EVENTOS":"8,0,19","COD_EVENTO":null,"HORA":"4:03:11","SENAL":"A","LAT":"10.3808830","LNG":"-75.4776933","CARD_LAT":"N","CARD_LNG":"W","VELOCIDAD":0,"ORIENTACION":null,"FECHA":"2019-01-03","HDOP":"12.2","ALTITUD":"194","IN_OUTS":"0000010000000000","AD1":0,"AD2":0,"AD3":"","AD4":"","AD5":"","ODOMETRO":13076011,"ODOMETRO_VIRTUAL":"","RFID":"","BAT_INTERNA":"","BAT_EXTERNA":"","NUM_SATELITES":"","DATETIME":"2019-01-03 4:03:11"}]},{"imei":"63158027485434","tramas":[{"trama":"242400676315802748543499553034303331372E30302C412C313032332E36383330302C4E2C30373532392E38353839332C572C302E3030302C2C3033303131397C31322E327C3139347C303030307C303030302C303030307C303637363730313636C8C70D0A","puerto":12798,"ip":"127.0.0.1","receptor":8512,"ES_TRAMA_POSICION":1,"ES_TRAMA_EVENTO":0,"ES_TRAMA_LOGIN":0,"ES_TRAMA_RESPUESTA":0,"IGNICION":"0","LONGITUD":"","IMEI":"63158027485434","EVENTO":"9955","EVENTOS":"8,0,19","COD_EVENTO":null,"HORA":"4:03:17","SENAL":"A","LAT":"10.3947167","LNG":"-75.4976488","CARD_LAT":"N","CARD_LNG":"W","VELOCIDAD":0,"ORIENTACION":null,"FECHA":"2019-01-03","HDOP":"12.2","ALTITUD":"194","IN_OUTS":"0000000000000000","AD1":0,"AD2":0,"AD3":"","AD4":"","AD5":"","ODOMETRO":67670166,"ODOMETRO_VIRTUAL":"","RFID":"","BAT_INTERNA":"","BAT_EXTERNA":"","NUM_SATELITES":"","DATETIME":"2019-01-03 4:03:17"}]}],
      socket : io('localhost:3006'),
      polar: {
        tooltip: {
          trigger: 'axis',
          formatter: function (params) {
              params = params[0];
              return params.name.substr(-8) + ' - ' + params.value[1] + ' Tramas';
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
      series: [{
          type: 'line',
          showSymbol: false,
          hoverAnimation: true,
          data: data
      }],
        animationDuration: 2000
      }
    }
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
    },
    viewNewTram(verify) {
      if (!verify || verify && this.tram_select == this.track_select.tramas[0]) {
        this.tram_select = this.track_select.tramas[0];
        this.new_tram = false;
      }
    },
    sendCommand(command) {
      if (this.connected) {
        if (confirm('Seguro?'))
          this.socket.emit('command', this.tram_select, command);
      } else {
        alert('Estas desconectado del servidor, verifica tu conexión e intenta de nuevo');
      }
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
    });
    this.socket.on('trama', (data) => {
      let track = this.trackers.find(obj => {
        return obj.imei == data.IMEI
      });
      if (track) {
        track.tramas.unshift(data); // Add start
        if (track.tramas.length >= this.history_limit) {
          track.tramas.pop(); // Remove end
        }
        if (this.track_select) {
          this.new_tram = track.imei == this.track_select.imei;
        }
      } else {
        this.trackers.push({imei: data.IMEI, tramas: [data]});
      }
      this.num_trams_now++;       
    });
    this.socket.on('disconect', () => {
      this.connected = false;
    });
    setInterval(() => {
      if (this.data.length >= this.limit_data_graph) {
        this.data.shift();
      }
      this.data.push({name: (new Date).toLocaleString(), value: [(new Date).toISOString(), this.num_trams_now]});
      this.polar.series.data = this.data;
      this.num_trams_now = 0;
    }, 2000);
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
    height: 200px;
  }
</style>