
<template>
  <div class="tile is-parent">
    <div class="tile is-child box">
      <div v-if="test.ID">
        <h1 class="title">{{ test.Name }} <a class="button" @click="isEditTestTitleModalActive = true;">
                <span class="icon is-small">
                    <font-awesome-icon icon="edit"/>
                </span>
                </a></h1>
        <p class="subtitle is-6">
          <a class="button is-small is-rounded is-link" @click="duplicateTest(test.ID);">duplicate</a>
          <a v-if="canDeleteTest" class="button is-small is-rounded is-danger" @click="isDeleteTestModalActive = true;">delete</a>
          <a v-if="test.Status=='Deploying'" class="button is-small is-rounded is-danger" @click="cancelTest(test.ID);">cancel</a>
          <a v-if="canStopTest" class="button is-small is-rounded is-danger" @click="isStopTestModalActive = true;">stop</a>
        </p>
        <div class="block">
            <p class="header is-6">Status: {{ test.Status }} (<a @click="showLogs(true);">show logs</a>)</p>
            <p class="header is-6">Created: {{ test.Created | moment("lll") }}</p>
        </div>
        <div class="block">
          <button :disabled=isLaunchDisabled class="button is-link" @click="isDeployTestModalActive = true; getGrids();">Launch</button>
            <form v-if="testIP.Status=='Success'" method="post" v-bind:action="'https://' + testIP.IP  + '/login'" target="_blank">
              <input type="hidden" id="token" name="authToken" v-bind:value="testIP.Auth">
              <button type="submit" id="locust" class="button is-primary">Master Locust</button>
            </form>
        </div>
        <div class="testDetails">
            <div class="field is-grouped is-grouped-multiline">
            <p class="title is-4">Labels: &#8203;</p>
            <div class="field">
                <div class="control">
                <input class="input is-small" v-model="label" v-on:keyup.enter="addLabel" type="text" placeholder="Add label">
                </div>
            </div>
            <p class="title is-4"> &#8203; &#8203; </p>
            <div v-for="label in test.Labels" :key="label" class="control">
              <span class="tag">
                  {{ label }}
                <button class="delete is-small" v-on:click="deleteLabel(label)"></button>
              </span>
            </div>
            </div>

            <div class="block">
                <a v-if="test.SnapshotURL != ''" v-bind:href="test.SnapshotURL" target="_blank" class="button is-link">Grafana Snapshot</a>
                <a v-else-if="grafanaConfig.Enabled == true" v-bind:href="grafanaConfig.BaseURL + '/d/' + grafanaConfig.DashboardUID" target="_blank" class="button is-link">Grafana</a>
            </div>

            <div class="level">
            <div class="level-left">
                <p class="title is-4">Description <a class="button" @click="isEditTestDescModalActive = true;">
                <span class="icon is-small">
                    <font-awesome-icon icon="edit"/>
                </span>
                </a></p>
            </div>
            </div>
            <div class="block">
            <p>{{ test.Desc }}</p>
            </div>
            <div class="block">
            <div class="level-left">
                <p class="title is-4">Locust Config <a class="button">
                <span class="icon is-small">
                    <font-awesome-icon icon="edit"/>
                </span>
                </a></p>
            </div>
            <div class="configWrap">
                <table class="table">
                <tr>
                    <th>Parameter</th>
                    <th>Value</th>
                </tr>
                <tr v-for="config in testConfig" :key="config">
                    <td>{{ config.Name }}</td>
                    <td>{{ config.Value }}</td>
                </tr>
                </table> 
            </div>
            <div class="block">
                <p class="title is-4">Test Files <a class="button" v-bind:href="'/api/test/' + testID + '/files/download'"><span class="icon is-small">
                        <font-awesome-icon icon="download"/>
                    </span></a></p>
                <div class="select is-multiple">
                <select multiple size="5">
                    <option v-for="(file, index) in testfilesNoFolders" :key="file" v-bind:value="index">
                        {{ file.Filename }}
                    </option>
                </select>
                </div>
            </div>


            <div class="block">
                <p class="title is-4">Attachments</p> 

                <div class="field is-grouped">
                  <button class="button" :disabled="selectedAttachment==undefined" @click="deleteAttachment(testID, selectedAttachment)">
                    <span class="icon is-small">
                        <font-awesome-icon icon="trash"/>
                    </span>
                  </button>

                  <div class="file">                                        
                    <label class="file-label">
                      <input class="file-input" type="file" name="testfile" id="attachmentid" accept="*" v-on:change="handleFileUpload()">
                      <span class="file-cta">
                        <span class="icon is-small">
                          <font-awesome-icon icon="upload"/>
                        </span>
                      </span>
                    </label>
                  </div>

                  <a v-if="selectedAttachment==undefined" class="button" disabled>
                    <span class="icon is-small">
                        <font-awesome-icon icon="download"/>
                    </span>
                  </a>
                  <a v-else class="button" v-bind:href="'/api/test/' + testID + '/attachment/' + selectedAttachment">
                    <span class="icon is-small">
                        <font-awesome-icon icon="download"/>
                    </span>
                  </a>
                 
                </div>
                
                <div class="select">
                  <select v-model="selectedAttachment">
                      <option :value="undefined" disabled style="display:none">Select Attachment</option>
                      <option v-for="attachment in testAttachments" :key="attachment.ID" v-bind:value="attachment.ID">
                          {{ attachment.Filename }}
                      </option>
                  </select>
                </div>
            </div>


            <div class="block">
                <p class="title is-4">Change Result</p> 

                <div class="field has-addons">

                  <div class="control">
                    <span class="select">
                      <select v-model="selectedResult">
                          <option :value="undefined" disabled style="display:none">Select Result</option>
                          <option v-for="result in testResults" :key="result.ID" v-bind:value="result.Name">
                              {{ result.Name }} 
                          </option>
                      </select>
                    </span>
                  </div>
                  <div class="control">
                    <a class="button is-rounded is-link" @click="changeResult(test.ID, selectedResult);">Change</a>
                  </div>
                </div>
            </div>
                


            </div>  
            </div>

        
        <div class="modal" v-bind:class="{ 'is-active': isShowLogsModalActive}">
          <div class="modal-background"></div>
          <div class="modal-card">
            <header class="modal-card-head">
              <p class="modal-card-title">Logs for {{test.Name}}</p>
              <button class="delete" aria-label="close" @click="showLogs(false);"></button>
            </header>
            <section class="modal-card-body">
              <div class="content">
                <h5>Currently Deploying: {{logStatus}}</h5>
              </div>
              <p v-for="log in logs" :key="log">{{ logPrint(log) }}</p> 
            </section>
            <footer class="modal-card-foot">
              <button class="button" @click="showLogs(false);">Close</button>
            </footer>
          </div>
        </div>
        



        <div class="modal" v-bind:class="{ 'is-active': isDeployTestModalActive }">
            <div class="modal-background"></div>
            <div class="modal-content">
            <div class="box">
             <div class="field">
              <label class="label">Grid</label>
              <div class="control">
                <div class="select">
                  <select v-model="grid" @change="validateTest();">
                    <option :value="undefined" disabled style="display:none">Select grid for test</option>
                    <option  v-for="_grid in listOfGrids" :key="_grid.ID" :value="_grid">{{_grid.Name}}</option>
                  </select>
                </div>
              </div>
            </div>
                <button v-bind:disabled="!testDeployReady" class="button is-success" @click="deployTest();">Deploy</button>
                <button class="button" @click="clearDeployModal();">Cancel</button>
            </div>
            </div>
        </div>


        <div class="modal" v-bind:class="{ 'is-active': isDeleteTestModalActive }">
            <div class="modal-background"></div>
            <div class="modal-content">
            <div class="box">
                <p>Are you sure you want to delete {{ test.Name }}?</p>
                <button class="button is-danger" @click="isDeleteTestModalActive = false; deleteTest(test.ID);">Delete</button>
                <button class="button" @click="isDeleteTestModalActive = false;">Cancel</button>
            </div>
            </div>
        </div>

        <div class="modal" v-bind:class="{ 'is-active': isStopTestModalActive }">
            <div class="modal-background"></div>
            <div class="modal-content">
            <div class="box">
                <p>Are you sure you want to stop {{ test.Name }}?</p>
                <button class="button is-danger" @click="isStopTestModalActive = false; stopTest(test.ID);">Stop</button>
                <button class="button" @click="isStopTestModalActive = false;">Cancel</button>
            </div>
            </div>
        </div>

        <div class="modal" v-bind:class="{ 'is-active': isEditTestTitleModalActive }">
            <div class="modal-background"></div>
            <div class="modal-content">
            <div class="box">
              <h1 class="title">Edit the test name for {{ test.Name }}</h1>
              <div class="field">
                <label class="label">Title</label>
                <div class="control">
                  <input class="input" type="text" v-model="newTestTitle" placeholder="Add new title here" v-bind:value="test.Title">
                </div>
              </div>
                <div class="field is-grouped">
                  <div class="control">
                    <button class="button is-danger" @click="isEditTestTitleModalActive = false; editTestTitle(testID, newTestTitle);">Edit</button>
                  </div>
                  <div class="control">
                    <button class="button" @click="isEditTestTitleModalActive = false; newTestTitle='';">Cancel</button>
                  </div>
                </div>
            </div>
            </div>
        </div>

        <div class="modal" v-bind:class="{ 'is-active': isEditTestDescModalActive }">
            <div class="modal-background"></div>
            <div class="modal-content">
            <div class="box">
              <h1 class="title">Edit the test description for {{ test.Name }}</h1>
              <div class="field">
                <label class="label">Description</label>
                <div class="control">
                  <textarea class="textarea" v-model="testEdit.Desc" placeholder="Enter a description."></textarea>
                </div>
              </div>
                <div class="field is-grouped">
                  <div class="control">
                    <button class="button is-danger" @click="isEditTestDescModalActive = false; editTestDesc(testID, testEdit.Desc);">Edit</button>
                  </div>
                  <div class="control">
                    <button class="button" @click="isEditTestDescModalActive = false; testEdit.Desc=test.Desc;">Cancel</button>
                  </div>
                </div>
            </div>
            </div>
        </div>

        </div>

  


        <div v-else>
          <h1 class="title">No tests selected</h1>
        </div>
      </div>
    </div>
