# yaml-workflow
A workflow specification defined in YAML language. The first implementation is the lightbot in Pythond language. 

## Here is the flow with simple steps. 

```
workflow:
  name: "My Automation Workflow"
  description: "A workflow to automate tasks on a website and analyze results"

  steps:
    - id: "step_1"
      name: "Navigate to the homepage"
      type: "browser_action"
      action: "navigate"
      url: "https://example.com"

    - id: "step_2"
      name: "Enter username and password"
      type: "browser_action"
      action: "enter_text"
      selector: "#username"
      text: "my_username"
    - id: "step_3"
      type: "browser_action"
      action: "enter_text"
      selector: "#password"
      text: "my_password"

    - id: "step_4"
      name: "Click the Login Button"
      type: "browser_action"
      action: "click"
      selector: "#login-button"


    - id: "step_5"
      name: "Wait for dashboard to load"
      type: "browser_action"
      action: "wait"
      selector: "#dashboard"
      timeout: 10

    - id: "step_6"
      name: "Click the download report"
      type: "browser_action"
      action: "click"
      selector: "#download-report"

    - id: "step_7"
      name: "Wait for download to complete"
      type: "browser_action"
       action: "wait"
      selector: "#download-completed"
      timeout: 30

    - id: "step_8"
      name: "Execute system command to check download"
      type: "system_command"
      command: "ls -l ~/Downloads/"

    - id: "step_9"
      name: "Analyze command output with Gemini"
      type: "gemini_analyze"
      input: "{{ step_8.output }}" # Using Jinja like template expression to access output of step 8
      prompt: "Analyze the output from the command and check if the download file is present in the directory. If it's there respond with 'true', otherwise 'false'"
      output_path : "step_9_gemini.txt"

    - id: "step_10"
       name : "If downloaded successfully"
       type: "condition"
       condition: "{{ step_9.output == 'true'}}"
       then :
          - id: "step_11"
              name: "Move download file to the reports directory"
              type: "system_command"
              command: "mv ~/Downloads/report.txt ~/reports/"
       else :
           - id: "step_12"
              name: "Send Notification of failed download"
              type: "notification"
              message: "Download Failed"
```

#### Explanation:

* workflow: The top-level element, containing the name and description of the workflow.

* steps: A list of steps to execute sequentially.

  * id: A unique identifier for each step.

  * name: A human-readable name for the step.

  * type: The type of action to perform:

	* browser_action: For actions in the browser.

    * system_command: For running terminal commands.

    * gemini_analyze: For analyzing output using Gemini.

    * condition: To apply conditional logic to the workflow execution.

    * notification: Send a message using a specific notification system

  * action: (For browser_action) Specifies the specific browser action (e.g., navigate, click, enter_text).

  * url, selector, text, timeout: Parameters specific to the action.

  * command: (For system_command) The command string to execute.

  * input (For gemini_analyze) : The input data to analyze by Gemini, can be a Jinja like template to access variables.

  * prompt: (For gemini_analyze) Prompt to send to Gemini to analyze the input.

  * output_path: (For gemini_analyze) Where to store the output of Gemini response.

  * condition: (For condition type). Jinja like expression to evaluate.

  * then: (For condition type). List of actions to execute if condition is true.

  * else: (For condition type). List of actions to execute if condition is false.

  * message: (For notification type). Message to send.

#### Key Features of This Design:

* Modularity: Each step is a discrete unit with a clear action and set of parameters.

* Extensibility: Easy to add new type for other functionalities.

* Template Variables: Jinja-like template variables allow sharing data between steps (like the output of a system command passed to Gemini).

* Conditional Execution: Allows you to dynamically adjust workflow based on Gemini response.

* Easy to Parse: YAML is easy to read and parse using libraries such as PyYAML.


#### Alternative Workflow Languages (and Why YAML is a Good Choice):

1. JSON:

  * Pros: Ubiquitous, well-supported.

  * Cons: Less human-readable than YAML, especially with nested structures.

2. TOML:

  * Pros: Human-readable, has a more structured approach to configuration than YAML.

  * Cons: Not as widely adopted as YAML or JSON.

3. Custom DSL:

  * Pros: You can design your own workflow language specific to your task.

  * Cons: Requires additional time and effort to design, implement and maintain.

#### Why YAML is Good for this Project:

* Readability: YAML's emphasis on readability helps you quickly understand and modify workflows.

* Human-Editable: The YAML files can be easily modified by developers.

* Python-Friendly: Python has excellent libraries for YAML parsing, and also very easy to process the output from YAML files.

* Flexibility: YAML is flexible and can support complex data structures.

Popular Choice: Widely used for configuration files and defining workflow in various tools.


## Loop Constructs in YAML


We can model loops in the YAML workflow by introducing new step types, for_loop , while_loop, and do_while_loop .

Here's how they can be structured:

