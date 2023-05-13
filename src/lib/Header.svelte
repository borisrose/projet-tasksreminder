<script>

//action 1
const headerAction = (header) => {


    console.log('header mounted')

    return {

        destroy(){

            console.log('destroyed header')
        }
    }

}  








</script>

<header  use:headerAction>
    <h1> Tasks Reminder</h1>
</header>


<style>

    :global(body) {
        
        margin: 0;
    
        

    }

    header {
        display: flex;
        justify-content: center;
        align-items: center;
        background-color: dimgrey;
        color: white;
    }






</style>


