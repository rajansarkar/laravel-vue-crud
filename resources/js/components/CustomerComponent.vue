<template>
    <div id="customer">
        <div class="row justify-content-center">
            <div class="col-md-12">
                <div class="card">
                    <div class="card-header">
                        <div class="clearfix">
                            <h4 class="float-left">Customers</h4>
                            <button @click="createCustomer()" class="btn btn-sm btn-success float-right ml-2">Add Customer <i class="fas fa-plus"></i></button>
                            <button @click="reloadPage()" class="btn btn-sm btn-primary float-right">Reload <i class="fas fa-sync"></i></button>
                        </div>
                        
                    </div>
                    <div class="card-body">
                        <div class="search-div">
                        <div class="form-group row">
                            <label for="inputPassword" class="col-sm-2 col-form-label">Search By :</label>
                            <select v-model="queryField" class="form-control col-sm-3">
                                <option value="name">Name</option>
                                <option value="email">Email</option>
                                <option value="phone">Phone</option>
                                <option value="address">Address</option>
                                <option value="total">Total</option>
                            </select>
                            <div class="col-sm-7">
                                <input type="text" v-model="query" class="form-control" placeholder="search here..">
                            </div>
                        </div>
                        </div>
                            <div class="table-responsive" v-if="customers.length>0">
                                <table class="table table-striped table-hover">

                                        <thead class="thead-dark">
                                        <tr>
                                            <th scope="col">#</th>
                                            <th scope="col">Name</th>
                                            <th scope="col">Email</th>
                                            <th scope="col">Phone</th>
                                            <th scope="col">Total</th>
                                            <th scope="col">Action</th>
                                        </tr>
                                        </thead>
                                        <tbody>
                                        <tr v-for="(customer, index) in customers" :key="index">
                                            <th scope="row">{{(index+1)+(pagination.from-1)}}</th>
                                            <td>{{customer.name}}</td>
                                            <td>{{customer.email}}</td>
                                            <td>{{customer.phone}}</td>
                                            <td>{{customer.total}}</td>
                                            <td>
                                                <button class="btn btn-sm btn-info" @click="viewCustomer(customer)"><i class="fas fa-eye"></i></button>
                                                <button class="btn btn-sm btn-primary" @click="editCustomer(customer)"><i class="fas fa-edit"></i></button>
                                                <button class="btn btn-sm btn-danger" @click="deleteCustomer(customer)"><i class="fas fa-trash-alt"></i></button>
                                            </td>
                                        </tr>
                                        </tbody>
                                </table>
                                <pagination v-if="pagination.last_page >1"
                                 :pagination="pagination" 
                                 :offset="5" 
                                 @paginate="query==='' ? getCustomers() : getSearchResult()"/>
                            </div>
                            <div class="text-center" v-else>
                                <h5 class="text-danger">Sorry! No Data Found</h5>
                            </div>

                            </div>
                </div>
            </div>
        </div>
        <!-- Modal -->
        <div class="modal fade" id="customerModal" tabindex="-1" role="dialog" aria-labelledby="exampleModalCenterTitle" aria-hidden="true">
        <div class="modal-dialog modal-dialog-centered" role="document">
            <div class="modal-content">
            <div class="modal-header">
                <h5 class="modal-title" id="Title">{{editMode? 'Update' : viewMode ? 'Details '+form.name : 'Add New'}} Customer</h5>
                <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                <span aria-hidden="true">&times;</span>
                </button>
            </div>
            <form @submit.prevent="editMode ? updateCustomer() : storeCustomer()" @keydown="form.onKeydown($event)">
                <div class="modal-body">
                    <alert-error :form="form"></alert-error>
                    <div class="form-group">
                        <label>Name</label>
                        <input v-model="form.name" type="text" name="name"
                            class="form-control" :class="{ 'is-invalid': form.errors.has('name') }" :disabled="viewMode ==1">
                        <has-error :form="form" field="name"></has-error>
                    </div>
                    <div class="form-group">
                        <label>Email</label>
                        <input v-model="form.email" type="email" name="email"
                            class="form-control" :class="{ 'is-invalid': form.errors.has('email') }" :disabled="viewMode ==1">
                        <has-error :form="form" field="email"></has-error>
                    </div>
                    <div class="form-group">
                        <label>Phone</label>
                        <input v-model="form.phone" type="text" name="phone" class="form-control" :class="{ 'is-invalid': form.errors.has('phone') }" :disabled="viewMode ==1">
                        <has-error :form="form" field="phone"></has-error>
                    </div>
                    <div class="form-group">
                        <label>Address</label>
                        <textarea v-model="form.address" name="address"
                            class="form-control" :class="{ 'is-invalid': form.errors.has('address') }" :disabled="viewMode ==1"></textarea>
                        <has-error :form="form" field="address"></has-error>
                    </div>
                    <div class="form-group">
                        <label>Total</label>
                        <input v-model="form.total" type="number" name="total"
                            class="form-control" :class="{ 'is-invalid': form.errors.has('total') }" :disabled="viewMode ==1">
                        <has-error :form="form" field="total"></has-error>
                    </div>
                
                </div>
            <div class="modal-footer" v-if="!viewMode">
                <button :disabled="form.busy" type="submit" class="btn btn-primary">Save</button>
            </div>
            </form>
            </div>
        </div>
        </div>
        <vue-progress-bar></vue-progress-bar>
        <vue-snotify></vue-snotify>
    </div>
