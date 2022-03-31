<template>
  <q-page dark class="flex flex-center">
    <div class="q-pa-lg q-gutter-lg">
      <div class="row">
        <div class="col-xs-4 offset-xs-4">
          <q-img style="max-width: 20vw" src="logo.png"></q-img>
        </div>
      </div>

      <q-card class="bg-black text-white">
        <q-card-section>
          <div class="text-h6">Footswitch</div>
          <div class="text-subtitle2">Select your footswitch device</div>
        </q-card-section>

        <q-card-section>
          Check what midi device is the proper one for you. Check console logs
          for input checks.
        </q-card-section>

        <q-select
          ref="footswitchRef"
          v-model="footswitch"
          :options="footswitchOptions"
          label="Footswitch"
          filled
          text-white
          emit-value
          option-value="id"
          option-label="name"
          color="light-blue-12"
          map-options
          dark
        >
          <template v-slot:no-option>
            <q-item>
              <q-item-section class="text-grey">
                No MIDI device found
              </q-item-section>
            </q-item>
          </template>
        </q-select>
      </q-card>
      <q-card class="bg-black text-white">
        <q-card-section>
          <div class="text-h6">Amp</div>
          <div class="text-subtitle2">Select your amp device</div>
        </q-card-section>

        <q-card-section>
          Choose KATANA for controlling your amp.
        </q-card-section>

        <q-select
          ref="receiverRef"
          v-model="receiver"
          :options="receiverOptions"
          label="Amp"
          filled
          text-white
          emit-value
          option-value="name"
          option-label="name"
          color="pink-12"
          map-options
          dark
        >
          <template v-slot:no-option>
            <q-item>
              <q-item-section class="text-grey">
                No MIDI device found
              </q-item-section>
            </q-item>
          </template>
        </q-select>
      </q-card>
    </div>
  </q-page>
</template>

<script>
import { defineComponent, ref } from "vue";
import { WebMidi } from "webmidi";

const CHANNELS = [15, 16, 17, 18, 19, 40, 50, 51, 52, 53];

export default defineComponent({
  name: "KatanaFooter",
  setup() {
    return {
      lorem:
        "Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.",
      footswitch: ref(null),
      midiFootswitch: ref(null),
      receiver: ref(null),
      midiReceiver: ref(null),
      enabled: ref(false),
      footswitchOptions: ref([]),
      receiverOptions: ref([]),
    };
  },
  beforeMount() {
    WebMidi.enable({ sysex: true });
    if (navigator.requestMIDIAccess) {
      navigator.requestMIDIAccess({ sysex: true }).then(
        (access) => {
          this.enabled = true;
          console.log("MIDI enabled.");
          access.inputs.forEach((input) => {
            this.footswitchOptions.push(input);
          });
          access.outputs.forEach((output) => {
            this.receiverOptions.push(output);
          });
          this.$refs.footswitchRef.refresh(1);
          this.$refs.receiverRef.refresh(1);
        },
        () => {
          new Error("MIDI access failed");
        }
      );
    } else {
      new Error("No MIDI access in this browser");
    }
  },
  methods: {
    parseMessage(msg) {
      if (msg && msg.data) {
        return {
          channel: msg.data[0],
          command: msg.data[1],
          value: msg.data[2],
        };
      }
    },
    sendMessage(msg) {
      let parsed_msg = this.parseMessage(msg);
      let command = parsed_msg.command - 1;
      let typeOf = command >= 39 ? "PC" : "CC";

      if (typeOf == "PC") {
        this.midiReceiver.sendProgramChange(command);
        this.clearLeds(parsed_msg.command);
      } else {
        this.midiReceiver.sendControlChange(command, parsed_msg.value);
      }
      console.log("SENT MESSAGE TO KATANA:" + command + " " + parsed_msg.value);
    },
    clearLeds(actual) {
      CHANNELS.forEach((input) => {
        if (input != actual) {
          this.midiReceiver.sendControlChange(input, 0);
        }
      });
      console.log("CLEAR LEDS ON FOOTSWITCH");
    },
  },
  watch: {
    footswitch(newVal, oldVal) {
      console.log("New: " + newVal + " old: " + oldVal);
      this.midiFootswitch = this.footswitchOptions.find(
        (item) => item.id == newVal
      );
      console.log(this.midiFootswitch);
      this.midiFootswitch.onmidimessage = (msg) => {
        console.log("Message Received from " + newVal);
        console.log(msg);
        this.sendMessage(msg);
      };
    },
    receiver(newVal, oldVal) {
      console.log("New: " + newVal + " old: " + oldVal);
      this.midiReceiver = WebMidi.getOutputByName(newVal);

      // let test = "F0 41 00 00 00 00 33 11 60 00 00 10 00 00 00 02 0E F7";

      // console.log(
      //   sender.send([
      //     0xf0, 0x41, 0x00, 0x00, 0x00, 0x00, 0x33, 0x12, 0x60, 0x00, 0x00,
      //     0x10, 0x00, 0x00, 0x00, 0x02, 0x0e, 0xf7,
      //   ])
      // );
    },
  },
});
</script>
