<template>
    <div id="app">
        <div class="container-fluid p-4">
            <div class="row">
                <div class="col-12">
                    <div class="card">
                        <div class="card-header">
                            Users
                            <button class="btn btn-success btn-sm float-right" v-on:click="addUser()">
                                <icon icon="fa-solid fa-plus" />
                            </button>
                            <button class="btn btn-success btn-sm float-right mr-2" v-on:click="getUsers()">
                                <icon icon="fa-solid fa-refresh" />
                            </button>
                        </div>
                        <div class="card-body">
                            <table class="table table-striped">
                                <thead>
                                    <tr>
                                        <th scope="col">#</th>
                                        <th scope="col">Profile</th>
                                        <th scope="col">Name</th>
                                        <th scope="col">Email</th>
                                        <th scope="col">Role</th>
                                        <th scope="col">Actions</th>
                                    </tr>
                                </thead>
                                <tbody v-if="users && users.length > 0">
                                    <tr v-for="(user, index) in users" v-bind:key="index">
                                        <th scope="row" v-text="user.id" class="align-middle"></th>
                                        <td class="align-middle">
                                            <img :src="user.image_link" :alt="user.name" class="img-fluid" width="100px">
                                        </td>
                                        <td v-text="user.name" class="align-middle"></td>
                                        <td v-text="user.email" class="align-middle"></td>
                                        <td v-text="getRole(user)" class="align-middle"></td>
                                        <td class="align-middle">
                                            <a href="#!" v-on:click="editUser(user)" class="btn btn-sm btn-primary">
                                                <icon icon="fa-solid fa-edit" />
                                            </a>
                                            <a href="#!" class="btn btn-sm btn-danger">
                                                <icon icon="fa-solid fa-trash" />
                                            </a>
                                        </td>
                                    </tr>
                                </tbody>
                                <tbody v-if="!users || users.length == 0">
                                    <tr>
                                        <td colspan="6">
                                            <h6 class="text-center">No data available</h6>
                                        </td>
                                    </tr>
                                </tbody>
                            </table>
                        </div>
                    </div>
                </div>
            </div>
        </div>
        <div class="modal fade" id="formModal" tabindex="-1" role="dialog" aria-labelledby="formModalLabel"
            aria-hidden="true">
            <div class="modal-dialog" role="document">
                <form v-on:submit.prevent="sendPost()" class="was-validated" enctype="multipart/form-data">
                    <div class="modal-content">
                        <div class="modal-header">
                            <h5 class="modal-title" id="formModalLabel" v-text="title"></h5>
                            <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                                <span aria-hidden="true">&times;</span>
                            </button>
                        </div>
                        <div v-if="!loading" class="modal-body">
                            <div class="form-row">
                                <div class="col-12 mb-3">
                                    <label for="name">Name</label>
                                    <input type="text" class="form-control" v-model="user.name" id="name" placeholder="Name"
                                        required>
                                </div>
                                <div class="col-12 mb-3">
                                    <label for="email">Email</label>
                                    <input type="email" class="form-control" v-model="user.email" id="email"
                                        placeholder="Email" required>
                                </div>
                                <div class="col-12 mb-3">
                                    <label for="role_id">Role</label>
                                    <select name="role_id" id="role_id" class="form-control" v-model="user.role_id"
                                        required>
                                        <option value="">Select a role</option>
                                        <option v-for="(role, index) in roles" :value="role.id" :key="index"
                                            v-text="role.name"></option>
                                    </select>
                                </div>
                                <div v-if="action == 'add'" class="col-12 mb-3">
                                    <label for="password">Password</label>
                                    <input type="password" class="form-control" v-model="user.password" id="password"
                                        required>
                                </div>

                                <div v-if="action == 'add'" class="col-12 mb-3">
                                    <label for="password_confirmation">Password confirmation</label>
                                    <input type="password" class="form-control" v-model="user.password_confirmation"
                                        id="password_confirmation" required>
                                </div>

                                <div class="col-12 mb-3">
                                    <label for="image">Image</label>
                                    <div class="input-group">
                                        <div class="custom-file">
                                            <input type="file" class="custom-file-input" id="image" ref="file"
                                                v-on:change="selectFile()">
                                            <label class="custom-file-label" for="image" id="image-label"
                                                v-text="image_label"></label>
                                        </div>
                                    </div>
                                </div>
                            </div>
                            <div class="modal-footer">
                                <button type="button" class="btn btn-danger" data-dismiss="modal">Cancel</button>
                                <button type="submit" class="btn btn-success">Save</button>
                            </div>
                        </div>
                        <div v-if="loading" class="modal-body">
                            <p class="text-center mt-0 mb-0">
                                <icon icon="fa-solid fa-spinner" />
                                Loading...
                            </p>
                        </div>
                    </div>
                </form>
            </div>
        </div>
    </div>
