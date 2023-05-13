<script>
import { flip } from "svelte/animate";
import Message from "./Message.svelte";

    
let editMode = false
let info = ''

$:preTasks = JSON
            .parse(localStorage.getItem('tasks')) 
            ? 
            JSON
            .parse(localStorage.getItem('tasks'))
            : []

$:ongoingTasks = preTasks.length ? preTasks.filter(t => t.isArchived !== true && t.isDeleted !== true) : []


$:archivedTasks = preTasks ? preTasks.filter(t => t.isArchived === true) : []

$:deletedTasks = preTasks ? preTasks.filter(t => t.isDeleted === true) : []




    $: {
        console.log('taks have been modified', preTasks);
       
    }

    let newTask = {}
    let editTask = {}

    let message;

    const sectionAction = (section, tasks) => {


        console.log('section mounted')


        const localStorage = window.localStorage


        const saveTasks = JSON.parse(localStorage.getItem('tasks'));

        console.log('save tasks', saveTasks)

        if(!saveTasks) {
            localStorage.setItem('tasks', JSON.stringify(tasks))
            info = 'Welcome 😁 ! Feel free to use this app anytime you want'
            setTimeout(() => {

                info = ''

            }, 3000)
        }
        else {

            const onTasks = saveTasks.filter(t => !t.isDeleted && !t.isArchived)
            info = `Welcome 😁 ! You have ${onTasks.length} on-going task(s)`
            setTimeout(() => {

                info = ''

            }, 3000)
       

        }
        
        

        return{

            destroy(){

                console.log('section element destroyed')
            }


        }
    }

    const handleAddTask = () => {


      
        console.log('handle add task')
        if(newTask.content){

            let date = new Date()
            console.log('date', date)
            preTasks.push({
                content: newTask.content,
                date: date.toString(),
                id: preTasks.length+1,
                isArchived : false,
                isDeleted: false,
                isOn: true,

            })

            info = `task with id ${preTasks.length+1} is added`
            setTimeout(() => {

                info = ''

            },2000)


            preTasks = preTasks

            localStorage.removeItem('tasks')
            localStorage.setItem('tasks', JSON.stringify(preTasks))
            newTask = {}

        }
        else {
            message = 'you need to add a task content'
        }
        

        
    }

    const handleDeleteTask = (id) => {
        

   



        console.log('handle archive task')

        const deletedTask = preTasks.filter(t => t.id === id)[0]
        const notDeletedTasks = preTasks.filter(t => t.id !== id)


        if(deletedTask.isDeleted){


            info = `task with id ${id} is fully deleted`
            setTimeout(() => {

                info = ''

            },2000)

         

            preTasks = [
                ...notDeletedTasks,
              
            ]

            preTasks = preTasks 

            localStorage.removeItem('tasks')
            localStorage.setItem('tasks', JSON.stringify(preTasks))


            return 
        }


        info = `task with id ${id} is deleted`
        setTimeout(() => {

            info = ''

        },2000)

        //modifying the task 
        deletedTask.isArchived = false
        deletedTask.isDeleted = true
        deletedTask.isOn = false

        preTasks = [
            ...notDeletedTasks,
            deletedTask
        ]

        preTasks = preTasks 

        localStorage.removeItem('tasks')
        localStorage.setItem('tasks', JSON.stringify(preTasks))

    }

    const handleEditTask = (id) => {

        info = `task with id ${id} is being edited`
        setTimeout(() => {

            info = ''

        },2000)

        console.log('handle edit task')


        if(editMode && editTask.id == id) {

            const editedTask = preTasks.filter(t => t.id === id)[0]
            const othersTasks = preTasks.filter(t => t.id !== id)
            
          

            const updatedTask = {

                ...editedTask,
                content: editTask.content
            }

            console.log('updatedTask', updatedTask)
         
            preTasks = [ ...othersTasks, updatedTask] 

            preTasks = preTasks

            localStorage.removeItem('tasks')
            localStorage.setItem('tasks', JSON.stringify(preTasks))

            editMode = false // close the modal
            editTask = {} // reset
            return
        }
        
        if(editMode && editTask.id !== id && editTask.content ){
            //someone wants to edit another task then the one they are editing
            
            info = `You want to edit another task, edit it then click on save `
            setTimeout(() => {

                info = ''

            },2000)
 
            
            editTask.content = ''
            editTask.id = id
            
            return

        }

        if(!editMode && !editTask.content && !editTask.id){

            editTask.id = id
            editMode = true

            return
        }







    }

    const  handleArchiveTask = (id) => {

        info = `task with id ${id} is archived`
        setTimeout(() => {

            info = ''

        },2000)

        console.log('handle archive task')

        const archivedTask = preTasks.filter(t => t.id === id)[0]
        const notArchivedtasks = preTasks.filter(t => t.id !== id)

        //modifying the task 
        archivedTask.isArchived = true
        archivedTask.isDeleted = false
        archivedTask.isOn = false

        preTasks = [
            ...notArchivedtasks,
            archivedTask,
        ]

        preTasks = preTasks 

        localStorage.removeItem('tasks')
        localStorage.setItem('tasks', JSON.stringify(preTasks))

    }

    const handleBackOnTask = (id) => {


        info = `task with id : ${id} is on again`
        setTimeout(() => {

            info = ''

        },2000)
        console.log('handle archive task')

        const backonTask = preTasks.filter(t => t.id === id)[0]
        const otherTasks = preTasks.filter(t => t.id !== id)

        //modifying the task 
        backonTask.isArchived = false
        backonTask.isDeleted = false
        backonTask.isOn = true

        preTasks = [
            ...otherTasks,
            backonTask,
        ]

        preTasks = preTasks 

        localStorage.removeItem('tasks')
        localStorage.setItem('tasks', JSON.stringify(preTasks))


    }