</template>

<script>
    export default {
        data(){
            return{
                editMode:false,
                viewMode:false,
                queryField:'name',
                query:'',
                customers:[],
                form: new Form({
                    id: '',
                    name: '',
                    email: '',
                    phone: '',
                    address: '',
                    total: '',
                }),
                pagination:{
                    current_page:1
                }
            }
        },

        watch:{
            query:function(newQ, oldQ){
                if(newQ===''){
                    this.getCustomers()
                }else{
                    this.getSearchResult()
                }
            },
        },
        

        methods:{

            getCustomers(){
                this.$Progress.start()
                axios.get('/api/customers?page='+this.pagination.current_page).then(response=>{
                    this.customers = response.data.data
                    this.pagination = response.data.meta
                    this.$Progress.finish()
                }).catch(e =>{
                    console.log(e)
                    this.$Progress.fail()
                })
            },

            getSearchResult(){
                this.$Progress.start()
                axios.get('/api/search/customers/'+this.queryField+'/'+this.query+'?page='+this.pagination.current_page)
                .then(response =>{
                    this.customers = response.data.data
                    this.pagination = response.data.meta
                    this.$Progress.finish()
                }).catch(e=>{
                    console.log(e)
                    this.$Progress.fail()
                })
            },

            reloadPage(){
                if(this.query !=='' || this.queryField !='name'){
                    this.$snotify.success("Successfully Page Refresh", 'Sucess')
                    this.getCustomers
                    this.query = ''
                    this.queryField = 'name'
                }
                
            },

            createCustomer(){
                this.editMode = false
                this.viewMode = false
                this.form.reset()
                this.form.clear()
                $('#customerModal').modal('show')
            },

            storeCustomer(){
                this.$Progress.start()
                this.form.busy = true
                this.form.post('/api/customers')
                    .then(response=>{
                        $('#customerModal').modal('hide')
                        this.getCustomers()
                        if(this.form.successful){
                            this.$Progress.finish()
                            this.$snotify.success("Customer Add succesfully", "success")
                        }else{
                            this.$Progress.fail()
                            this.$snotify.error("Something went Wrong.", "Error")
                        }
                        

                    }).catch(e=>{
                        console.log(e)
                        this.$Progress.fail()
                    })
            },

            editCustomer(customer){
                this.editMode = true
                this.viewMode = false
                this.form.reset()
                this.form.clear()
                console.log(customer)
                this.form.fill(customer)
                $('#customerModal').modal('show')
            },

            updateCustomer(){
                this.$Progress.start()
                this.form.busy = true
                this.form.put('/api/customers/'+this.form.id)
                    .then(response=>{
                        $('#customerModal').modal('hide')
                        this.getCustomers()
                        if(this.form.successful){
                            this.$Progress.finish()
                            this.$snotify.success("Customer Update succesfully", "success")
                        }else{
                            this.$Progress.fail()
                            this.$snotify.error("Something went Wrong.", "Error")
                        }
                        

                    }).catch(e=>{
                        console.log(e)
                        this.$Progress.fail()
                    })
            },

            deleteCustomer(customer){
                    this.$snotify.confirm("You will not be able to recover this data!", "Are you sure?", {
                    showProgressBar: false,
                    closeOnClick: false,
                    pauseOnHover: true,
                    buttons: [
                        {text: 'Yes', action: toast =>{
                            this.$snotify.remove(toast.id);
                            this.$Progress.start();
                            axios.delete('/api/customers/'+customer.id)
                            .then(response =>{
                                this.getCustomers();
                                this.$Progress.finish();
                                this.$snotify.success(
                                    "Customer Successfully Deleted",
                                    "Success"
                                );
                            }).catch(e=>{
                                console.log(e)
                                this.$Progress.fail();
                            })

                        }, bold: true},
                        {text: 'No', action: toast => {
                            this.$snotify.remove(toast.id);
                        }, bold: true}
                    ]
                    });
            },

            viewCustomer(customer){
                this.editMode = false
                this.viewMode = true
                this.form.reset()
                this.form.clear()
                this.form.fill(customer)
                $('#customerModal').modal('show')
            }

        },
        mounted() {
            
            this.getCustomers();
        }
    }
</script>
