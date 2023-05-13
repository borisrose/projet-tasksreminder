<script>
import { flip } from "svelte/animate";

    


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

    let message;

    const sectionAction = (section, tasks) => {

        console.log('section mounted')


        const localStorage = window.localStorage


        const saveTasks = JSON.parse(localStorage.getItem('tasks'));

        console.log('save tasks', saveTasks)

        if(!saveTasks) {
            localStorage.setItem('tasks', JSON.stringify(tasks))
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

        console.log('handle edit task')






    }

    const  handleArchiveTask = (id) => {


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
            <div class="task-description">
                {task.content}
            </div>
            <div class="task-handlers">
                <button on:click={() => handleDeleteTask(task.id)}>Supprimer</button>
                <button on:click={() => handleArchiveTask(task.id)}>Archiver</button>
                <button on:click={() => handleEditTask(task.id)}>Modifier</button>
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
                <button on:click={() => handleDeleteTask(task.id)}>Supprimer</button>
                <button on:click={() => handleArchiveTask(task.id)}>Désarchiver</button>
             
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

                <button on:click={() => handleBackOnTask(task.id)}> Réactualiser </button>
             
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
        }
        input {
            font-size: 18px;
            width: 100%;
            height: 50px;
            text-indent: 10px;
            &::placeholder {

                text-indent: 10px;
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
   
        width: 100%;
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