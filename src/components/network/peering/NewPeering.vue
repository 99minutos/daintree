<template>
  <div>
    <Header hide-refresher />
    <div class="container mt-2">
      <h2>Create a new Peering</h2>
      <gl-alert variant="tip" class="mb-2 mt-2" :dismissible="false">
        A Peering is an isolated portion of the AWS cloud populated by AWS
        objects, such as Amazon EC2 instances. You must specify an IPv4 address
        range for your Peering. Specify the IPv4 address range as a Classless
        Inter-Domain Routing (CIDR) block; for example, 10.0.0.0/16.<br />

        The smallest Peering you can create uses a /28 netmask (16 IPv4
        addresses), and the largest uses a /16 netmask (65,536 IPv4 addresses)
      </gl-alert>
      <gl-form-group
        id="region-id"
        label="Region"
        label-size="sm"
        description="To see other regions, enable them in the settings"
        label-for="region-input"
      >
        <gl-form-select
          id="region-input"
          v-model="selectedRegion"
          :options="this.$store.getters['sts/regions']"
        />
      </gl-form-group>
      <gl-form-input-group
        class="mt-3"
        v-model="peeringName"
        placeholder="Create a tag with key 'Name' and the value you insert."
      >
        <template #prepend>
          <b-input-group-text>Name</b-input-group-text>
        </template>
      </gl-form-input-group>
      <gl-form-input-group
        required
        v-model="cidrBlock"
        class="mt-3"
        placeholder="The range of IPv4 addresses for your Peering in CIDR block format."
      >
        <template #prepend>
          <b-input-group-text>IPv4 CIDR block</b-input-group-text>
        </template>
      </gl-form-input-group>
      <div class="row justify-content-between mt-3">
        <gl-button category="secondary" variant="danger" to="/network/peerings">
          Cancel
        </gl-button>
        <gl-button
          class="float-right"
          category="primary"
          variant="success"
          @click="createPeering"
          >Create Peering
        </gl-button>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import Header from "@/components/Header/Header.vue";
import {
  GlAlert,
  GlFormGroup,
  GlFormInputGroup,
  GlFormSelect,
  GlButton,
} from "@gitlab/ui";
import { BInputGroupText } from "bootstrap-vue";
import EC2Client from "aws-sdk/clients/ec2";
import { Component } from "vue-property-decorator";
import Notifications from "@/mixins/notifications";

@Component({
  components: {
    Header,
    GlFormSelect,
    GlFormGroup,
    GlAlert,
    GlFormInputGroup,
    BInputGroupText,
    GlButton,
  },
})
export default class NewPeering extends Notifications {
  selectedRegion = "";
  cidrBlock = "";
  peeringName = "";

  get credentials() {
    return this.$store.getters["sts/credentials"];
  }

  createPeering() {
    const EC2 = new EC2Client({
      region: this.selectedRegion,
      credentials: this.credentials,
    });
    EC2.createPeering({ CidrBlock: this.cidrBlock }, (err, data) => {
      if (err) {
        this.showError(err.message, "createPeering");
      } else {
        this.hideErrors("createPeering");

        //We cannot set tags with `createPeering`
        if (this.peeringName && data.Peering && data.Peering.PeeringId) {
          const params = {
            Resources: [data.Peering.PeeringId],
            Tags: [{ Key: "Name", Value: this.peeringName }],
          };
          EC2.createTags(params, (err) => {
            if (err) {
              this.showError(err.message, "createPeering");
            } else {
              this.$router.push("/network/peerings");
            }
          });
        } else {
          this.$router.push("/network/peerings");
        }
      }
    });
  }
}
</script>

<style scoped></style>
