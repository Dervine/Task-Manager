<template>
    <v-data-table
      :headers="headers"
      :items="users"
      :hide-default-footer="true"
      @click:row="handleClick"
      class="elevation-1"
    >
      <template v-slot:top>
        <v-toolbar
          flat
        >
          <v-toolbar-title>Users</v-toolbar-title>
          <v-divider
            class="mx-4"
            inset
            vertical
          ></v-divider>
          <v-spacer></v-spacer>
          <v-dialog
            v-model="dialog"
            max-width="500px"
          >
            <template v-slot:activator="{ on, attrs }">
              <v-btn
                color="primary"
                dark
                class="mb-2 btn-new-user"
                v-bind="attrs"
                v-on="on"
              >
                New User
              </v-btn>
            </template>
            <v-card>
              <v-card-title>
                <span class="text-h5">{{ formTitle }}</span>
              </v-card-title>
  
              <v-card-text>
                <v-container>
                  <v-row>
                    <v-col
                      cols="12"
                      sm="6"
                      md="4"
                    >
                      <v-text-field
                        v-model="editedItem.name"
                        label="User Name"
                      ></v-text-field>
                    </v-col>
                    <v-col
                      cols="12"
                      sm="6"
                      md="4"
                    >
                      <v-text-field
                        v-model="editedItem.email"
                        label="User Email"
                      ></v-text-field>
                    </v-col>
                    <v-col
                      cols="12"
                      sm="6"
                      md="4"
                    >
                      <v-text-field
                        type = "password"
                        v-model="editedItem.password"
                        label="Password"
                      ></v-text-field>
                    </v-col>
                  </v-row>
                </v-container>
              </v-card-text>
  
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn
                  color="blue darken-1"
                  text
                  @click="close"
                >
                  Cancel
                </v-btn>
                <v-btn
                  color="blue darken-1"
                  text
                  @click="save"
                >
                  Save
                </v-btn>
              </v-card-actions>
            </v-card>
          </v-dialog>
          <v-dialog v-model="dialogDelete" max-width="500px">
            <v-card>
              <v-card-title class="text-h5">Are you sure you want to delete this user?</v-card-title>
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="blue darken-1" text @click="closeDelete">Cancel</v-btn>
                <v-btn color="blue darken-1" text @click="deleteItemConfirm(item)">OK</v-btn>
                <v-spacer></v-spacer>
              </v-card-actions>
            </v-card>
          </v-dialog>
        </v-toolbar>
      </template>
      <template v-slot:[`item.actions`]="{ item }">
        <v-icon
          small
          class="mr-2"
          @click="editItem(item)"
        >
          mdi-pencil
        </v-icon>
        <v-icon
          small
          @click="deleteItem(item)"
        >
          mdi-delete
        </v-icon>
      </template>
      <template v-slot:no-data>
        <v-btn
          color="primary"
          @click="initialize"
        >
          Reset
        </v-btn>
      </template>
    </v-data-table>
  </template>

<script>
  export default {
    data: () => ({
      dialog: false,
      dialogDelete: false,
      currentUser: 0,
      headers: [
        { text: 'Id', value: 'id', align:'left' },
        {
          text: 'Name',
          align: 'start',
          sortable: false,
          value: 'name',
        },
        { text: 'Email', value: 'email', sortable: false, },
        { text: 'Actions', value: 'actions', sortable: false },
      ],
      users: [],
      editedIndex: -1,
      editedItem: {
        name: '',
        email: '',
        password: '',
      },
      defaultItem: {
        name: '',
        email: '',
        password: '',
      },
    }),

    computed: {
      baseUrl () {
        return 'http://localhost:3333/v1/users/'
      },
      formTitle () {
        return this.editedIndex === -1 ? 'New User' : 'Edit User'
      },
    },

    watch: {
      dialog (val) {
        val || this.close()
      },
      dialogDelete (val) {
        val || this.closeDelete()
      },
    },

    created () {
      this.initialize()
    },

    methods: {
      async initialize () {
        const self = this;
        fetch(this.baseUrl)
        .then(
        function(response) {
            if (response.status === 200) {
              response.json().then(function(data) {
              self.users = data;
            });
            }
        }
        )
        .catch(function(err) {
        console.log('Fetch Error :-S', err);
        });
      },

      refreshList() {
        this.initialize();
      },

      editItem (item) {
        this.editedIndex = this.users.indexOf(item)
        this.editedItem = Object.assign({}, item)
        this.dialog = true
      },

      deleteItem (item) {
        this.editedIndex = this.users.indexOf(item)
        this.editedItem = Object.assign({}, item)
        this.dialogDelete = true
      },

      handleClick(value) {
        this.currentUser = value.id
      },

      deleteItemConfirm () {
        const self = this;
        fetch(this.baseUrl + this.currentUser, {
          method: 'DELETE',
        })
        .then(res => {     console.log(res);     self.refreshList() })
        this.closeDelete()
      },

      close () {
        this.dialog = false
        this.$nextTick(() => {
          this.editedItem = Object.assign({}, this.defaultItem)
          this.editedIndex = -1
        })
      },

      closeDelete () {
        this.dialogDelete = false
        this.$nextTick(() => {
          this.editedItem = Object.assign({}, this.defaultItem)
          this.editedIndex = -1
        })
      },

      save () {
        const self = this;
        let callMethod = null;
        let url = '';
        if(this.editedItem.id) {
          url = `http://localhost:3333/v1/users/${this.editedItem.id}`
          callMethod = {
            method: 'put',
            headers: {
              'Content-type': 'application/json; charset=UTF-8' 
            },
            body: JSON.stringify(this.editedItem)
          }
        } else {
          url = this.baseUrl
          callMethod = {
            method: 'post',
            headers: {
              'Content-type': 'application/json; charset=UTF-8' 
            },
            body: JSON.stringify(this.editedItem)
          }
        }
        fetch(url, callMethod)
          .then(response => response.json())
          .then(res => {     console.log(res);     self.refreshList() })
          .catch(function (error) {
          console.log('Request failed', error);
        });
        this.close()
      },
    },
  }
</script>
<style scoped>
.btn-new-user {
  background-color: red !important ;
}
</style>