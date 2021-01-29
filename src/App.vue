<template>
  <div id="app">
    
    <div class="container">
      <div class="center">
        <img alt="Vue logo" src="./assets/logo.png" style="max-width: 75px;">
        <h2>Ricerca Treni Cinque Terre - Vue APP <span class="h6"><b-badge style="background-color:red">New</b-badge></span></h2>
      </div>
      
      <b-form @submit="onSubmit" @reset="onReset" >
        <b-container class="mt-5 mb-5">
          <b-row>
            <b-col md="6" class="mt-3">
              <b-form-group
                id="gr-partenza"
                 :label="`${stazione_partenza}`"
                label-for="partenza"
              >
                <b-form-select required v-model="form.stazione_partenza" id="partenza" :options="stazioni"></b-form-select>
              </b-form-group>
            </b-col>
            <b-col md="6" class="mt-3">
              <b-form-group
                id="gr-arrivo"
                :label="`${stazione_arrivo}`"
                label-for="arrivo"
              >
                <b-form-select required v-model="form.stazione_arrivo" id="arrivo" :options="stazioni"></b-form-select>
              </b-form-group>
            </b-col>
          </b-row>
          <b-row class="mt-3">
            <div class="center">
              <b-button class="m-3" type="submit" variant="primary">Submit</b-button>
              <b-button class="m-3" type="reset" variant="danger">Reset</b-button>
            </div>
          </b-row>
        </b-container>
      </b-form>
      <div class="center">
        
        <span class="h5 mb-3">
          Da: {{ this.form.stazione_partenza }} a: {{ this.form.stazione_arrivo }}<br> <br>          
        </span>

        <b-table striped hover :items="this.items">
          <template #cell(ora_partenza)="data">
                {{timeConverter(data.item.ora_partenza)}}
          </template>
          <template #cell(ora_arrivo)="data">
                {{timeConverter(data.item.ora_arrivo)}}
          </template>
        </b-table>

        <hr>
        <h4>Prossimi TRENI nella stazione di ... </h4>
        <b-table striped :items="this.itemStazione">
          <template #cell(orario)="data">
                {{timeConverter(data.item.orario)}}
          </template>
        </b-table>

        <!-- <ul class="mt-3">
          <li class="myli" v-for="dati in this.info" :key="dati.id">
            <span v-if="dati.saleable">
              {{timeConverter(dati.departuretime)}} 
              <b-icon icon="arrow-right"></b-icon> {{timeConverter(dati.arrivaltime)}} 
              <em>({{dati.duration}})</em> - Costo: € {{dati.originalPrice}} - Tipo treno: {{dati.trainlist[0].trainidentifier}}
            </span>
          </li>
        </ul> -->

      </div>
    
    </div>
        
  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'App',
  components: {
    
  },
  data() {
    return {
      stazione_partenza: 'Stazione di partenza: ',
      stazione_arrivo: 'Stazione di arrivo: ',
      urlStazione: '',
      infoStazione: '',
      form: {
        stazione_partenza: '',
        stazione_arrivo: '',
      },
      stazioni: [
        { value: 'Levanto', text: 'Levanto' },
        { value: 'Monterosso', text: 'Monterosso' },
        { value: 'Vernazza', text: 'Vernazza' },
        { value: 'Corniglia', text: 'Corniglia' },
        { value: 'Manarola', text: 'Manarola' },
        { value: 'Riomaggiore', text: 'Riomaggiore' },
        { value: 'La Spezia Centrale', text: 'La Spezia Centrale' },
      ],
      infoStazioni: [
        { value: 'S04731', text: 'Levanto' },
        { value: 'S04732', text: 'Monterosso' },
        { value: 'S04733', text: 'Vernazza' },
        { value: 'S04734', text: 'Corniglia' },
        { value: 'S04735', text: 'Manarola' },
        { value: 'S04736', text: 'Riomaggiore' },
        { value: 'S06000', text: 'La Spezia Centrale' },
      ],
      now: null,
      nowgmt: null,
      adate: null,
      atime: null,
      myUrl: '',
      info: '',
      items:'',
      itemStazione: '',
      fields: [
        'Ora partenza',
        'Ora arrivo',
        'Durata',
        'Prezzo biglietto',
        'Dettaglio treno'
      ],
    }
  },
  methods:{
    onSubmit(event) {
        event.preventDefault()
        
        if (this.form.stazione_arrivo == this.form.stazione_partenza){
          alert ("seleziona due stazioni differenti!");
        }
        else{
          this.now = Date.now();
          this.nowgmt = new Date(this.now);
          this.adate = this.dateConverter(this.now);
          this.atime = this.timeConverter(this.now);

          this.myUrl = 'https://www.lefrecce.it/msite/api/solutions?origin='+
          this.form.stazione_partenza+'&destination='+
          this.form.stazione_arrivo+'&arflag=A&adate='+this.adate+'&atime='+this.atime+
          '&adultno=1&childno=0&direction=A&frecce=false&onlyRegional=false';

          const PROXY = window.location.hostname === "localhost"
            ? "https://cors-anywhere.herokuapp.com"
            : "/cors-proxy";

          axios
            .get(`${PROXY}/`+this.myUrl)
            .then(res => {
              this.info = res.data;
              this.items = this.info.map(x => ({
                        ora_partenza: x.departuretime,
                        ora_arrivo: x.arrivaltime,
                        durata: x.duration,
                        costo_biglietto: x.originalPrice,
                        dettaglio_treno: x.trainlist[0].trainidentifier,
              }))
            })
            let vernazza = 'S04733'
            this.urlStazione = 'http://www.viaggiatreno.it/viaggiatrenonew/resteasy/viaggiatreno/partenze/'+vernazza+'/'+this.nowgmt;

            axios
              .get(`${PROXY}/`+this.urlStazione)
              .then(result => {
                this.infoStazione = result.data;
                this.itemStazione = this.infoStazione.map(y => ({
                    destinazione: y.destinazione,
                    orario: y.orarioPartenza,
                    numeroTreno: y.numeroTreno

                }));
                
              })


        }

        

    },
    onReset(event) {
      event.preventDefault()
      // Reset our form values
      this.form.stazione_partenza = ''
      this.form.stazione_arrivo = ''
      
    },
    timeConverter(UNIX_timestamp){
      // let orario = new Date(UNIX_timestamp).toLocaleTimeString("it-IT")  
      let orario = new Date(UNIX_timestamp).getHours()+":"+new Date(UNIX_timestamp).getMinutes();
      
      return orario;
    },
    dateConverter(UNIX_timestamp){
      let data = new Date(UNIX_timestamp).toLocaleDateString("it-IT");
      return data;
    },
  }
}
</script>

<style>
#app { 
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: left;
  color: #2c3e50;
  margin-top: 60px;
}

.center{
  margin: 0 auto;
  text-align: center;
}

.myli{
  list-style-type: none !important;
}
</style>
