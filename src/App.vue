<script>
import axios from 'axios';

export default {
  data() {
    return {
      period: 144,
      freq:'h',
      seasonality :'auto',
      seasonality_period : 7,
      f_order : 5,
      changepoint_prior_scale : 0.05,
      seasonality_prior_scale : 10,
      holidays_prior_scale : 10,
      seasonality_mode : 'additive',
      changepoint_range : 0.8,
      growth : 'linear',
      daily_seasonality : 'auto',
      csvData: null,
      response: null,
    };
  },

  methods: {
    async handleCsvFile(event) {
      const file = event.target.files[0];

      // Converte o arquivo CSV em JSON usando a função nativa do JavaScript
      const fileReader = new FileReader();
      fileReader.readAsText(file);
      fileReader.onload = () => {
        const jsonData = this.csvToJson(fileReader.result);
        this.csvData = jsonData;
        // Salva o arquivo JSON localmente usando o FileSaver.js
        //const blob = new Blob([JSON.stringify(jsonData, null, 2)], {type: "application/json"});
        //saveAs(blob, "arquivo_depuracao.json");
      };
    },

    csvToJson(csv) {
      const lines = csv.split("\r\n");
      const result = [];

      const headers = lines[0].split(';');
      for (let i = 1; i < lines.length; i++) {
        const obj = {};
        const currentLine = lines[i].split(';');

        for (let j = 0; j < headers.length; j++) {
          obj[headers[j]] = currentLine[j];
        }

        result.push(obj);
      }

      return result;
    },

    async handleSubmit() {
      try {
        const payload = {
          period: this.period,
          freq:this.freq,
          seasonality :this.seasonality,
          seasonality_period : this.seasonality_period,
          f_order : this.f_order,
          changepoint_prior_scale : this.changepoint_prior_scale,
          seasonality_prior_scale : this.seasonality_prior_scale,
          holidays_prior_scale : this.holidays_prior_scale,
          seasonality_mode : this.seasonality_mode,
          changepoint_range : this.changepoint_range,
          growth : this.growth,
          daily_seasonality : this.daily_seasonality,
          //data: JSON.parse(this.csvData),
          data: this.csvData,
        };
        // Salva o arquivo JSON localmente usando o FileSaver.js
        const blob2 = new Blob([JSON.stringify(payload, null, 2)], {type: "application/json"});
        saveAs(blob2, "payload.json");

        const response = await axios.post('https://x93azsspdk.execute-api.us-east-1.amazonaws.com/prod', payload, { headers: {'Content-Type': 'application/json'} });
        this.response = response.data;
        console.log(response.data)
      } catch (error) {
        console.error(error);
        this.response = `Erro ao conectar à API: ${error.message}`;
      }
    },
  },
};
</script>

<template>
  <div class="container">
    <h1>Previsão de séries temporais</h1>
    <form @submit.prevent="handleSubmit">
      <div class="form-group">
        <label for="period">Período:</label>
        <input type="number" class="form-control" id="period" v-model="period" required>
      </div>
      <div class="form-group">
        <label for="csv-file">Arquivo CSV:</label>
        <input type="file" class="form-control-file" id="csv-file" @change="handleCsvFile">
      </div>
      <button class="btn btn-primary" type="submit">Enviar</button>
    </form>
  </div>    
</template>   
