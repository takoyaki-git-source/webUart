<template>
  <v-card
    class="mx-auto"
    max-width="1000"
  >
    <v-toolbar
      color="teal"
      dark
    >
      <v-app-bar-nav-icon></v-app-bar-nav-icon>

      <v-toolbar-title>UART debug tool</v-toolbar-title>


      <v-spacer></v-spacer>

      <v-btn icon>
        <v-icon v-if="uartConnected" color="white" @click="disconnectUart()">mdi-broadcast</v-icon>
        <v-icon v-else color="grey darken-3" @click="connectUart()">mdi-broadcast-off</v-icon>
      </v-btn>
      <v-menu
        offset-y
      >
        <template v-slot:activator="{ on, attrs }">
          <v-btn
            icon
            color="white"
            v-bind="attrs"
            v-on="on"
          >
            <v-icon>mdi-dots-vertical</v-icon>
          </v-btn>
        </template>

        <v-list>
          <v-list-item @click="saveSettings()">
            <v-list-item-title>save</v-list-item-title>
          </v-list-item>
          <v-list-item @click="loadSettings()">
            <v-list-item-title>load</v-list-item-title>
          </v-list-item>
          <v-list-item @click="resetSettings()">
            <v-list-item-title>reset</v-list-item-title>
          </v-list-item>
        </v-list>
      </v-menu>

    </v-toolbar>

  <v-subheader>UART settings</v-subheader>
  <v-container fluid>
    <v-row align="center">
      <v-col
        class="d-flex"
        cols="12"
        sm="12"
      >
    <v-select v-model="baudrate" :items="baudrates" label="Baud rate" outlined></v-select>
    <v-select v-model="databit" :items="databits" label="Data bits" outlined></v-select>
    <v-select v-model="parity" :items="parities" label="Parity" outlined></v-select>
    <v-select v-model="stopbit" :items="stopbits" label="Stop bits" outlined></v-select>

      </v-col>
    </v-row>
  </v-container>


    <v-subheader>Command list</v-subheader>
    <v-list two-line v-for="item in items" :key="item.id" cols="12" flat>
      <v-list-item-group>
          <v-list-item>
              <v-list-item-action>
                  <v-checkbox :input-value=item.executeFlag @click="updateExecuteFlag(item.id)"></v-checkbox>
              </v-list-item-action>
              <!-- <v-list-item-content>
                  <v-list-item-title>{{item.id}}</v-list-item-title>
              </v-list-item-content> -->
              <v-list-item-content @click="executeCommand(item.command)">
                  <v-list-item-title>{{item.command}}</v-list-item-title>
                  <v-list-item-subtitle>{{item.overview}}</v-list-item-subtitle>
              </v-list-item-content>
              <v-list-item-action>
                <v-btn @click="deleteTask(item.id)" icon>               
                    <v-icon>mdi-trash-can</v-icon>
                </v-btn>
            </v-list-item-action>
          </v-list-item>
      </v-list-item-group>
    </v-list>
      <v-col
          cols="12"
          class="d-flex">
          <v-list-item-content>
            <v-text-field
            v-model="command"
            label="command"
            required
          ></v-text-field>
            <v-text-field
            v-model="overview"
            label="overview"
            required
          ></v-text-field>
              </v-list-item-content>

              <v-list-item-action>
              <v-btn @click.stop="addTask(command, overview)" icon>               
                  <v-icon>
                    mdi-plus-circle
                  </v-icon>
                </v-btn>
              </v-list-item-action>
        </v-col>
           
  <!-- log output area -->
  <v-container fluid>
    <v-row>
      <v-col
        cols="12"
        md="12"
      >
        <v-textarea
          outlined
          name="log-area"
          label="log output"
          hint="Hint text"
          v-model="uartLog"
        ></v-textarea>
      </v-col>
          </v-row>
  </v-container>

  </v-card>
</template>

<script>
  export default {
    data: () => ({
      items: [
        {
            id:1,
            overview:'read IG1 voltage',
            command:'t getIG1',
            executeFlag:true,
        },
        {
            id:2,
            overview:'turn on Lo beam',
            command:'t Lo 3000',
            executeFlag:true,
        },
      ],
      uartConnected: false,
      baudrates: [9600,14400,19220,28800,38400,57600,115200,230400,460800,921600],
      baudrate: 115200,
      databits: [7,8],
      databit: 8,
      parities: ['None', 'even', 'odd'],
      parity: 'None',
      stopbits: [1,2],
      stopbit: 1,
      port: '',
      uartLog: 'sample data.',
    }),
    mounted() {
      this.resetSettings()
      // this.readUart()
    },
    methods: {
      resetSettings() {
        this.resetUartSettings()
      },
      resetUartSettings() {
        this.baudrate = 115200
        this.databit = 8
        this.parity = "None"
        this.stopbit = 1
      },
      saveSettings() {
        console.log("saveSettings not supported yet.")
      },
      loadSettings() {
        console.log("loadSettings not supported yet.")
      },
      async disconnectUart() {
        await this.port.close()
        this.uartConnected = false
      },
      async connectUart() {
        try {
          this.port = await navigator.serial.requestPort()
          await this.port.open({baudRate:this.baudrate})
          this.uartConnected = true
          
          // read loop
          while (this.port.readable) {
            const reader = this.port.readable.getReader()
            try {
              for (;;) {
                const {value, done} = await reader.read()
                if (done) {
                  console.log("canceled")
                  break
                }
                // copy value to text area
                console.log(value)
                this.uartLog = value
              } 
            } catch (error) {
                // error handling
            } finally {
              reader.releaseLock()
            }
          }
        } catch (e) {
          console.log("UART connection error.")
        }
      },
      async readUart() {
        while (this.port.readable) {
          const reader = this.port.readable.getReader()
          try {
            for (;;) {
              const {value, done} = await reader.read()
              if (done) {
                console.log("canceled")
                break
              }
              // copy value to text area
              console.log(value)
            } 
          } catch (error) {
              // error handling
          } finally {
            reader.releaseLock()
          }
        }
      },
      executeCommand(command) {
        console.log(command)
      },
        updateExecuteFlag(id) {
            let index = this.items.findIndex((elem) => elem.id === id)
            this.items[index].executeFlag = !this.items[index].executeFlag
            console.log(this.items[index].executeFlag)
        },
        deleteTask(id) {
            this.items = this.items.filter((item) => item.id !==id)
        },
        addTask(command, overview) {
            this.items.push({id:3,overview:overview,command:command,executeFlag:false})
        }
    },
  }
</script>