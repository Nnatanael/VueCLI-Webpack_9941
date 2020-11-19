<template>
    <v-main class="list">
        <h3 class="text-h3 font-weight-medium mb-5">To Do List</h3>

        <v-card>
            <v-card-title>
                <v-text-field
                    v-model="search"
                    append-icon="mdi-magnify"
                    label="Search"
                    single-line
                    hide-details>
                </v-text-field>
                <v-spacer></v-spacer>
                <v-btn color="success" dark @click="dialog = true">
                    Tambah
                </v-btn>
            </v-card-title>
            <v-data-table 
                :headers="headers" 
                :items="todos" 
                :search="search" 
                item-key="task"
                show-expand
            >
                <template v-slot:[`item.actions`]="{ item }">
                    <v-btn small class="mr-2" @click="editItem(item)">
                        edit
                    </v-btn>
                    <v-btn small @click="showDeleteDialog(item)">
                        delete
                    </v-btn>
                </template>

                <template v-slot:[`item.priority`]="{ item }">
                    <v-chip v-if="item.priority == 'Penting'" color="red" outlined>{{item.priority}}</v-chip>
                    <v-chip v-if="item.priority == 'Tidak penting'" color="green" outlined>{{item.priority}}</v-chip>
                    <v-chip v-if="item.priority == 'Biasa'" color="blue" outlined>{{item.priority}}</v-chip>
                </template>

                <template v-slot:[`item.checkbox`]="{ item }">
                    <v-container fluid>
                        <v-checkbox
                           color="red"
                           @click="pushSelected(item)"
                        ></v-checkbox>
                    </v-container>
                </template>

            </v-data-table>
        </v-card>

        <!-- DIALOG FORM -->
        <v-dialog v-model="dialog" persistent max-width="600px">
            <v-card>
                <v-card-title>
                    <span class="headline">Form Todo</span>
                </v-card-title>
                <v-card-text>
                    <v-container>
                        <v-text-field
                            v-model="formTodo.task"
                            label="Task"
                            required>
                        </v-text-field>

                        <v-select
                            v-model="formTodo.priority"
                            :items="['Penting', 'Biasa', 'Tidak penting']"
                            label="Priority"
                            required>
                        </v-select>

                        <v-textarea
                            v-model="formTodo.note"
                            label="Note"
                            required>
                        </v-textarea>
                    </v-container>
                </v-card-text>
                <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn color="blue darken-1" text @click="cancel">
                        Cancel
                    </v-btn >
                    <v-btn v-if="editing == 0" color="blue darken-1" text @click="save">
                        Save
                    </v-btn>
                    <v-btn v-if="editing == 1" text @click="saveEdit(item)">Save</v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>

        <v-dialog v-model="dialogDelete" max-width="400px">
            <v-card>
                <v-card-title class="headline">
                    Yakin ingin menghapus ?
                </v-card-title>
                <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn
                        color="green darken-1"
                        text
                        @click="dialogDelete = false"
                    >
                        Tidak
                    </v-btn>
                    <v-btn
                        color="red darken-1"
                        text
                        @click="deleteItem"
                    >
                        Ya
                    </v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>

        <v-card>
            <span v-if="selectedItem.length >0">
               <div>
                <p>Delete Mutiple : </p>
                <ul v-for="(item, index) in selectedItem" :key="index">
                    {{ item.task }}
                </ul>
                <br>
                <v-btn color="red" @click="deleteAll">
                    Hapus Semua
                </v-btn>
            </div>
            </span>
            
        </v-card>

    </v-main>
</template>

<script>
export default {
    name: "List",
    data() {
        return {
            search: null,
            dialog: false,
            index: null,
            dialogDelete: false,
            editing: 0,
            selectedItem: [],
            headers: [
                { text:  '', value: 'data-table-expand' },
                {
                    text: "Task",
                    align: "start",
                    sortable: true,
                    value: "task",
                },
                { text: "Priority", value: "priority" },
                { text: "Actions", value: "actions" },
                { text: '', value: 'checkbox' },
            ],
            todos: [
                {
                    task: "bernafas",
                    priority: "Penting",
                    note: "huffttt",
                },
                {
                    task: "nongkrong",
                    priority: "Tidak penting",
                    note: "bersama teman2",
                },
                {
                    task: "masak",
                    priority: "Biasa",
                    note: "masak air 500ml",
                },
            ],
            formTodo: {
                task: null,
                priority: null,
                note: null,
            },
            selected: {
                task: null,
                priority: null,
                note: null,
            },
        };
    },
    methods: {
        save() {
            if (this.index != null) {
                Object.assign(this.todos[this.index], this.formTodo)
            } else {
                this.todos.push(this.formTodo)
            }

            this.resetForm();
            this.index = null;
            this.dialog = false;
        },
        saveEdit(item) {
            this.todos.splice(item, 1);
            this.todos.push(this.formTodo);
            this.dialog = false;
            this.editing = 0;
            this.resetForm();
        },
        cancel() {
            this.resetForm();
            this.dialog = false;
        },
        resetForm() {
            this.formTodo = {
                task: null,
                priority: null,
                note: null,
            };
        },
        editItem(item) {
            this.dialog = true
            this.index = this.todos.indexOf(item)
            this.formTodo = Object.assign({}, item)
        },
        showDeleteDialog(item) {
            this.index = this.todos.indexOf(item);
            this.dialogDelete = true;
        },
        deleteItem() {
            this.todos.splice(this.index, 1);
            this.dialogDelete = false;
            this.index = null;
        },

        pushSelected(item) {
            if(this.selectedItem.includes(item)) {
                this.selectedItem.splice(this.selectedItem.indexOf(item), 1);
            } else {
                 this.selectedItem.push(item);
            }
        },

        deleteAll(){
            for(var i = 0; i < this.selected.length; i++) {
                for(var j = 0; j < this.todos.length; j++){
                    if(this.selected.getAttribute('task')==this.todos.getAttribute('task')){
                        this.index = this.todos.indexOf(this.todos[j]);
                        this.todos.splice(this.index, 1);
                    }
                }
            }
            this.index = null;
            this.selectedItem = [];
        },
    },
};
</script>