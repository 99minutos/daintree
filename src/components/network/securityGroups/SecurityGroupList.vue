<template>
  <div>
    <gl-drawer
      :open="drawerOpened && selectedResourceKey !== ''"
      @close="close"
      style="min-width: 80%;"
    >
      <template #header>{{ selectedResourceTitle }}</template>

      <SecurityGroup :securityGroup="selectedResource" v-on:deleted="close" />
    </gl-drawer>

    <div class="container-fluid">
      <div
        class="row justify-content-between mt-3 mb-2 ml-2 mr-2"
        v-if="resourcesAsList.length > 0"
      >
        <gl-form-input
          class="col-12 col-sm-8 col-lg-9 mb-3 mb-sm-0"
          id="filter"
          v-model="filter"
          placeholder="Type to filter..."
        />

        <gl-button
          icon="plus"
          category="secondary"
          variant="success"
          class="col-12 col-sm-3 col-lg-2"
          to="/network/securityGroups/new"
          >Create new security group
        </gl-button>
      </div>
      <gl-table
        :items="resourcesAsList"
        :fields="fields"
        :filter="filter"
        :busy="isLoading"
        ref="resourcesTable"
        :primary-key="resourceUniqueKey"
        selectable
        select-mode="single"
        @row-selected="onRowSelected"
        v-show="resourcesAsList.length > 0"
        show-empty
        hover
      >
        <template v-slot:emptyfiltered="">
          <gl-empty-state
            class="mt-5"
            title="No resource matching your search!"
            svg-path="/assets/undraw_file_searching_duff.svg"
            description="Remove the filter above to see all your resources"
            compact
          />
        </template>
        <template v-slot:cell(VpcId)="data">
          <router-link :to="`/network/vpcs?vpcId=${data.value}`">
            {{ data.value }}
          </router-link>
        </template>
        <template v-slot:cell(region)="data">
          <RegionText :region="data.value" />
        </template>
      </gl-table>

      <div class="container">
        <gl-skeleton-loading
          class="mt-5"
          v-if="isLoading && resourcesAsList.length < 1"
        />

        <gl-empty-state
          class="mt-5"
          v-if="!isLoading && resourcesAsList.length === 0"
          title="No security group found in the selected regions!"
          svg-path="/assets/undraw_empty_xct9.svg"
          :description="emptyStateDescription"
          compact
        >
          <template #actions>
            <gl-button
              icon="plus"
              variant="success"
              to="/network/securityGroups/new"
              >Create new security group
            </gl-button>
            <gl-button
              category="secondary"
              variant="success"
              class="ml-2"
              v-gl-modal-directive="'region-modal-id'"
              >Change selected regions
            </gl-button>
          </template>
        </gl-empty-state>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import {
  DescribeSecurityGroupsRequest,
  IpPermission,
  SecurityGroup as EC2SecurityGroup,
} from "aws-sdk/clients/ec2";
import RegionText from "@/components/common/RegionText.vue";
import {
  GlButton,
  GlDrawer,
  GlEmptyState,
  GlFormInput,
  GlIcon,
  GlModalDirective,
  GlSkeletonLoading,
  GlTable,
} from "@gitlab/ui";
import Component from "vue-class-component";
import StateText from "@/components/common/StateText.vue";
import SecurityGroup from "@/components/network/securityGroups/SecurityGroup.vue";
import { NetworkComponent } from "@/components/network/networkComponent";

@Component({
  components: {
    StateText,
    GlTable,
    RegionText,
    GlIcon,
    GlDrawer,
    SecurityGroup,
    GlButton,
    GlFormInput,
    GlSkeletonLoading,
    GlEmptyState,
  },
  directives: {
    "gl-modal-directive": GlModalDirective,
  },
})
export default class SecurityGroupList extends NetworkComponent<
  EC2SecurityGroup,
  "GroupId"
> {
  resourceName = "security group";
  canCreate = true;
  resourceUniqueKey: "GroupId" = "GroupId";

  fields = [
    {
      key: "GroupName",
      label: "Name",
      sortable: true,
    },
    { key: "GroupId", label: "Security group ID", sortable: true },
    { key: "region", sortable: true },
    { key: "VpcId", sortable: true },
    {
      key: "IpPermissions",
      label: "# inbound rules",
      formatter: (value: IpPermission[]) => value.length,
    },
    {
      key: "IpPermissionsEgress",
      label: "# outbound rules",
      formatter: (value: IpPermission[]) => value.length,
    },
  ];

  get selectedResourceTitle(): string {
    if (!this.selectedResource) {
      return "";
    }

    if (this.selectedResource.GroupName) {
      return `${this.selectedResource.GroupName} (${this.selectedResource?.GroupId})`;
    }

    return this.selectedResource.GroupId || "";
  }

  async getResourcesForRegion(
    region: string,
    filterByGroupId?: string[]
  ): Promise<EC2SecurityGroup[]> {
    const EC2 = await this.client(region);
    if (!EC2) {
      return [];
    }

    const params: DescribeSecurityGroupsRequest = {};

    if (filterByGroupId) {
      params.Filters = [{ Name: "group-id", Values: filterByGroupId }];
    }

    const data = await EC2.describeSecurityGroups(params).promise();
    if (data.SecurityGroups === undefined) {
      return [];
    }

    return data.SecurityGroups;
  }
}
</script>