</script>




<section use:sectionAction={preTasks} class="todolist-section">
    {#if info}
        <Message {info}/>
    {/if }


    <form aria-label="The adding task div" class="add-task-container"
    
        on:submit|trusted|preventDefault={handleAddTask}
    >

        <div class="add-task-input-container" aria-label="">
            <label for="add-task"></label>
            <input  
                placeholder={message ? message : 'add a new task'}
                id="add-task" 
                type="text" 
                class="add-task-input" 
                on:input={() => console.log('before change', newTask.content)}
                bind:value={newTask.content} 
                on:input={() => console.log('after change', newTask.content)}
            />
        </div>
        <button type="submit" class="submit-button" disabled="{!newTask.content}">
            Ajouter
        </button>



    </form>


    <ul aria-label="The tasks lists div" class="list-container">
        {#if ongoingTasks.length}
        <div class="existing-task">
            <p>Ongoing task ({ongoingTasks.length})</p>
        </div>
        {/if}

        {#each ongoingTasks as task, i(task.id)}
        <li class="task-container" animate:flip="{{duration : 3000}}">
                
                {#if editMode && editTask.id === task.id} 
                <div class="task-edit-input">
                    <input type="text" placeholder={task.content} bind:value={editTask.content} class="edit-input"> 
                </div>    
                {:else} 
                <div class="task-description"> 
                {task.content}
                </div>
                {/if}
         
            <div class="task-handlers">
                <button on:click={() => handleDeleteTask(task.id)}>Delete</button>
                <button on:click={() => handleArchiveTask(task.id)}>Archive</button>
                <button on:click={() => handleEditTask(task.id)}>{editMode  && editTask.id === task.id? 'Save' : 'Edit'}</button>
            </div>
        </li>
        {:else}
        <div class="no-task">
            No ongoing task ({ongoingTasks.length})
        </div>
        {/each}

    </ul>

    <ul aria-label="Archived tasks list div" class="list-container">
        {#if archivedTasks.length}
        <div class="existing-task">
            <p>Archived task ({archivedTasks.length})</p>
        </div>
        {/if}

        {#each archivedTasks as task, i(task.id)}
        
            <li class="task-container" animate:flip="{{duration : 3000}}">
            <div class="task-description">
                {task.content}
            </div>
            <div class="task-handlers">
                <button on:click={() => handleDeleteTask(task.id)}>Delete</button>
                <button on:click={() => handleBackOnTask(task.id)}>Back on</button>
             
            </div>
            </li>
            
    
        {:else}
        <div class="no-task">
            No archived task ({archivedTasks.length})
        </div>
   

        {/each}


    </ul>


    <ul aria-label="Deleted tasks list div" class="list-container">
        {#if deletedTasks.length}
        <div class="existing-task">
            <p>Deleted task ({deletedTasks.length})</p>
        </div>
        {/if}


        {#each deletedTasks as task, i(task.id)}
        
            <li class="task-container" animate:flip="{{duration : 3000}}">
            <div class="task-description">
                {task.content}
            </div>
            <div class="task-handlers">

                <button on:click={() => handleBackOnTask(task.id)}> Back on</button>
                <button on:click={() => handleDeleteTask(task.id)}> Delete permanently </button>
             
            </div>
            </li>
            
    
        {:else}
        <div class="no-task">
            No deleted task ({deletedTasks.length})
        </div>
   

        {/each}


    </ul>

</section>


<style lang="scss">

   
    .todolist-section {
        margin: 10px;
  
    }

    .add-task-container {

        display: flex;
     
        padding: 0 10px;
        align-items: center;
        height: 100px;

        .add-task-input-container {
            display: flex;
            width: 80%;

            input {
            font-size: 18px;
            width: 100%;
            height: 50px;
            text-indent: 10px;
            &::placeholder {

                text-indent: 10px;
            }
        }
        }
      

        .submit-button {

            border:none;

            padding: 20px 20px;
            background-color: black;
            color: white;
            &:hover {
                opacity: 0.8;
                cursor: pointer;
           
            }

        }
    
    }

    .list-container {

        display: flex;
        //border: solid black 1px;
        padding: 20px;
        flex-direction: column;
        background-color: darkgrey;
        @media (max-width: 900px) {
    
            padding: 0px;
            align-items: center;
            
        }

    }

    .existing-task {
     
        display: flex;
     
        align-items: center;
        justify-content: flex-start;
        height: 100%;
   
      
        padding: 10px 0;
        p {
            margin: 0;
       
        }
    }

    .edit-input {
        height: 100%;
        border: none;
        display: flex;
        font-size: 18px;
        margin: 10px 0;
        outline: none;
        text-indent: 10px;
        width: 100%;
    }

    .no-task {


        display: flex;
     
        align-items: center;
        justify-content: center;
        height: 100%;
   
      
        padding: 10px 0;
        p {
            margin: 0;
       
        }
    }

    .task-container {
        display: flex;
        flex-direction: row;
        justify-content: space-between;
     
        background-color: white;

        margin: 20px 0;
        align-items: center;
       
        width: 100%;
        @media (max-width: 900px) {
            padding: 0;
            flex-direction: column;
            width: 97%;
        
            justify-content: flex-start;
            align-items: flex-start;
            
        }

        .task-edit-input {

            width: 50%;
            display: flex;
       
            @media (max-width: 900px) {
              
               
                width: 100%;
                
            
            }

            
        }

        .task-description {

            width: 50%;
            padding: 10px;
            @media (max-width: 900px) {
              
                padding: 10px;
                width: 100%;
                
            
            }
        }
        .task-handlers {
           
            width: 50%;
            display: flex;
            justify-content: end;
        
            @media (max-width: 900px) {
                flex-direction: row;
                width: 100%;
                justify-content: flex-start;
           
                
            
            }
            button {
                border: none;
                margin: 10px;
                padding: 10px 20px;
                background-color: gray;
                color: white;

                @media (max-width: 900px) {
                    flex-direction: row;
                    width: 100%;
                    margin: 0px;
                   
                
            
                }
                
              
                &:hover {
                    opacity: 0.8;
                    cursor: pointer;
                }

            }
        }
    }




</style>