<template>
  <div>
    <!-- have to add click event for modal with modal method-->
    <button @click="loadCreateModal" class="btn btn-primary btn-block">Add New Book</button>
    <!-- if we have tasks then show the table -->
    <table class="table" v-if="tasks">
      <thead>
        <tr>
          <!-- <th scope="col">#</th>
      <th scope="col">Book</th>
      <th scope="col">Author</th>
          <th scope="col">Publication date</th>-->
          <th scope="col"></th>
          <th scope="col">ID</th>
          <th scope="col">Name</th>
          <th scope="col">Body</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(task,index) in tasks">
          <th scope="row"></th>
          <td>{{index + 1}}</td>
          <td>{{task.name}}</td>
          <td>{{task.body}}</td>
          <td>
            <button @click="loadUpdateModal(index)" class="btn btn-info">Edit</button>
            <td><button @click="deleteTask(index)" class="btn btn-danger">Delete</button></td>
        
        </tr>
       
      </tbody>
    </table>
    <!-- use Bootstrap Modal for Create-->
    <!-- have to create method to load modal -->
    <div class="modal fade" id="create-modal" tabindex="-1" role="dialog">
      <div class="modal-dialog" role="document">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title">Add a Book</h5>
            <button type="button" class="close" data-dismiss="modal" aria-label="Close">
              <span aria-hidden="true">&times;</span>
            </button>
          </div>
          <div class="modal-body">
            <div class="form-group">
                <div class="alert alert-danger" v-if="errors.length > 0">
                    <ul>
                        <li v-for="error in errors">{{error}}</li>
                    </ul>
                    </div>

              <label for="Name">Name</label>
              <!-- vue modal inputs on data() -- task.name -->
              <input v-model="new_update_task.name" type="text" id="name" class="form-control" />
            </div>
            <div class="form-group">
              <label for="description">Description</label>
              <!-- vue modal inputs on data method task.body -->
              <input v-model="new_update_task.body" type="text" id="description" class="form-control" />
            </div>
          </div>
          <div class="modal-footer">
            <button type="button" class="btn btn-secondary" data-dismiss="modal">Close</button>
            <button @click="updateTask" type="button" class="btn btn-primary">Save changes</button>
          </div>
        </div>
      </div>
   
   
   
    </div>
    <div class="modal fade" id="update-modal" tabindex="-1" role="dialog" aria-labelledby="exampleModalLabel" aria-hidden="true">
            <div class="modal-dialog" role="document">
                <div class="modal-content">
                    <div class="modal-header">
                        <h5 class="modal-title" id="exampleModalLabel">Update Modal</h5>
                        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                            <span aria-hidden="true">&times;</span>
                        </button>
                    </div>
                    <div class="modal-body">

                        <div class="alert alert-danger" v-if="errors.length > 0">
                            <ul>
                                <li v-for="error in errors">{{error}}</li>
                            </ul>
                        </div>

                        <div class="form-group">
                            <label for="name">Name</label>
                            <input v-model="new_update_task.name" type="text" id="name" class="form-control">
                        </div>

                        <div class="form-group">
                            <label for="description">Description</label>
                            <input v-model="new_update_task.body" type="text" id="description" class="form-control">
                        </div>

                    </div>
                    <div class="modal-footer">
                        <button type="button" class="btn btn-secondary" data-dismiss="modal">Close</button>
                        <button @click="updateTask" type="button" class="btn btn-primary">Save changes</button>
                    </div>
                </div>
            </div>
        </div>

    
  </div>
</template>

<script>
export default {
  data() {
    return {
      task: {
        name: "",
        body: ""
      },
      //properties
      tasks: [],
      uri: "http://localhost:8000/tasks/",
        errors: [],
        new_update_task: []
    };
  },

  methods: {
    loadCreateModal() {
      $("#create-modal").modal("show");
    },

    loadUpdateModal(index){
            this.errors = [];
            $("#update-modal").modal("show");
            this.new_update_task = this.tasks[index]; //assigns tasks to new_update_task
    },

    createTask() { //axios call that takes name and body inputs and
    // pushes them into tasks array,  backend controllers, then MySQL database
      axios.post(this.uri, { name: this.task.name, body: this.task.body }).then(response=> 
      {
          this.tasks.push(response.data.task);
          $("$create-modal").modal("hide")
          }).catch(error=>{

              this.errors =[];

              if(error.response.data.errors.name){
                  this.errors.push(error.response.data.errors.name[0])
              }
              if(error.response.data.errors.body){
                  this.errors.push(error.response.data.errors.body[0])
              }
          })
    },
    updateTask(){
        axios.patch(this.uri + this.new_update_task.id, {
            name: this.new_update_task.name,
            body: this.new_update_task.body,
        }).then(response => {
            $("#update-modal").modal("hide");

        }).catch(error=> {
               if(error.response.data.errors.name){
                  this.errors.push(error.response.data.errors.name[0])
              }
              if(error.response.data.errors.body){
                  this.errors.push(error.response.data.errors.body[0])
              }

        })


    },
    deleteTask(index){
                //need to confirm if user needs to delete item from list/database
                let confirmBox = confirm("Do you really want to delete this?");

                if(confirmBox == true){

                    axios.delete(this.uri + this.tasks[index].id)
                            .then(response=>{

                              this.$delete(this.tasks, index);
                               toastr.success(response.data.message);


                          }).catch(error=>{

                         console.log("Could not delete for some reason")
                    });

                }


            },
    loadTasks() {
      //axios call to dynamically use data
      axios.get(this.uri).then(response => {
        this.tasks = response.data.tasks;
        this.loading = true;
      });
    }
  },
  mounted() {
    this.loadTasks();
  }
};
</script>
