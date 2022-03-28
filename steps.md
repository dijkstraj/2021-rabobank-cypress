# 0-prep

- `$ gco demo`
- `$ gcrh 1-inputs`
- start application: `$ npm run dev`
- start cypress: `$ npx cypress open`

# 1-inputs

- `$ gcrh 1-inputs`
- WHAT: check input has focus on load
- open _cypress/integration/input-form.spec.js_
- `cy.visit('http://localhost:3030')`
- `cy.get('.new-todo').should('have.focus')`
- `cy.focused().should('have.class', 'new-todo')`
- open _src/components/TodoForm.js_
- add `autoFocus` to component
- show runner
- WHAT: check input accepts text
- open _cypress/integration/input-form.spec.js_
- `cy.get('.new-todo').type('Learn Cypress').should('have.value', 'Learn Cypress')`
- `it.only()`
- refactor: `beforeEach()`
- refactor: set "baseUrl" in cypress.json and navigate to '/'

# 2-form-submission

- `$ gcrh 2-form-submission`
- open _src/components/TodoApp.js_
- show importance of `should('have.value')` by removing `handleNewTodoChange()`
- show runner
- WHAT: check create new todo items
- open _cypress/integration/input-form.spec.js_
- `context('Form submission' () => {})`
- `it.only('Adds a new todo on submit', () => {})`
- `cy.get('.new-todo').type('Learn Cypress').type('{enter}')`
- show page actions in runner
- open _src/components/TodoApp.js_
- uncomment implementation of `handleTodoSubmit()`
- show POST 404, pin and show inspector
- open _cypress/integration/input-form.spec.js_
- add stub
    - `cy.server()`
    - `cy.route('POST', '/api/todos', {name: 'Learn Cypress', id: 1, isComplete: false})`
- `cy.get('.todo-list li').should('have.length', 1).and('contain', 'Learn Cypress')`
- `.should('have.value', '')`
- open _src/components/TodoApp.js_
- set `currentTodo` to `''` in TodoApp
- open _cypress/integration/input-form.spec.js_
- `it('Shows an error message on a failed submission', () => {})`
- `cy.server()`
- `cy.route({method: 'POST', url: '/api/todos', status: 500, response: {}})`
- `cy.get('.new-todo').type('Failure{enter}')`
- `cy.get('.todo-list li').should('not.exist')`
- `cy.get('.error').should('be.visible')`
- refactor: `cy.server()` to `beforeEach()` within context

# 3-custom-command

- `$ gcrh 3-custom-command`
- WHAT: check startup
- create file _cypress/integration/app-init.spec.js_
- `describe('App initialization', () => {})`
- `it.only('Loads todos on page load', () => {})`
- `cy.server()`
- add
  ```
  const todos = [
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
  ```
- `cy.route('GET', '/api/todos', todos)`
- `cy.visit('/')`
- `cy.get('.todo-list li').should('have.length', 4)`
- run test
- fix code in _src/components/TodoApp.js_ (uncomment code in `componentDidMount()`)
- create fixture _cypress/fixtures/todos.json_
- remove todos const
- replace todos with `'fixture:todos'`
- WHAT: create reusable command for setup
- create command for setup
    - `Cypress.Commands.add('seedAndVisit', () => {})`
- test error on load
    - `it.only('Displays an error on failure', () => {})`
    - `cy.server()`
    - `cy.route({method: 'GET', url: '/api/todos', status: 500, response: {}})`
    - `cy.visit('/')`
    - `cy.get('.todo-list li').should('not.exist')`
    - `cy.get('.error').should('be.visible')`
- run input form tests, see error, replace visit with seedAndVisit
- see that list checks fail
- add seedData parameter to seedAndVisit command

# 4-todo-items

- `$ gcrh 4-todo-items`
- WHAT: check item display and delete
- create file _cypress/integration/list-items.spec.js_
- `describe('List items', () => {})`
- `beforeEach(() => {})`
- `cy.seedAndVisit()`
- check completed
    - `it.only('Properly displays completed items', () => {})`
    - change 2nd item to completed true
    - `cy.get('.todo-list li').filter('.completed')`
    - `.should('have.length', 1).and('contain', 'Eggs')`
    - `.find('.toggle').should('be.checked')`