```
workflow:
  name: "Workflow with Loops"
  description: "Demonstrates loops and conditional steps"

  steps:
    - id: "step_1"
      name: "Set initial value"
      type: "set_variable"
      variable: "counter"
      value: 0

    - id: "loop_1"
      name: "For loop example"
      type: "for_loop"
      variable: "i"
      start: 0
      end: 3  # Executes from 0 up to 2
      increment: 1
      steps:
        - id: "loop_step_1"
          name: "Print Loop Counter"
          type: "system_command"
          command: "echo 'Loop iteration: {{ i }} ; counter: {{ counter }}'"
        - id: "loop_step_2"
            name: "Increment counter variable"
            type: "increment_variable"
            variable: "counter"
            amount: 1

    - id: "step_2"
      name: "Set initial value for while loop"
      type: "set_variable"
      variable: "while_counter"
      value: 0

    - id: "loop_2"
      name: "While loop example"
      type: "while_loop"
      condition: "{{ while_counter < 3 }}"
      steps:
        - id: "loop_step_3"
            name : "Print while loop counter"
            type: "system_command"
            command: "echo 'while loop iteration : {{ while_counter }}'"
        - id: "loop_step_4"
           name: "Increment while counter variable"
           type: "increment_variable"
           variable: "while_counter"
           amount: 1


    - id: "step_3"
      name: "Set initial value for do-while loop"
      type: "set_variable"
      variable: "do_while_counter"
      value: 0

    - id: "loop_3"
      name: "do-while loop example"
      type: "do_while_loop"
      condition: "{{ do_while_counter < 3 }}"
      steps:
        - id: "loop_step_5"
            name : "Print do-while loop counter"
            type: "system_command"
            command: "echo 'do-while loop iteration : {{ do_while_counter }}'"
        - id: "loop_step_6"
           name: "Increment do-while counter variable"
           type: "increment_variable"
           variable: "do_while_counter"
           amount: 1
    - id: "step_4"
      name: "Final Message"
      type: "system_command"
      command: "echo 'Finished all loops!'"
```

#### Explanation of Loop Types:

* for_loop:

  * variable: Loop variable.

  * start: Initial value for the loop variable.

  * end: End value, loop will run until just before the end.

  * increment: Increment value to be added after each loop.

  * steps: A list of steps that run for each iteration of the for loop.

* while_loop:

  * condition: A Jinja-like template expression to evaluate the looping condition. The loop runs until this expression is false.

  * steps: A list of steps that run as long as the condition evaluates to true.

* do_while_loop:

  * condition: A Jinja-like template expression to evaluate the looping condition. The loop runs until this expression is false, but executes the steps at least once.

  * steps: A list of steps that run at least once, then as long as the condition evaluates to true.

* set_variable:

  * variable: variable name to create/update

  * value: variable value

* increment_variable:

  * variable: variable to increment

  * amount: amount to increment

## Goto Label

We can emulate a simple goto (jumping to specific points in the workflow) by using a goto action and labels

```
workflow:
  name: "Workflow with Goto"
  description: "Demonstrates the usage of goto functionality"
  steps:
    - id: "start"
      name: "Start of the workflow"
      type: "system_command"
      command: "echo 'Start Workflow'"
    - id: "step_1"
      name: "First Step"
      type: "system_command"
      command: "echo 'Step 1 executed'"
    - id: "condition_1"
        type: "condition"
        condition: "{{ counter < 3}}" # need to create the counter before using this step
        then :
          - id: "goto_label_1"
            type: "goto"
            target: "label_1"
        else :
            - id: "step_2"
              name: "Else part"
              type: "system_command"
              command: "echo 'Step 2 Executed'"
    - id: "label_1"
      name: "Label 1"
      type: "label"
    - id: "step_3"
      name: "Third Step"
      type: "system_command"
      command: "echo 'Step 3 executed after goto'"
```  

* goto Step:

  * type: Specifies the type as "goto".

  * target: specifies label name to which the workflow execution should jump.

* label step

  * type: Specifies the type as "label".

  * id : id acts as the label name.


## List-Based Branching (Parallel Processing of Items)

let's tackle how to model branching in your YAML workflow language, including the three scenarios you've described:

* List-Based Branching: An action outputs a list, and each item in the list spawns a branch.

* AND Branches: All branches must complete before moving to the next step.

* OR Branches: Only one branch needs to complete to move to the next step.

1. List-Based Branching (Parallel Processing of Items)

let's tackle how to model branching in your YAML workflow language, including the three scenarios you've described:

List-Based Branching: An action outputs a list, and each item in the list spawns a branch.

AND Branches: All branches must complete before moving to the next step.

OR Branches: Only one branch needs to complete to move to the next step.

1. List-Based Branching (Parallel Processing of Items)


We'll use a parallel_for_each type to handle this scenario. The output of a previous step can be used as the list, and actions are performed on each list item.