</template>  

<script>
import axios from 'axios';

export default {
  name: 'TestDetails',
  props: {
    testID: String,
    currentTestStatus: String
  },
  computed: {
    testfilesNoFolders: function () {
      return this.testfiles.filter(file => file.Filesize > 0)
    },
    isLaunchDisabled: function () {
      if (this.test != null && this.test.Status == 'Ready' || this.test.Status == 'Expired') {
        return false
      }
      return true
    },
    canDeleteTest: function () {
      switch (this.test.Status) {
        case 'Deploying':
        case 'Deployed':
        case 'Running':
        case 'Launching':
        case 'Launched':
          return false
        default:
          return true
      }
    },
    canStopTest: function () {
      switch (this.test.Status) {
        case 'Deployed':
        case 'Running':
        case 'Launched':
          return true
        default:
          return false
      }
    }
  },
  beforeMount() {
    this.getGrafanaConfigs()
    this.currentTest = this.testID
    this.loadTestData(this.testID)
    if (this.$route.name == 'testlogs') {
      this.showLogs(true)
    }
  },
  watch: {
    testID: function (newVal) {
      this.testIP.Status = ""
      this.testIP.IP = ""
      this.testIP.Auth = ""
      this.testIP.Description = ""
      this.loadTestData(newVal)
    },
    currentTestStatus: function() {
      // adding if statement to prevent duplicate API calls on changes.
      if (this.currentTest == this.testID) {
        this.loadTestData(this.testID)
      } else {
        this.currentTest = this.testID
      }
    },
  },
  data: function () {
    return {
      test: {Status: null, Desc: ""},
      testEdit: {Status: null, Desc: ""},
      grafanaConfig: {Enabled: false, BaseURL: "", DashboardUID: ""},
      testfiles: [],
      testConfig: [
        {Name: "test-config-file", Value: "testconfig.txt"}, 
        {Name: "clients", Value: "100"},
        {Name: "hatch-rate", Value: "10"},
        {Name: "run-time", Value: "3600"},
        {Name: "loglevel", Value: "ERROR"},
        {Name: "load-profile", Value: "[(0,0), (10m,100%), (+15m,0%)]"}
      ],
      isEditTestTitleModalActive: false,
      isEditTestDescModalActive: false,
      selectedResult: undefined,
      testResults: [
        {ID: 1, Name: "" },
        {ID: 2, Name: "Pass" },
        {ID: 3, Name: "Partial" },
        {ID: 4, Name: "Fail"}
      ],
      testAttachments: [],
      uploadPercentage: 0,
      selectedAttachment: undefined,
      isDeleteTestModalActive: false,
      isStopTestModalActive: false,
      isShowLogsModalActive: false,
      isDeployTestModalActive: false,
      gettingLogs: false,
      logStatus: '',
      logs: '',
      listOfGrids: [],
      grid: {},
      region: "us-east-1",
      testDeployReady: false,
      startAutomatically: false,
      testIP: {Status: "",
               IP: "",
               Auth: "",
               Description: ""
              },
      locust_url: null,
      locust_token: null,
      currentTest: null,
      label: "",
    }
  },
  methods: {
    getGrafanaConfigs: function() {
      fetch('/api/grafana/info')
        .then(response => {
          if (response.status === 200) {
             response.json().then(data => {
               this.grafanaConfig = data;
            });
            
          }
        })
    },
    editTestTitle: function (testID, title) {
      fetch('/api/test/' + testID + '/edit', {
        method: 'POST',
        body: new URLSearchParams({Title: title}),
        headers: new Headers({'Content-Type': 'application/x-www-form-urlencoded'})
      }).then(response => {
          this.loadTestData(testID)
          this.$emit('get-tests', "")
        })
        
    },
    editTestDesc: function (testID, desc) {
      fetch('/api/test/' + testID + '/edit', {
        method: 'POST',
        body: new URLSearchParams({Desc: desc}),
        headers: new Headers({'Content-Type': 'application/x-www-form-urlencoded'})
      }).then(response => {
          this.loadTestData(testID)
          this.$emit('get-tests', "")
        })
        
    },
    changeResult: function (testID, result) {
      fetch('/api/test/' + testID + '/edit', {
        method: 'POST',
        body: new URLSearchParams({Result: result}),
        headers: new Headers({'Content-Type': 'application/x-www-form-urlencoded'})
      }).then(response => {
          this.loadTestData(testID)
          this.$emit('get-tests', "")
          this.selectedResult = undefined
        })
    },
    handleFileUpload: function() {
      this.uploadPercentage = 0;
      this.uploadAttachment(document.getElementById('attachmentid').files[0]);
    },
    loadTestData: function (testid) {
      this.testAttachments = []
      this.selectedAttachment = undefined
      if (testid) {
        axios
          .get('/api/test?id=' + testid)
          .then(response => {
            this.test = response.data
            this.testEdit = JSON.parse(JSON.stringify(this.test))
            this.getTestLink(testid)
            })
          .catch(error => {console.log("FAILURE: ", error)}); 
        axios
          .get('/api/test/' + testid + '/files')
          .then(response => (this.testfiles = response.data))
          .catch(error => {console.log("FAILURE: ", error)});
        
        this.getTestAttachments(testid)
        
      } else {
        this.test = {Status: null}
        this.testfiles = []
      }    
    },
    deleteTest: function(id) {
      console.log("Delete id: ", id)
      axios
      .post('/api/test/' + id + '/delete')
      .then(response => this.$emit('get-tests', ""))
      .then(this.$router.push('/tests'))
    },
    stopTest: function(id) {
      console.log("Delete id: ", id)
      axios
      .post('/api/test/' + id + '/stop')
      .then(response => this.$emit('get-tests', ""))
      .then(this.$router.push('/tests'))
    },
    deleteAttachment: function(testID, attachmentID) {
      axios
      .post('/api/test/' + testID + '/attachment/' + attachmentID + '/delete')
      .then(response => {
        this.selectedAttachment = undefined
        this.getTestAttachments(testID)})
    },
    cancelTest: function(id) {
      axios
        .post('/api/test/' + id + '/cancel')
        .then(response => {
          this.$emit('get-tests', "")
          this.loadTestData(id)
        } )
    },
    duplicateTest: function(id) {
      axios
        .post('/api/test/' + id + '/duplicate')
        .then(response => {
          this.$emit('get-tests', response.data.TestID)
        } )
    },
    showLogs: function(bool) {
      if (bool == true) {
        this.getLogs(this.testID)
        this.$router.push('/tests/' + this.testID + '/logs')
        this.isShowLogsModalActive = true
      } else {
        this.$router.push('/tests/' + this.testID)
        this.stopGettingLogs()
        this.isShowLogsModalActive = false
      }
      
    },
    logPrint: function (log) {
      var logprint = ""
      if (log.Output != "") {
        logprint = this.$moment(log.Timestamp).format("MMM D, YYYY h:mm:ssA") + ": " + log.Output
      }
      return logprint
    },
    getLog: function(id){
      if(this.logs.length > 0 && this.logs[this.logs.length-1].Running == true) {
      axios
        .get("/api/test/" + id + "/deploylogs")
        .then(response => (this.logs = response.data))
        .then(this.logStatus = this.logs[this.logs.length-1].Running) 
      }else{
        clearInterval(this.gettingLogs)
        this.logStatus = false
      }
    },
    getLogs: function(id){
      axios
        .get("/api/test/" + id + "/deploylogs")
        .then(response => {
          this.logs = response.data
          this.logStatus = response.data[response.data.length-1].Running
          this.gettingLogs = setInterval(() => {this.getLog(id)}, 3000)
        })
    },
    stopGettingLogs: function() {
      this.$router.push('/tests/' + this.testID)
      this.logs = ''
      clearInterval(this.gettingLogs)
      this.logStatus = ''
    },
    getGrids: function () {
      axios
        .get('/api/grids?status=Available')
        .then(response => (this.listOfGrids = response.data))
    },
    validateTest: function () {
      if (this.gridID != '') {
        this.testDeployReady = true
      } else {
        this.testDeployReady = false
      }
    },
    deployTest: function () {
      var path, body
      path = "/api/test/" + this.test.ID + "/start"
      body = {GridID: this.grid.ID, GridRegion: this.grid.Region, StartAutomatically: this.startAutomatically }
      axios
        .post(path, body)
        .then(response => {
          this.isDeployTestModalActive=false
          this.loadTestData(this.test.ID)
          this.$emit('get-tests', "")
        })
    },
    clearDeployModal: function () {
      this.grid = {}
      this.isDeployTestModalActive=false
    },
    getTestLink: function (testID) {
      var getLink = false
      switch (this.test.Status) {
        case 'Deployed':
        case 'Running':
        case 'Launching':
        case 'Launched':
          getLink = true;
          break;
        default:
          getLink = false;
      }

      if (getLink) {
        axios
          .get('/api/test/' + testID + '/ip')
          .then(response => (this.testIP = response.data))
      } else {
        this.testIP = {Status: "", IP: "", Auth: "", Description: ""}
      }
    },
    getTestAttachments: function (testID) {
      axios
        .get('/api/test/' + testID + '/attachments')
        .then(response => {
          if (response.data == null) {
            this.testAttachments = []
          } else {
            console.log("Adding attachment filenames to list.")
            this.testAttachments = response.data
          }
        })
    },
    uploadAttachment: function(file) {
      var formData
      formData = new FormData();
      formData.append("file", file);
      axios
        .post('/api/test/' + this.testID + "/attachment", formData, {
          headers: {'Content-Type': 'multipart/form-data'},
          validateStatus: function(status) {
            return true;
          },
          onUploadProgress: function( progressEvent ) {
            this.uploadPercentage = parseInt( Math.round( ( progressEvent.loaded * 100 ) / progressEvent.total ) );
          }.bind(this)
        })
        .then(response => {
          this.response = response.data
          if (response.data.Status == 'Success') {
            this.getTestAttachments(this.testID)
            return
          }
        })
        .catch(error => {console.log("FAILURE: ", error)});
    },
    addLabel: function() {
      if (this.label != "") {
        axios
          .post('/api/test/' + this.testID + '/label/' + this.label)
          .then(response => {
            this.$emit('get-tests', "")
            this.loadTestData(this.testID)
          })
      }
      this.label = ""
    },
     deleteLabel: function(label) {
        axios
          .delete('/api/test/' + this.testID + '/label/' + label)
          .then(response => {
            this.$emit('get-tests', "")
            this.loadTestData(this.testID)
          })
    },
  }
}
</script>
