<template>
  <div>
    <NetworkMain />
    <EC2Main />
    <MessagesMain />
    <ECSMain />
  </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import NetworkMain from "./network/NetworkMain.vue";
import Header from "@/components/Header/Header.vue";
import EC2Main from "@/components/EC2/EC2Main.vue";
import MessagesMain from "@/components/messages/MessagesMain.vue";
import ECSMain from "@/components/ECS/ECSMain.vue";

@Component({
  components: { ECSMain, MessagesMain, NetworkMain, Header, EC2Main },
})
export default class MainMenu extends Vue {}
</script>
