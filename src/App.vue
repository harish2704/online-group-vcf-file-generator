<template>
  <main>
    <div class="py-5">
      <h2>Vcf generator</h2>
      <section v-if="step === 1">
        <div class="mb-3">
          <label class="form-label">Paste csv data</label>
          <textarea class="form-control" rows="10" v-model="csvStr"></textarea>
        </div>
        <button class="btn btn-primary" @click="step = 2">Next</button>
      </section>

      <section v-if="step === 2">
        <h4>Data Preview</h4>
        <button class="btn btn-primary" @click="step = 3">Next</button>
        <div class="col-md-12 csv-preview">
          <table class="table table-striped-columns">
            <thead>
              <tr>
                <th v-for="(col, i) in csvData[0]">
                  <strong>
                    {{ col }}
                  </strong>
                </th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="row in csvData.slice(1)">
                <td v-for="col in row">{{ col }}</td>
              </tr>
            </tbody>
          </table>
        </div>

        <button class="btn btn-primary" @click="step = 3">Next</button>
      </section>

      <section v-if="step === 3">
        <div class="col-md-12">
          <strong>Available Columns</strong>
          <br />
          <span class="float" v-for="col in availableCols">
            <code>{{ col }}</code
            >,
          </span>
        </div>
        <div class="row">
          <div class="col-sm-12 col-md-6">
            <div class="card">
              <div class="card-header">Output format</div>
              <div class="card-body">
                <div class="col-md-12">
                  <label class="form-label">Name format</label>
                  <input
                    type="text"
                    class="form-control is-invalid"
                    v-model="form.nameFmt"
                  />
                  <div class="invalid-feedback">
                    example: {{ nameFmtSample }}
                  </div>
                </div>
                <div class="col-md-12">
                  <label class="form-label">Phone number format</label>
                  <input
                    type="text"
                    class="form-control is-invalid"
                    v-model="form.phoneFmt"
                  />
                  <div class="invalid-feedback">
                    example: {{ phoneFmtSample }}
                  </div>
                </div>
              </div>
            </div>
          </div>
          <div class="col-sm-12 col-md-6">
            <div class="card">
              <div class="card-header">Generated contact samples</div>
              <div class="card-body">
                <ul class="list-group list-group-flush">
                  <li class="list-group-item" v-for="row in csvData.slice(1, 4)" >
                    <strong>Name: </strong>
                    <code>{{ formatStr(row, form.nameFmt) }}</code>
                    <br/>
                    <strong>Phone: </strong>
                    <code>{{ formatStr(row, form.phoneFmt) }}</code>
                  </li>
                </ul>
              </div>
            </div>
          </div>
        </div>
        <button class="btn btn-primary" @click="generateVcf()">Generate</button>
      </section>
    </div>
  </main>
</template>
<script>

const formatExpr = new RegExp('{(.*?)}', 'g');
export default {
  data() {
    return {
      csvStr: "",
      step: 1,
      form: {},
    };
  },

  methods:{
    formatStr(row, fmt){
      if(!fmt){
        return '';
      }
      const columnMapping = this.columnMapping;
      return fmt.replace( formatExpr, function(match, group1){
        return row[ columnMapping[group1] ];
      });
    },

    generateVcf(){

      const out = this.csvData.map(row=>{
        const name = this.formatStr(row, this.form.nameFmt);
        const phone = this.formatStr(row, this.form.phoneFmt);
        return`BEGIN:VCARD
VERSION:2.1
N:${name};;
FN:${name}
TEL;CELL:${phone}
END:VCARD`;
      }).join('\n');
      const blob = new Blob([out], {type: "text/vcard;charset=utf-8"});
      saveAs(blob, `generated-contacts-${Date.now()}.vcf`);
    }
  },

  computed: {
    nameFmtSample() {
      const allCols = this.availableCols;
      const nameCol = allCols.find((v) => v.match(/name/i)) || allCols[0];
      return `myprefix-{${nameCol}}`;
    },

    phoneFmtSample() {
      const allCols = this.availableCols;
      const nameCol = allCols.find((v) => v.match(/phone/i)) || allCols[0];
      return `+91-{${nameCol}}`;
    },

    csvData() {
      return this.csvStr.split("\n").map((v) => v.split("\t"));
    },

    availableCols() {
      return this.csvData[0];
    },

    columnMapping() {
      const out = {};
      const colHead = this.csvData[0];
      if (!colHead[0]) {
        return out;
      }
      for (let i in colHead) {
        out[colHead[i]] = parseInt(i);
      }
      return out;
    },
  },
};
</script>
<style></style>
