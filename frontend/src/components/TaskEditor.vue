<template>
  <div>
    <v-card>
      <v-card-title>{{ id ? "Edit" : "Create" }} task</v-card-title>
      <v-card-text>
        <v-form v-model="isTaskValid">
          <v-text-field
            variant="solo"
            label="Name"
            v-model="task.name"
            :rules="[rules.required]"
          ></v-text-field>
          <v-select
            label="Project"
            @update:modelValue="getWorkers"
            v-model="task.project_id"
            :rules="[rules.required]"
            :items="
              projects.map((project) => ({
                value: project._id,
                title: project.shortcut,
                props: {
                  subtitle: project.name + ' ' + project.startDate.slice(0, 10),
                },
              }))
            "
          ></v-select>
          <v-text-field
            variant="solo"
            type="date"
            label="Start date"
            v-model="task.startDate"
            :rules="[rules.validBirthDate]"
          ></v-text-field>
          <v-text-field
            variant="solo"
            type="date"
            label="End date"
            v-model="task.endDate"
          ></v-text-field>
          <v-radio-group label="Status" v-model="task.status" inline>
            <v-radio :value="0" label="PREPARATION"></v-radio>
            <v-radio :value="1" label="PENDING"></v-radio>
            <v-radio :value="2" label="IN TESTS"></v-radio>
            <v-radio :value="3" label="COMPLETED"></v-radio>
          </v-radio-group>
          <v-select
            v-if="task.project_id && workers.length > 0"
            v-model="task.workers"
            label="Workers"
            :items="
              workers.map((worker) => ({
                value: worker._id,
                title: worker.firstName + ' ' + worker.lastName,
              }))
            "
            chips
            multiple
          >
          </v-select>
        </v-form>
      </v-card-text>
      <v-card-actions>
        <v-spacer></v-spacer>
        <v-btn
          variant="elevated"
          color="success"
          @click="add"
          :disabled="!isTaskValid"
          v-if="!id"
          >Add</v-btn
        >
        <v-btn
          variant="elevated"
          color="success"
          @click="modify"
          :disabled="!isTaskValid"
          v-if="id"
          >Modify</v-btn
        >
        <v-btn variant="elevated" color="error" @click="remove" v-if="id"
          >Remove</v-btn
        >
        <v-btn variant="elevated" color="warning" @click="cancel">Cancel</v-btn>
      </v-card-actions>
    </v-card>
    <v-dialog v-model="confirmation" width="auto">
      <ConfirmationDialog
        :question="'Are you sure to delete \'' + task.name + '\' ?'"
        @ok="removeReal"
        @cancel="confirmation = false"
      />
    </v-dialog>
  </div>
</template>

<script>
import ConfirmationDialog from "./ConfirmationDialog.vue";

export default {
  name: "TaskEditor",
  props: ["id"],
  components: { ConfirmationDialog },
  emits: ["cancel", "dataChanged"],
  methods: {
    add() {
      console.log(this.task);
      fetch("/task", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(this.task),
      })
        .then((res) => {
          res
            .json()
            .then(() => {
              this.$emit("dataChanged");
            })
            .catch((err) => console.error(err.message));
        })
        .catch((err) => console.error(err.message));
    },
    modify() {
      fetch("/task?_id=" + this.id, {
        method: "PUT",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(this.task),
      })
        .then((res) => {
          res
            .json()
            .then(() => {
              this.$emit("dataChanged");
            })
            .catch((err) => console.error(err.message));
        })
        .catch((err) => console.error(err.message));
    },
    remove() {
      this.confirmation = true;
    },
    removeReal() {
      this.confirmation = false;
      fetch("/task?_id=" + this.id, {
        method: "DELETE",
      })
        .then((res) => {
          res
            .json()
            .then(() => {
              this.$emit("dataChanged");
            })
            .catch((err) => console.error(err.message));
        })
        .catch((err) => console.error(err.message));
    },
    cancel() {
      this.$emit("cancel");
    },
    getWorkers() {
      setTimeout(() => {
        fetch("/person?project_id=" + this.task.project_id + "&limit=1000", {
          method: "GET",
        })
          .then((res) => res.json())
          .then((data) => {
            this.workers = data;
          });
      }, 300);
    },
  },
  data() {
    return {
      isTaskValid: false,
      rules: {
        required: (value) => !!value || "empty value is not allowed",
        validBirthDate: (value) =>
          !isNaN(new Date(value)) || "valid date required",
      },
      task: {},
      dialog: false,
      confirmation: false,
      workers: [],
      projects: [],
    };
  },
  mounted() {
    fetch("/project?limit=1000", { method: "GET" })
      .then((res) => res.json())
      .then((data) => (this.projects = data));
    if (this.id) {
      fetch("/task?_id=" + this.id, { method: "GET" })
        .then((res) => {
          res
            .json()
            .then((data) => {
              Object.assign(this.task, data);
            })
            .catch((err) => console.error(err.message));
        })
        .catch((err) => console.error(err.message));
    } else {
      this.task = {
        startDate: new Date().toISOString().slice(0, 10),
        status: 0,
      };
    }
  },
};
</script>
