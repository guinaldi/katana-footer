<template>
  <q-page class="flex flex-center">
    <div class="q-pa-md row items q-gutter-md">
      <q-card class="bg-black text-white">
        <q-card-section>
          <div class="text-h6">Footswitch</div>
          <div class="text-subtitle2">Select your footswitch device</div>
        </q-card-section>

        <q-card-section>
          {{ lorem }}
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
          color="lime-11"
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
          <div class="text-h6">Device</div>
          <div class="text-subtitle2">Select your receiver device</div>
        </q-card-section>

        <q-card-section>
          {{ lorem }}
        </q-card-section>

        <q-select
          ref="receiverRef"
          v-model="receiver"
          :options="receiverOptions"
          label="Receiver"
          filled
          text-white
          emit-value
          option-value="id"
          option-label="name"
          color="lime-11"
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
        console.log(this.parseMessage(msg));
      },
    };
  },
  beforeMount() {
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

    this.$watch("footswitch", (newVal, oldVal) => {
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
    });
  },
});
</script>
