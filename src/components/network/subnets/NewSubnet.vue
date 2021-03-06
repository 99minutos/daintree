<template>
  <div class="container mt-2">
    <h2>Create a new subnet</h2>
    <gl-alert variant="tip" class="mb-2 mt-2" :dismissible="false">
      Specify your subnet's IP address block in CIDR format; for example,
      10.0.0.0/24. IPv4 block sizes must be between a /16 netmask and /28
      netmask, and can be the same size as your VPC. An IPv6 CIDR block must be
      a /64 CIDR block.
    </gl-alert>

    <RegionDropdown v-model="selectedRegion" />

    <gl-form-input-group
      class="mt-3"
      v-model="subnetName"
      placeholder="Create a tag with key 'Name' and the value you insert."
    >
      <template #prepend>
        <b-input-group-text>Name</b-input-group-text>
      </template>
    </gl-form-input-group>

    <VpcDropdown
      v-model="selectedVpc"
      :region="selectedRegion"
      description="The VPC in which the subnet will be created."
    />

    <gl-form-group
      id="az-id"
      label="Availability zone"
      label-size="sm"
      description='The availability zone where this subnet will reside. Select "No preference" to let Amazon choose an Availability Zone for you'
      label-for="az-input"
      class="mt-2"
    >
      <gl-form-select
        id="az-input"
        :disabled="selectedRegion === '' || isLoading"
        v-model="selectedAz"
        :options="azOptions"
      />
    </gl-form-group>

    <gl-form-input-group
      required
      v-model="cidrBlock"
      class="mt-3"
      placeholder="The range of IPv4 addresses for your subnet in CIDR block format."
    >
      <template #prepend>
        <b-input-group-text>IPv4 CIDR block</b-input-group-text>
      </template>
    </gl-form-input-group>
    <div class="row justify-content-between mt-3">
      <gl-button category="secondary" variant="danger" to="/network/subnets">
        Cancel
      </gl-button>
      <gl-button
        class="float-right"
        category="primary"
        variant="success"
        :disabled="!canClick"
        @click="createSubnet"
        >Create subnet
      </gl-button>
    </div>
  </div>
</template>

<script lang="ts">
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
import {
  AvailabilityZoneList,
  CreateSubnetRequest,
  VpcList,
} from "aws-sdk/clients/ec2";
import { mixins } from "vue-class-component";
import { Formatters } from "@/mixins/formatters";
import { DaintreeComponent } from "@/mixins/DaintreeComponent";
import RegionDropdown from "@/components/common/formComponents/RegionDropdown.vue";
import VpcDropdown from "@/components/common/formComponents/VpcDropdown.vue";

@Component({
  components: {
    GlFormSelect,
    GlFormGroup,
    GlAlert,
    GlFormInputGroup,
    BInputGroupText,
    GlButton,
    RegionDropdown,
    VpcDropdown,
  },
})
export default class NewSubnet extends mixins(DaintreeComponent, Formatters) {
  selectedRegion = "";
  selectedVpc = "";
  selectedAz = "No preference";

  cidrBlock = "";
  subnetName = "";

  az: AvailabilityZoneList = [];
  loadingCount = 0;

  get canClick(): boolean {
    return this.selectedRegion !== "" && this.selectedVpc !== "";
  }

  get azOptions(): string[] {
    const options: string[] = [];
    this.az.forEach((az) => {
      if (az.ZoneName) options.push(az.ZoneName);
    });

    options.push("No preference");
    return options;
  }

  async EC2(): Promise<EC2Client> {
    const credentials = await this.credentials();
    return new EC2Client({
      region: this.selectedRegion,
      credentials,
    });
  }

  async createSubnet(): Promise<void> {
    const params: CreateSubnetRequest = {
      CidrBlock: this.cidrBlock,
      VpcId: this.selectedVpc,
    };

    if (this.selectedAz !== "No preference") {
      params.AvailabilityZone = this.selectedAz;
    }
    const client = await this.EC2();
    if (!client) {
      return;
    }
    try {
      const data = await client.createSubnet(params).promise();

      if (this.subnetName && data.Subnet && data.Subnet.SubnetId) {
        const params = {
          Resources: [data.Subnet.SubnetId],
          Tags: [{ Key: "Name", Value: this.subnetName }],
        };

        await client.createTags(params).promise();
      }

      this.$router.push(`/network/subnets?subnetId=${data.Subnet?.SubnetId}`);
    } catch (err) {
      this.showError(err.message, "createSubnet", this.selectedRegion);
    }
  }

  async getAzForCurrentRegion() {
    this.hideErrors("getAz");
    if (this.selectedRegion === "") {
      this.az = [];
    } else {
      this.incrementLoadingCount();
      const client = await this.EC2();
      client.describeAvailabilityZones(
        { Filters: [{ Name: "state", Values: ["available"] }] },
        (err, data) => {
          this.decreaseLoadingCount();
          if (err) {
            this.showError(err.message, "getAz");
          } else if (data.AvailabilityZones) {
            this.az = data.AvailabilityZones;
          }
        }
      );
    }
  }

  mounted(): void {
    this.$root.$on("refresh", this.regionChanged);
  }

  beforeDestroy(): void {
    this.$root.$off("refresh");
  }

  regionChanged(): void {
    this.getAzForCurrentRegion();
  }
}
</script>
