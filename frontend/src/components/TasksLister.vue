<template>
    <div v-if="checkIfInRole(user, [1])">
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
                        <v-col cols="3">
                            <v-select
                                v-model="project"
                                label="project"
                                :items="projects"
                                @update:modelValue="retrieve"
                            >
                            </v-select>
                        </v-col>
                        <v-col cols="3">
                            <v-select
                                v-model="status"
                                label="status"
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
                            <th class="text-left">name</th>
                            <th class="text-left">project</th>
                            <th class="text-left">status</th>
                            <th class="text-left">workers</th>
                            <th class="text-left">Start date</th>
                            <th class="text-left">End date</th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr
                            v-for="(task, index) in tasks"
                            :key="index"
                            @click="checkIfInRole(user, [1]) && click(task)"
                        >
                            <td>{{ task.name }}</td>
                            <td>{{ task.project ? task.project.name : "" }}</td>
                            <td>
                                <v-chip>{{
                                    [
                                        "PREPARATION",
                                        "PENDING",
                                        "IN TESTS",
                                        "COMPLETED",
                                    ][task.status]
                                }}</v-chip>
                            </td>
                            <td>
                                <v-chip
                                    v-for="(worker, wIndex) in task.workers"
                                    :key="wIndex"
                                >
                                    {{ worker.firstName }} {{ worker.lastName }}
                                </v-chip>
                            </td>
                            <td>
                                {{
                                    new Date(
                                        task.startDate
                                    ).toLocaleDateString()
                                }}
                            </td>
                            <td>
                                {{
                                    new Date(task.endDate).toLocaleDateString()
                                }}
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
            <TaskEditor
                :id="id"
                :projectId="project"
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
                    "&limit=" +
                    this.limit +
                    "&status=" +
                    JSON.stringify(this.status) +
                    "&project=" +
                    this.project,
                {
                    method: "GET",
                }
            )
                .then((res) => {
                    res.json()
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
            limit: 10,
            projects: [],
            project: "",
            status: [0, 1, 2, 3],
        };
    },
    mounted() {
        // 获取所有项目
        fetch("/project?limit=1000", { method: "GET" })
            .then((res) => res.json())
            .then((data) => {
                // 判断data是数组并且有值
                console.log(data)
                if (Array.isArray(data) && data.length > 0) {
                    this.projects = data.map((item) => ({
                        value: item._id,
                        title: item.name,
                    }));
                    this.project = this.projects[0].value;
                }
                this.retrieve();
            });
    },
};
</script>
