# 00_prep

- start application: npm run dev
- start cypress: npx cypress open

# 01_setup

- gwco 01_setup
- WHAT: check input has focus on load
- open cypress/integration/input-form.spec.js
- cy.visit('http://localhost:3030')
- cy.get('.new-todo').should('have.focus')
- cy.focused().should('have.class', 'new-todo')
- open src/components/TodoForm.js
- add "autoFocus" to component
- show runner
- WHAT: check input accepts text
- open cypress/integration/input-form.spec.js
- cy.get('.new-todo').type('Learn Cypress').should('have.value', 'Learn Cypress')
- it.only()
- refactor: beforeEach()
- refactor: set "baseUrl" in cypress.json and navigate to '/'

# 02_inputs

- gcrh 02_inputs
- open src/components/TodoApp.js
- show importance of should('have.value') by removing handleNewTodoChange()
- WHAT: check create new todo items
- open cypress/integration/input-form.spec.js
- context('Form submission' () => {})
- it.only('Adds a new todo on submit', () => {})
- cy.get('.new-todo').type('Learn Cypress').type('{enter}')
- show page actions in runner
- open src/components/TodoApp.js
- uncomment implementation of handleTodoSubmit()
- show POST 404, pin and show inspector
- open cypress/integration/input-form.spec.js
- add stub
    - cy.server()
    - cy.route('POST', '/api/todos', {name: 'Learn Cypress', id: 1, isComplete: false})
- cy.get('.todo-list li').should('have.length', 1).and('contain', 'Learn Cypress')
- .should('have.value', '')
- open src/components/TodoApp.js
- set currentTodo to '' in TodoApp
- open cypress/integration/input-form.spec.js
- it('Shows an error message on a failed submission', () => {})
- cy.server()
- cy.route({method: 'POST', url: '/api/todos', status: 500, response: {}})
- cy.get('.new-todo').type('Failure{enter}')
- cy.get('.todo-list li').should('not.exist')
- cy.get('.error').should('be.visible')
- refactor: cy.server() to beforeEach() within context

# 03_form_sub

- gcrh 03_form_sub
- WHAT: check startup
- create file app-init.spec.js
- describe('App initialization', () => {})
- it.only('Loads todos on page load', () => {})
- cy.server()
- add const todos = [
  {
    "id": 1,
    "name": "Buy Milk",
    "isComplete": false
  },
  {
    "id": 2,
    "name": "Buy Eggs",
    "isComplete": false
  },
  {
    "id": 3,
    "name": "Buy Bread",
    "isComplete": false
  },
  {
    "id": 4,
    "name": "Make French Toast",
    "isComplete": false
  }
]
- cy.route('GET', '/api/todos', todos)
- cy.visit('/')
- cy.get('.todo-list li').should('have.length', 4)
- run test
- fix code in TodoApp
- create fixture todos.json
- remove todos const
- replace todos with 'fixture:todos'
- create command for setup
    - Cypress.Commands.add('seedAndVisit', () => {})
- test error on load
    - it.only('Displays an error on failure', () => {})
    - cy.server()
    - cy.route({method: 'GET', url: '/api/todos', status: 500, response: {}})
    - cy.visit('/')
    - cy.get('.todo-list li').should('not.exist')
    - cy.get('.error').should('be.visible')
- run input form tests, see error, replace visit with seedAndVisit
- see that list checks fail
- add seedData parameter to seedAndVisit command

# 04_custom_cmd

- gcrh 04_custom_cmd
- WHAT: check item display and delete
- create file list-items.spec.js
- describe('List items', () => {})
- beforeEach(() => {})
- cy.seedAndVisit()
- check completed
    - it.only('Properly displays completed items', () => {})
    - change 2nd item to completed true
    - cy.get('.todo-list li').filter('.completed')
    - .should('have.length', 1).and('contain', 'Eggs')
    - .find('.toggle').should('be.checked')
- check number of open items
    - it.only('Shows remaining todos in the footer', () => {})
    - cy.get('.todo-count').should('contain', 3)
    - add {props.remaining} to Footer
- check remove
    - it.only('Removes a todo', () => {})
    - cy.route({method: 'DELETE', url: '/api/todos/1', status: 200, response: {}})
    - cy.get('.todo-list li').first().find('.destroy').click()
    - run test, see fail because element not visible
    - add {force: true}
    - add .invoke('show')
    - tell that const list = cy.get('.todo-list li') might not work
    - cy.get('.todo-list li').as('list')
    - cy.get('@list').should('have.length', 3).and('not.contain', 'Milk')

# 05_todo_items

- gcrh 05_todo_items
- check item toggle
    - open cypress/integration/list-items.spec.js
    - it.only('Marks an incomplete item complete', () => {})
    - cy.fixture('todos')
          .then(todos => {
            const target = Cypress._.head(todos)
            cy.route('PUT', `/api/todos/${target.id}`, Cypress._.merge(target, {isComplete: true}))
          })
    - cy.get('.todo-list li').first().as('first-todo')
    - cy.get('@first-todo')
          .find('.toggle')
          .click()
          .should('be.checked')
    - cy.get('@first-todo').should('have.class', 'completed')
    - cy.get('.todo-count').should('contain', 2)
    - uncomment handleToggle code
    - add debugger
    - change todos in debugger to show tests green
    - fix code
    - replace splat codes to this.state.todos.map(t => t.id === data.id ? data : t)
    - remove targetIndex as well

# 06_toggle_debug

- gcrh 06_toggle_debug
- WHAT: test todo/todos and filters in footer
- create file footer.spec.js
- describe('Footer', () => {})
- check pluralization
    - context('with a single todo', () => {
        it.only('displays a singular todo in count', () => {
          cy.seedAndVisit([{id: 1, name: 'Learn Cypress', isComplete: false}])
          cy.get('.todo-count')
            .should('contain', '1 todo left')
        })
      })
    - context('with multiple todos', () => {
        it.only('displays plural todos in count', () => {
          cy.seedAndVisit()  
          cy.get('.todo-count')
            .should('contain', '3 todos left')
        })
      })
- check active filter
    - refactor seedAndVisit to beforeEach
    - it.only('Filters the active todos', () => {
          cy.contains('Active')
            .click()

          cy.get('.todo-list li')
            .should('have.length', 3)
        })
    - see url change
    - fix filter
- check completed filter
    - duplicate active filter test
    - refactor to
          it.only('Handles filter links', () => {
            const filters = [
              {link: 'Active', expectedLength: 3},
              {link: 'Completed', expectedLength: 1}
            ]

            cy.wrap(filters)
              .each(filter => {
                cy.contains(filter.link)
                  .click()

                cy.get('.todo-list li')
                  .should('have.length', filter.expectedLength)
              })
          })
    - add check for filter "all"