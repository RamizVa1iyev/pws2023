<template>
  <div>
    <v-card variant="text">
      <v-card-title>Projects</v-card-title>
      <v-card-subtitle>
        <v-container>
          <v-row>
            <v-col>
              <v-text-field
                variant="solo"
                label="Search"
                v-model="search"
                @input="retrieve"
              ></v-text-field>
            </v-col>
            <v-col cols="2">
              <v-text-field
                variant="solo"
                type="number"
                label="Skip"
                v-model="skip"
                @input="retrieve"
              ></v-text-field>
            </v-col>
            <v-col cols="2">
              <v-text-field
                variant="solo"
                type="number"
                label="Limit"
                v-model="limit"
                @input="retrieve"
              ></v-text-field>
            </v-col>
          </v-row>
        </v-container>
      </v-card-subtitle>
      <v-card-text>
        <v-table density="compact" hover>
          <thead>
            <tr>
              <th class="text-left">Name</th>
              <th class="text-left">Shortcut</th>
              <th class="text-right">Start date</th>
              <th class="text-right">Members</th>
              <th>#</th>
            </tr>
          </thead>
          <tbody>
            <tr
              v-for="(project, index) in projects"
              :key="index"
              @click="checkIfInRole(user, [1]) && click(project)"
            >
              <td>{{ project.name }}</td>
              <td>
                <v-chip :color="project.color">{{ project.shortcut }}</v-chip>
              </td>
              <td class="text-right">
                {{ new Date(project.startDate).toLocaleDateString() }}
              </td>
              <td class="text-right">{{ project.members }}</td>
              <td>
                <v-btn @click="(e) => openChart(e, project._id)"
                  ><v-icon icon="mdi-chart-timeline"></v-icon
                ></v-btn>
              </td>
            </tr>
          </tbody>
        </v-table>
      </v-card-text>
      <v-card-actions>
        <v-spacer></v-spacer>
        <v-btn
          variant="elevated"
          color="success"
          @click="add"
          v-if="checkIfInRole(user, [1])"
          >Add</v-btn
        >
      </v-card-actions>
    </v-card>
    <v-dialog v-model="editor" width="50%">
      <ProjectEditor :id="id" @dataChanged="retrieve" @cancel="cancel" />
    </v-dialog>
    <v-dialog v-model="chart" width="50%">
      <ProjectTaskChart :project="id" />
    </v-dialog>
  </div>
</template>

<script>
import common from "../mixins/common";
import ProjectEditor from "./ProjectEditor.vue";
import ProjectTaskChart from "./ProjectTaskChart.vue";

export default {
  name: "ProjectsLister",
  components: { ProjectEditor, ProjectTaskChart },
  mixins: [common],
  props: ["user"],
  methods: {
    retrieve() {
      this.id = null;
      this.editor = false;
      fetch(
        "/project?search=" +
          this.search +
          "&skip=" +
          this.skip +
          "&limit=" +
          this.limit,
        {
          method: "GET",
        }
      )
        .then((res) => {
          res
            .json()
            .then((data) => {
              this.projects = data;
            })
            .catch((err) => console.error(err.message));
        })
        .catch((err) => console.error(err.message));
    },
    add() {
      this.id = null;
      this.editor = true;
    },
    click(row) {
      this.id = row._id;
      this.editor = true;
    },
    cancel() {
      this.id = null;
      this.editor = false;
    },
    openChart(e, projectId) {
      e.stopPropagation();
      this.id = projectId;
      this.chart = true;
    },
  },
  data() {
    return {
      editor: false,
      projects: [],
      id: null,
      search: "",
      skip: 0,
      limit: 10,
      chart: false,
    };
  },
  mounted() {
    this.retrieve();
  },
};
</script>
