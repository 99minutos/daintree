<template>
  <div>
    {{ tasksRevisions }}
  </div>
</template>
<script lang="ts">
import { DaintreeComponent } from "../../../mixins/DaintreeComponent";
import { Component, Prop } from "vue-property-decorator";
import ECS from "aws-sdk/clients/ecs";

@Component
export default class TasksDefinitionsForFamily extends DaintreeComponent {
  @Prop(String) readonly region!: string;
  @Prop(String) readonly family!: string;
  errorMessage = "";

  tasksRevisions: string[] = [];

  async retrieveData(): Promise<void> {
    const credentials = await this.credentials();
    if (!credentials) {
      return;
    }

    const client = new ECS({ region: this.region, credentials });
    try {
      const data = await client
        .listTaskDefinitions({ familyPrefix: this.family })
        .promise();
      this.tasksRevisions = data.taskDefinitionArns || [];
    } catch (err) {
      this.errorMessage = err;
    }
  }

  mounted(): void {
    this.retrieveData();
  }
}
</script>