- check number of open items
    - `it.only('Shows remaining todos in the footer', () => {})`
    - `cy.get('.todo-count').should('contain', 3)`
    - `add {props.remaining} to Footer`
- check remove
    - `it.only('Removes a todo', () => {})`
    - `cy.route({method: 'DELETE', url: '/api/todos/1', status: 200, response: {}})`
    - `cy.get('.todo-list li').first().find('.destroy').click()`
    - run test, see fail because element not visible
    - add `{force: true}`
    - add `.invoke('show')`
    - tell that `const list = cy.get('.todo-list li')` might not work
    - `cy.get('.todo-list li').as('list')`
    - `cy.get('@list').should('have.length', 3).and('not.contain', 'Milk')`

# 5-toggle-debug

- `$ gcrh 5-toggle-debug`
- check item toggle
    - open _cypress/integration/list-items.spec.js_
    - `it.only('Marks an incomplete item complete', () => {})`
    - ```
      cy.fixture('todos')
        .then(todos => {
          const target = Cypress._.head(todos)
          cy.route('PUT', `/api/todos/${target.id}`, Cypress._.merge(target, {isComplete: true}))
        })
      ```
    - `cy.get('.todo-list li').first().as('first-todo')`
    - ```
      cy.get('@first-todo')
        .find('.toggle')
        .click()
        .should('be.checked')
      ```
    - `cy.get('@first-todo').should('have.class', 'completed')`
    - `cy.get('.todo-count').should('contain', 2)`
    - uncomment handleToggle code in _src/components/TodoApp.js_
    - add `debugger` before `setState()`
    - change todos in debugger to show tests green (`todos.splice(1, 1)`)
    - fix code in _src/components/TodoApp.js_
      - replace splat codes to `this.state.todos.map(t => t.id === data.id ? data : t)`
      - remove `targetIndex` as well

# 6-data-driven

- `$ gcrh 6-data-driven`
- WHAT: test todo/todos and filters in footer
- create file _cypress/integration/footer.spec.js_
- `describe('Footer', () => {})`
- check pluralization
    - ```
      context('with a single todo', () => {
        it.only('displays a singular todo in count', () => {
          cy.seedAndVisit([{id: 1, name: 'Learn Cypress', isComplete: false}])
          cy.get('.todo-count')
            .should('contain', '1 todo left')
        })
      })
      ```
    - ```
      context('with multiple todos', () => {
        it.only('displays plural todos in count', () => {
          cy.seedAndVisit()  
          cy.get('.todo-count')
            .should('contain', '3 todos left')
        })
      })
      ```
- check active filter
    - refactor `seedAndVisit()` to `beforeEach()`
    - ```
      it.only('Filters the active todos', () => {
        cy.contains('Active')
          .click()
        cy.get('.todo-list li')
          .should('have.length', 3)
      })
      ```
    - see url change in runner
    - fix filter
      - open _src/lib/utils.js_
      - uncomment the correct implementation
- check completed filter
    - duplicate active filter test
    - refactor to
      ```
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
      ```
    - add check for filter "all"

# 7-cucumber

- `$ gcrh 7-cucumber`
- WHAT: use Cucumber to write the first test
- open _cypress/integration/InputForm.feature_
- open _cypress/integration/InputForm/input-form.steps.js_
- show runner
- WHAT: ignore \*.js tests (i.e. only use features)
- open cypress.json
- add `"ignoreTestFiles": "*.js"`
- show runner
- WHAT: implement the tests again
- open _cypress/integration/InputForm.feature_
- ```
  Then the input has focus
  ```
- open _cypress/integration/InputForm/input-form.steps.js_
- ```
  Then(`the input has focus`, () => {
    cy.get('.new-todo').should('have.focus')
  })
  ```
- show runner
- WHAT: add the typing scenario
- open _cypress/integration/InputForm.feature_
- ```
  Scenario: Type something
    Given I am on the home page
    When I type "some words" in the input
    Then the input shows "some words"
  ```
- open _cypress/integration/InputForm/input-form.steps.js_
- ```
  When(`I type {string} in the input`, (typed) => {
    cy.get('.new-todo').type(typed)
  })
  Then(`the input shows {string}`, (showing) => {
    cy.get('.new-todo').should('have.value', showing)
  })
  ```
