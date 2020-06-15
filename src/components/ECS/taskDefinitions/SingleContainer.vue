<template>
  <div>
    <DrawerCards :cards="cards" />

    <gl-table
      v-if="container.environment.length > 0"
      :items="container.environment"
      :fields="environmentFields"
      small
      caption-top
      stacked="sm"
    >
      <template v-slot:table-caption><h4>Environment variables</h4></template>
    </gl-table>
  </div>
</template>

<script lang="ts">
import { Component, Prop } from "vue-property-decorator";
import { DaintreeComponent } from "@/mixins/DaintreeComponent";
import { ContainerDefinition } from "aws-sdk/clients/ecs";
import { CardContent } from "@/components/common/cardContent";
import DrawerCards from "@/components/common/DrawerCards.vue";
import { GlTable } from "@gitlab/ui";

@Component({
  components: { DrawerCards, GlTable },
})
export default class SingleContainer extends DaintreeComponent {
  @Prop(Object) readonly container!: ContainerDefinition;

  readonly environmentFields = [
    {
      key: "name",
      sortable: true,
    },
    "value",
  ];

  //TODO: repositoryCredentials, links, portMappings, environmentFiles, mountPOints, volummesFrom, linuxParameters, secrets, dependsOn, dnsservers, dnssearchdomain, extrahosts, dockerSecurityOptions, dockerlabels, ulimits, logconfiguration, healthcheck, systemcontrol, resourcerequirements, firelensConfiguration
  get cards(): CardContent[] {
    return [
      {
        title: "Name",
        value: this.container.name,
        helpText:
          "The name of a container. This parameter maps to name in the Create a container section of the Docker Remote API and the --name option to docker run.",
      },
      {
        title: "Image",
        value: this.container.image,
        helpText:
          "The image used to start a container. This string is passed directly to the Docker daemon. Images in the Docker Hub registry are available by default. Other repositories are specified with either repository-url/image:tag or repository-url/image@digest . This parameter maps to Image in the Create a container section of the Docker Remote API and the IMAGE parameter of docker run.",
      },
      {
        title: "CPUs",
        value: this.container.cpu,
        helpText:
          "The number of cpu units reserved for the container. This parameter maps to CpuShares in the Create a container section of the Docker Remote API and the --cpu-shares option to docker run.",
      },
      {
        title: "Memory",
        value: `${this.container?.memory || "N/A"} MiB`,
        helpText:
          "The amount of memory to present to the container. If your container attempts to exceed the memory specified here, the container is killed. The total amount of memory reserved for all containers within a task must be lower than the task memory value, if one is specified. This parameter maps to Memory in the Create a container section of the Docker Remote API and the --memory option to docker run.",
      },
      {
        title: "Memory reservation",
        value: `${this.container?.memoryReservation || "N/A"} MiB`,
        helpText:
          "The soft limit (in MiB) of memory to reserve for the container. When system memory is under heavy contention, Docker attempts to keep the container memory to this soft limit. However, your container can consume more memory when it needs to, up to either the hard limit specified with the memory parameter (if applicable), or all of the available memory on the container instance, whichever comes first. This parameter maps to MemoryReservation in the Create a container section of the Docker Remote API and the --memory-reservation option to docker run.",
      },
      {
        title: "Is essential?",
        value: this.container.essential,
        helpText:
          "If the essential parameter of a container is marked as true, and that container fails or stops for any reason, all other containers that are part of the task are stopped. If the essential parameter of a container is marked as false, then its failure does not affect the rest of the containers in a task.",
      },
      {
        title: "Entry point",
        value: this.container.entryPoint?.join(" "),
        helpText:
          "The entry point that is passed to the container. This parameter maps to Entrypoint in the Create a container section of the Docker Remote API and the --entrypoint option to docker run.",
        isCode: true,
      },

      {
        title: "Command",
        value: this.container.command?.join(" "),
        helpText:
          "The command that is passed to the container. This parameter maps to Cmd in the Create a container section of the Docker Remote API and the COMMAND parameter to docker run.",
        isCode: true,
      },
      {
        title: "Start timeout",
        value: `${this.container.startTimeout || "N/A"} seconds`,
        helpText:
          "Time duration to wait before giving up on resolving dependencies for a container. For example, you specify two containers in a task definition with containerA having a dependency on containerB reaching a COMPLETE, SUCCESS, or HEALTHY status. If a startTimeout value is specified for containerB and it does not reach the desired status within that time then containerA will give up and not start. This results in the task transitioning to a STOPPED state.",
      },
      {
        title: "Stop timeout",
        value: `${this.container.stopTimeout || "N/A"} seconds`,
        helpText:
          "Time duration to wait before the container is forcefully killed if it doesn't exit normally on its own.",
      },
      {
        title: "Hostname",
        value: this.container.hostname,
        helpText:
          "The hostname to use for your container. This parameter maps to Hostname in the Create a container section of the Docker Remote API and the --hostname option to docker run.",
      },
      {
        title: "User",
        value: this.container.user,
        helpText:
          "The user name to use inside the container. This parameter maps to User in the Create a container section of the Docker Remote API and the --user option to docker run.",
      },
      {
        title: "Working directory",
        value: this.container.workingDirectory,
        helpText:
          "The working directory in which to run commands inside the container. This parameter maps to WorkingDir in the Create a container section of the Docker Remote API and the --workdir option to docker run.",
        isCode: true,
      },
      {
        title: "Disable networking?",
        value: this.container.disableNetworking,
        helpText:
          "When this parameter is true, networking is disabled within the container. This parameter maps to NetworkDisabled in the Create a container section of the Docker Remote API.",
      },
      {
        title: "Is privileged?",
        value: this.container.privileged,
        helpText:
          "When this parameter is true, the container is given elevated privileges on the host container instance (similar to the root user). This parameter maps to Privileged in the Create a container section of the Docker Remote API and the --privileged option to docker run.",
      },
      {
        title: "Read only root file system?",
        value: this.container.readonlyRootFilesystem,
        helpText:
          "When this parameter is true, the container is given read-only access to its root file system. This parameter maps to ReadonlyRootfs in the Create a container section of the Docker Remote API and the --read-only option to docker run.",
      },
      {
        title: "Is interactive?",
        value: this.container.interactive,
        helpText:
          "When this parameter is true, this allows you to deploy containerized applications that require stdin or a tty to be allocated. This parameter maps to OpenStdin in the Create a container section of the Docker Remote API and the --interactive option to docker run.",
      },
      {
        title: "Pseudo terminal?",
        value: this.container.pseudoTerminal,
        helpText:
          "When this parameter is true, a TTY is allocated. This parameter maps to Tty in the Create a container section of the Docker Remote API and the --tty option to docker run.",
      },
    ];
  }
}
</script>