</template>

<script>

class User {
    constructor(
        id = '',
        name = '',
        email = '',
        image = '',
        password = '',
        password_confirmation = '',
        role_id = ''
    ) {
        this.id = id;
        this.name = name;
        this.email = email;
        this.image = image;
        this.password = password;
        this.password_confirmation = password_confirmation;
        this.role_id = role_id;
    }
}

export default {
    name: "App",
    data() {
        return {
            users: [],
            user: new User(),
            roles: [],
            file: null,
            title: null,
            action: null,
            image_label: 'Choose file',
            loading: false,
        };
    },
    mounted() {
        this.getUsers();
        this.getRoles();
    },
    methods: {
        getUsers() {
            axios.get('/users').then(data => {
                return data.data.data;
            })
                .then((data) => {
                    this.users = data;
                })
                .catch(err => {
                    console.error(JSON.stringify(err));
                });
        },
        getRoles() {
            axios.get('/roles').then(data => {
                return data.data.data;
            })
                .then((data) => {
                    this.roles = data;
                })
                .catch(err => {
                    console.error(JSON.stringify(err));
                });
        },
        getRole(user) {
            return user.roles[0].name
        },
        editUser(user) {

            this.resetForm();

            this.title = 'Edit user';
            this.action = 'edit';

            this.user = new User(user.id, user.name, user.email, '', '', '', user.roles[0].id);
            $('#formModal').modal('show');

        },
        addUser() {
            this.resetForm();

            this.title = 'Add new user';
            this.action = 'add';

            $('#formModal').modal('show');
        },
        resetForm() {
            this.user = new User();
            this.file = null
            this.title = null;
            this.action = null;
            this.loading = false;
            this.image_label = 'Choose file';
        },
        sendPost() {

            this.loading = true;
            let data = this.user;

            if (this.file && this.file.name != '') {

                let formData = new FormData();
                formData.append("name", this.user.name);
                formData.append("email", this.user.email);
                formData.append("password", this.user.password);
                formData.append("password_confirmation", this.user.password_confirmation);
                formData.append("role_id", this.user.role_id);
                formData.append("image", this.file);

                data = formData;
            } else {
                delete data.image;
            }

            let url = this.action == 'edit' ? `/users/${this.user.id}` : '/users';

            if (this.action == 'edit') {
                url = `${url}?_method=PUT`;
            }

            axios.post(url, data)
                .then(() => {

                    this.getUsers();
                    this.resetForm();

                    $('#formModal').modal('hide');

                })
                .catch(err => {
                    console.error(JSON.stringify(err));
                    this.loading = false;
                });
        },
        selectFile() {
            this.file = this.$refs.file.files[0];
            this.image_label = this.file.name;
        }
    }
};
</script>

<style>
@keyframes spinner {
    to {
        transform: rotate(360deg);
    }
}

.fa-spinner {
    animation: spinner 1s linear infinite;
}
</style>