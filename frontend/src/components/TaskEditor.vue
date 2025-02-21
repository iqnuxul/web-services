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
                        v-model="task.project"
                        label="Project"
                        :rules="[rules.required]"
                        :items="
                            projects.map((project) => ({
                                value: project._id,
                                title: project.name,
                            }))
                        "
                        chips
                        readonly="true"
                    >
                    </v-select>
                    <v-radio-group label="Status" v-model="task.status" inline>
                        <v-radio :value="0" label="PREPARATION"></v-radio>
                        <v-radio :value="1" label="PENDING"></v-radio>
                        <v-radio :value="2" label="IN TESTS"></v-radio>
                        <v-radio :value="3" label="COMPLETED"></v-radio>
                    </v-radio-group>
                    <v-select
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
                    <v-text-field
                        variant="solo"
                        type="date"
                        label="Start date"
                        v-model="task.startDate"
                        :rules="[rules.required, rules.validStartDate]"
                    ></v-text-field>
                    <v-text-field
                        variant="solo"
                        type="date"
                        label="End date"
                        v-model="task.endDate"
                        :rules="[rules.validStartDate]"
                    ></v-text-field>
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
                <v-btn
                    variant="elevated"
                    color="error"
                    @click="remove"
                    v-if="id"
                    >Remove</v-btn
                >
                <v-btn variant="elevated" color="warning" @click="cancel"
                    >Cancel</v-btn
                >
            </v-card-actions>
        </v-card>
        <v-dialog v-model="confirmation" width="auto">
            <ConfirmationDialog
                :question="'Are you sure to delete ?'"
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
    props: ["id", "projectId"],
    components: { ConfirmationDialog },
    emits: ["cancel", "dataChanged"],
    methods: {
        add() {
            fetch("/task", {
                method: "POST",
                headers: { "Content-Type": "application/json" },
                body: JSON.stringify(this.task),
            })
                .then((res) => {
                    res.json()
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
                    res.json()
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
                    res.json()
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
            projects: [],
            workers: [],
        };
    },
    mounted() {
        fetch("/project?limit=1000", { method: "GET" })
            .then((res) => res.json())
            .then((data) => (this.projects = data));
        fetch("/person?limit=1000&minProjects=1", { method: "GET" })
            .then((res) => res.json())
            .then((data) => {
                console.dir(data);
                console.log("this.projectId:", this.projectId);
                this.workers = data.filter((user) =>
                    user.projects.some(
                        (project) => project._id === this.projectId
                    )
                );
                console.log("this.workers:", this.workers);
            });
        if (this.id) {
            fetch("/task?_id=" + this.id, { method: "GET" })
                .then((res) => {
                    res.json()
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
        if (this.projectId) {
            this.task.project = this.projectId;
        }
    },
};
</script>
