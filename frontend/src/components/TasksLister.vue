<template>
  <div v-if="checkIfInRole(user, [0, 1])">
    <v-card variant="text">
      <v-card-title>Tasks</v-card-title>
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
            <v-col>
              <v-select
                label="Project"
                v-model="project"
                @update:modelValue="retrieve"
                :items="
                  projects.map((project) => ({
                    value: project._id,
                    title: project.shortcut,
                    props: {
                      subtitle:
                        project.name + ' ' + project.startDate.slice(0, 10),
                    },
                  }))
                "
              ></v-select>
            </v-col>
            <v-col cols="3">
              <v-select
                v-model="status"
                label="Status"
                :items="[
                  { value: 0, title: 'PREPARATION' },
                  { value: 1, title: 'PENDING' },
                  { value: 2, title: 'IN TESTS' },
                  { value: 3, title: 'COMPLETED' },
                ]"
                chips
                multiple
                @update:modelValue="retrieve"
              >
              </v-select>
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
              <th class="text-left">Project Name</th>
              <th class="text-left">Start date</th>
              <th class="text-left">End date</th>
              <th class="text-left">Status</th>
              <th class="text-left">Workers</th>
            </tr>
          </thead>
          <tbody>
            <tr
              v-for="(task, index) in tasks"
              :key="index"
              @click="checkIfInRole(user, [1]) && click(task)"
            >
              <td>{{ task.name }}</td>
              <td>{{ task.projectName }}</td>
              <td>
                {{ new Date(task.startDate).toLocaleDateString() }}
              </td>
              <td>
                {{
                  task.endDate && new Date(task.endDate).toLocaleDateString()
                }}
              </td>
              <td>
                <v-chip>{{
                  ["PREPARATION", "PENDING", "IN TESTS", "COMPLETED"][
                    task.status
                  ]
                }}</v-chip>
              </td>
              <td>
                <v-chip v-for="(worker, pindex) in task.workers" :key="pindex">
                  {{ worker.firstName + " " + worker.lastName }}
                </v-chip>
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
          :disabled="!project"
          v-if="checkIfInRole(user, [1])"
          >Add</v-btn
        >
      </v-card-actions>
    </v-card>
    <v-dialog v-model="editor" width="50%">
      <TaskEditor
        :id="id"
        :project="project"
        @dataChanged="retrieve"
        @cancel="cancel"
      />
    </v-dialog>
  </div>
</template>

<script>
import common from "../mixins/common";
import TaskEditor from "./TaskEditor.vue";

export default {
  name: "TasksLister",
  components: { TaskEditor },
  props: ["user"],
  mixins: [common],
  methods: {
    retrieve() {
      this.id = null;
      this.editor = false;
      fetch(
        "/task?search=" +
          this.search +
          "&status=" +
          JSON.stringify(this.status) +
          "&project=" +
          (this.project ?? "") +
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
              this.tasks = data;
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
  },
  data() {
    return {
      editor: false,
      tasks: [],
      id: null,
      search: "",
      status: [0, 1, 2, 3],
      skip: 0,
      limit: 10,
      projects: [],
      project: null,
    };
  },
  mounted() {
    fetch("/project?limit=1000", { method: "GET" })
      .then((res) => res.json())
      .then((data) => {
        this.projects = data;
      });
    this.retrieve();
  },
};
</script>