```
workflow:
  name: "Workflow with List-Based Branching"
  description: "Demonstrates parallel processing of a list"
  steps:
    - id: "step_1"
      name: "Get User IDs"
      type: "system_command"
      command: "echo 'user1,user2,user3'"

    - id: "branch_1"
      name: "Process each user"
      type: "parallel_for_each"
      list_source: "{{step_1.output.split(',')}}"
      item_variable: "user_id"
      steps:
        - id: "branch_step_1"
            name: "send email to each user"
            type: "notification"
            message: "Sending email to user {{ user_id }}"


    - id: "step_2"
      name: "All emails sent"
      type: "system_command"
      command: "echo 'Email for all users have been sent!'"
```

* parallel_for_each: This step will create branches from the data from list_source.

  * list_source: A Jinja-like template expression that provides list to iterate over. In our case, this is split string of output from previous step.

  * item_variable: Name of the loop variable to access the item inside the list.

  * steps: Actions to be executed in each branch



##  AND/OR Branches (Parallel Execution and Synchronization)


To implement AND/OR branches, we will use parallel_and_branch, and parallel_or_branch.

```
workflow:
  name: "Workflow with AND/OR Branching"
  description: "Demonstrates AND/OR branches with parallel execution"

  steps:
    - id: "start"
      name: "Start of the workflow"
      type: "system_command"
      command: "echo 'Start Workflow'"

    - id: "and_branch_1"
        name: "AND Branch execution"
        type: "parallel_and_branch"
        branches :
           -  id : "branch_a"
              steps :
                - id: "and_branch_step_1"
                   name: "AND Branch step 1"
                   type: "system_command"
                   command: "sleep 2; echo 'AND branch step 1 executed'"
           -  id : "branch_b"
               steps :
                 - id: "and_branch_step_2"
                   name: "AND Branch Step 2"
                   type: "system_command"
                   command: "sleep 1; echo 'AND Branch Step 2 executed'"


    - id: "or_branch_1"
        name: "OR Branch Execution"
        type: "parallel_or_branch"
        branches :
           -  id : "branch_c"
              steps :
                - id: "or_branch_step_1"
                   name: "OR branch step 1"
                   type: "system_command"
                   command: "sleep 4; echo 'OR branch step 1 executed'"
           -  id : "branch_d"
               steps :
                 - id: "or_branch_step_2"
                   name: "OR Branch Step 2"
                   type: "system_command"
                   command: "sleep 1; echo 'OR Branch Step 2 executed'"

    - id : "end"
      name : "End workflow"
      type : "system_command"
      command: "echo 'Workflow ended'"
```

* parallel_and_branch: Executes branches in parallel and waits until all branches finish.

  * branches: A list of branches to execute.

    * id: unique id of the branch

    * steps: list of actions that has to be executed in that branch.

* parallel_or_branch: Executes branches in parallel but will move on if any of the branch completes.

  * branches: A list of branches to execute.

  * id: unique id of the branch.

    * steps: list of actions that has to be executed in that branch.


Important Considerations:

* Thread Pool Size: The max_workers value (set to 10 in the code) should be adjusted based on your available system resources and how many parallel branches you need.

* Error Handling: The code provides basic error handling. Improve the error reporting, especially on failure of branches, to make your code robust.

* Concurrency Issues: While the code provided uses thread pool executor, if two branches try to update a variable in shared context, then that can cause issues, so make sure to handle it in your design.

* Branch Communication: If you need to transfer data from one branch to another, you should implement that feature explicitly.

## split_list

We'll introduce a new step type called split_list that creates the groups.

```
workflow:
  name: "Workflow with List Splitting and Parallel Processing"
  description: "Demonstrates list splitting and parallel processing"

  steps:
    - id: "step_1"
      name: "Get a list of items"
      type: "system_command"
      command: "echo 'item1,item2,item3,item4,item5,item6,item7,item8,item9,item10'"

    - id: "split_1"
      name: "Split the list into groups"
      type: "split_list"
      list_source: "{{ step_1.output.split(',') }}"
      groups:
        group_a: 30
        group_b: 70

    - id: "process_groups"
      name: "Process each group"
      type: "parallel_for_each"
      list_source: "{{ split_1.output_groups }}"
      item_variable: "group"
      steps:
        - id: "branch_step_1"
          name: "Log the group id and items"
          type: "system_command"
          command: "echo 'Group: {{ group.id }}, Items: {{ group.items }}'"

        - id: "branch_step_2"
          name: "Process each item of a group"
          type: "parallel_for_each"
          list_source: "{{ group.items }}"
          item_variable: "item"
          steps:
           - id: "branch_step_3"
               name: "Process an item"
               type: "notification"
               message: "Processing item {{item}}"

    - id: "step_2"
      name: "All Items Processed"
      type: "system_command"
      command: "echo 'All items have been processed'"
```

split_list Step:

* list_source: The list that has to be split

* groups: This is a mapping with the group id as key and percentage as value.

