# README

## Demo

- Setup Rails 5.1 with Redux
- Extend API with new endpoint
- Add new redux elements
- Further readings

## Requirements

- Ensure you have yarn > 0.20.0
```
brew update
brew install yarn

And add export PATH="$PATH:$HOME/.yarn/bin" to your profile (this may be in your .profile, .bashrc, .zshrc, etc.)

yarn --version #check version to make sure installed and in path
```

## Setup

``` bash
rvm use 2.4.1@TodoRails51ReactV1 #--create
git init
gem install bundler
gem install rails -v='5.1.2'
rails new todos --webpack=react
cd todos
yarn add --dev jest enzyme babel-preset-stage-2 react-addons-test-utils redux-mock-store
echo "{\n  \"presets\": [\"es2015\", \"react\", \"stage-2\"]\n}\n" > .babelrc
yarn add react react-dom babel-preset-react babel-preset-es2015 react-redux redux-undo redux
echo "rails: bin/rails s\nwebpack: ./bin/webpack-dev-server" > Procfile
```

In `config/environments/development.rb` add webpack dev server config:

```
config.x.webpacker[:dev_server_host] = "http://localhost:8080"
```

Add `gem 'foreman'` to the `:development` group in the `Gemfile` and

``` bash
bundle
```

In `package.json` add the test script and configuration

``` json
  "scripts": {
    "test": "NODE_PATH='./node_modules:../app/javascript' jest --watch --coverage"
  },
  "jest": {
    "roots": [
      "app/javascript"
    ]
  },
```

Add `/coverage` to `.gitignore`

You can now start the application with

``` bash
foreman start
```

## Rails application code

Create the Todos controller `app/controllers/todos_controller.rb` with

``` ruby
class TodosController < ApplicationController
  def index
  end
end
```

Create the Todos view `app/views/todos/index.html.erb` with

``` erb
<h1>Todos</h1>

<div id='todos'></div>
```

Route to the Todos controller by default by adding the default route to `config/routes.rb`

``` ruby
root to: 'todos#index'
```

## React and Redux client-side code

### Components

![Client side application](https://www.evernote.com/shard/s24/sh/11b1439c-342a-4922-b2e7-28857b1b923e/ec9f335dcd59e5b1/res/356a1ecb-32bb-4ce3-ab45-507067254fed/skitch.png)

Create a Todo item component in `app/javascript/components/todos/Todo.js` with

``` js
import React, { PropTypes } from 'react'

const Todo = ({ onClick, completed, text }) => (
  <li
    onClick={onClick}
    style={{
      textDecoration: completed ? 'line-through' : 'none'
    }}
  >
    {text}
  </li>
)

Todo.propTypes = {
  onClick: PropTypes.func.isRequired,
  completed: PropTypes.bool.isRequired,
  text: PropTypes.string.isRequired
}

export default Todo;
```

Create a Todo list component made of Todos in `app/javascript/components/todos/TodoList.js` with
``` js
import React, { PropTypes } from 'react'
import Todo from './Todo'

const TodoList = ({ todos, onTodoClick }) => (
  <ul>
    {todos.map(todo =>
      <Todo
        key={todo.id}
        {...todo}
        onClick={() => onTodoClick(todo.id)}
      />
    )}
  </ul>
)

TodoList.propTypes = {
  todos: PropTypes.arrayOf(
    PropTypes.shape({
      id: PropTypes.number.isRequired,
      completed: PropTypes.bool.isRequired,
      text: PropTypes.string.isRequired
    }).isRequired
  ).isRequired,
  onTodoClick: PropTypes.func.isRequired
}

export default TodoList;
```

Create an Add Todo form in `app/javascript/components/todos/AddTodoForm.js` with

``` js
import React, { PropTypes } from 'react'

const onSubmit = (event, input, action) => {
  event.preventDefault()
  if (!input.value.trim()) {
    return
  }
  action(input.value);
  input.value = ''
}

const AddTodoForm = ({ addTodo }) => {
  let input;
  return(
  <div>
    <form onSubmit={event => onSubmit(event, input, addTodo)}>
      <input ref={node => {input = node}} />
      <button type="submit">
        Add Todo
      </button>
    </form>
  </div>
  );
}

AddTodoForm.propTypes = {
  addTodo: PropTypes.func.isRequired
}

export default AddTodoForm
```

Create a Link component in `app/javascript/components/Link.js` with

``` js
import React, { PropTypes } from 'react'

const Link = ({ active, children, onClick }) => {
  if (active) {
    return <span>{children}</span>
  }

  return (
    <a href="#"
       onClick={e => {
         e.preventDefault()
         onClick()
       }}
    >
      {children}
    </a>
  )
}

Link.propTypes = {
  active: PropTypes.bool.isRequired,
  children: PropTypes.node.isRequired,
  onClick: PropTypes.func.isRequired
}

export default Link
```

Create a filters component using the Link component in `app/javascript/components/todos/TodoFilters.js` with

``` js
import React from 'react'
import FilterLink from '../../containers/todos/FilterLinkContainer'

const TodoFilters = () => (
  <p>
    {"Filter: "}
    <FilterLink filter="SHOW_ALL">
      All
    </FilterLink>
    {" | "}
    <FilterLink filter="SHOW_ACTIVE">
      Active
    </FilterLink>
    {" | "}
    <FilterLink filter="SHOW_COMPLETED">
      Completed
    </FilterLink>
  </p>
)

export default TodoFilters;
```

Create Undo/Redo button components in `app/javascript/components/UndoRedoButtons.js` with

``` js
import React from 'react';

const UndoRedoButtons = ({ canUndo, canRedo, onUndo, onRedo }) => (
  <p>
    <button onClick={onUndo} disabled={!canUndo}>
      Undo
    </button>
    <button onClick={onRedo} disabled={!canRedo}>
      Redo
    </button>
  </p>
)

export default UndoRedoButtons;
```

Create a Todo screen in `app/javascript/components/todos/TodoScreen.js` with

``` js
import React from 'react'
import TodoFilters from './TodoFilters'
import AddTodoForm from '../../containers/todos/AddTodoFormContainer'
import VisibleTodoList from '../../containers/todos/VisibleTodoList'
import UndoRedo from '../../containers/UndoRedo'

const TodoScreen = () => (
  <div>
    <AddTodoForm />
    <VisibleTodoList />
    <TodoFilters />
    <UndoRedo />
  </div>
)

export default TodoScreen
```

### Redux

Create the main reducer in `app/javascript/reducers/index.js` with

``` js
import { combineReducers } from 'redux'
import todos from './todos'
import visibilityFilter from './visibilityFilter'

const todoApp = combineReducers({
  todos,
  visibilityFilter
})

export default todoApp
```

Create Todo Redux actions in `app/javascript/actions/todos.js` with

``` js
let nextTodoId = 0
const addTodo = (text) => ({
  type: 'ADD_TODO',
  id: nextTodoId++,
  text
})

const setVisibilityFilter = (filter) => ({
  type: 'SET_VISIBILITY_FILTER',
  filter
})

const toggleTodo = (id) => ({
  type: 'TOGGLE_TODO',
  id
})

module.exports = {
 addTodo: addTodo,
  setVisibilityFilter: setVisibilityFilter,
 toggleTodo: toggleTodo
}
```

Create the Todos reducer in `app/javascript/reducers/todos.js` with

``` js
import undoable, { distinctState } from 'redux-undo'

export const todo = (state, action) => {
  switch (action.type) {
    case 'ADD_TODO':
      return {
        id: action.id,
        text: action.text,
        completed: false
      }
    case 'TOGGLE_TODO':
      if (state.id !== action.id) {
        return state
      }

      return {
        ...state,
        completed: !state.completed
      }
    default:
      return state
  }
}

export const todos = (state = [], action) => {
  switch (action.type) {
    case 'ADD_TODO':
      return [
        ...state,
        todo(undefined, action)
      ]
    case 'TOGGLE_TODO':
      return state.map(t => todo(t, action))
    default:
      return state
  }
}

const undoableTodos = undoable(todos, {
  filter: distinctState()
})

export default undoableTodos
```

Create the visibility reducer in `app/javascript/reducers/visibilityFilter.js` with

``` js
const visibilityFilter = (state = 'SHOW_ALL', action) => {
  switch (action.type) {
    case 'SET_VISIBILITY_FILTER':
      return action.filter
    default:
      return state
  }
}

export default visibilityFilter
```

### Containers

Create an add Todo form container in `app/javascript/containers/todos/AddTodoFormContainer.js` with

``` js
import { connect } from 'react-redux';
import AddTodoForm from '../../components/todos/AddTodoForm'
import actions from '../../actions/todos';

export const mapDispatchToProps = (dispatch) => ({
  addTodo: (value) => dispatch(actions.addTodo(value))
})

const AddTodoFormContainer = connect(null, mapDispatchToProps)(AddTodoForm)

export default AddTodoFormContainer
```

Create a visible Todo list container in `app/javascript/containers/todos/VisibleTodoList.js` with

``` js
// Container components connect the presentational components to Redux

import { connect } from 'react-redux'
import { toggleTodo } from '../../actions/todos'
import TodoList from '../../components/todos/TodoList'

// returns an array of todos for the corresponding filter
const getVisibleTodos = (todos, filter) => {
  switch (filter) {
    case 'SHOW_ALL':
      return todos
    case 'SHOW_COMPLETED':
      return todos.filter(t => t.completed)
    case 'SHOW_ACTIVE':
      return todos.filter(t => !t.completed)
    default:
      throw new Error('Unknown filter: ' + filter)
  }
}

// mapStateToProps defines how to transform the current Redux store
// state into the props you want to pass to the presentational component
const mapStateToProps = (state) => ({
  todos: getVisibleTodos(state.todos.present, state.visibilityFilter)
})

const mapDispatchToProps = (dispatch) => ({
  onTodoClick: (id) => {
    dispatch(toggleTodo(id))
  }
})

// A container component is just a React component that uses store.subscribe()
// to read a part of the Redux state tree and supply props to a
// presentational component it renders
const VisibleTodoList = connect(
  mapStateToProps,
  mapDispatchToProps
)(TodoList)
// Note that TodoList component takes two props:
// The todos array and the onClick function

export default VisibleTodoList
```

Create a filter link container in `app/javascript/containers/todos/FilterLinkContainer.js` with

``` js
import { connect } from 'react-redux'
import { setVisibilityFilter } from '../../actions/todos'
import Link from '../../components/Link'

const mapStateToProps = (state, ownProps) => ({
  active: ownProps.filter === state.visibilityFilter
})

const mapDispatchToProps = (dispatch, ownProps) => ({
  onClick: () => {
    dispatch(setVisibilityFilter(ownProps.filter))
  }
})

const FilterLink = connect(
  mapStateToProps,
  mapDispatchToProps
)(Link)

export default FilterLink
```

Create a Undo/Redo container in `app/javascript/containers/UndoRedo.js` with

``` js
import { ActionCreators as UndoActionCreators } from 'redux-undo'
import { connect } from 'react-redux'
import UndoRedoButtons from '../components/UndoRedoButtons'

const mapStateToProps = (state) => ({
  canUndo: state.todos.past.length > 0,
  canRedo: state.todos.future.length > 0
})

const mapDispatchToProps = ({
  onUndo: UndoActionCreators.undo,
  onRedo: UndoActionCreators.redo
})

const UndoRedo = connect(
  mapStateToProps,
  mapDispatchToProps
)(UndoRedoButtons)

export default UndoRedo
```

## Hooking it all up

Create the Todo pack in `app/javascript/packs/todos.js`

``` js
import React from 'react'
import { render } from 'react-dom'
import { createStore } from 'redux';
import reducer from '../reducers';
import { Provider } from 'react-redux';
import TodoScreen from '../components/todos/TodoScreen';

const store = createStore(reducer)

document.addEventListener("DOMContentLoaded", e => {
  render((
    <Provider store={store}>
      <TodoScreen />
    </Provider>
    ),
    document.getElementById('todos')
  )
})
```

Add the package to the view.

``` erb
<%= javascript_pack_tag 'todos' %>
```
## Add some style
- Add your style file `style.scss` under the app/assets/stylesheets directory
- Add your sass:
```

* { -moz-box-sizing: border-box; -webkit-box-sizing: border-box; box-sizing: border-box; }
.clr {clear:both;}

/* Variables & Mixins */
$bg: #123;  /* Changes Entire Color Scheme */
$text: lighten($bg,35%);
$link-text: lighten($bg,75%);

a {
  color: $link-text;
}
/* General Layout */
body {
  background: $bg; 
  color:$text; 
  font-family:Century Gothic;
}


```
**NOTE**: I skipped testing. Have a look at [this commit](https://github.com/paulsturgess/todos-5.1.0/commit/979af2ac4a762bd3eb712a1dc6d602a811417c8d) to see how tests were implemented for all this React and Redux code.

# TODO

- [x] Finish initial app set up
- [ ] Add a Todo model
- [ ] Add a Todo controller
- [ ] Update client to load and save data
- [x] Add more links
- [x] Figure out why there was no .babelrc or webpack scripts in bin/, or indeed react, react-dom, and so on

# Useful Links:

- [Original project: Paul Sturgess Todos 5.1.0](https://github.com/paulsturgess/todos-5.1.0)

## Rails 5.1:

- [Rails 5.1: Loving JavaScript, System Tests, Encrypted Secrets, and more](http://weblog.rubyonrails.org/2017/4/27/Rails-5-1-final/)
- [Rails 5.1 Loves JavaScript](https://medium.com/@hpux/rails-5-1-loves-javascript-a1d84d5318b)
- [Introducing Webpacker](https://medium.com/statuscode/introducing-webpacker-7136d66cddfb)

## Redux:

- [A cartoon intro to redux](https://code-cartoons.com/a-cartoon-intro-to-redux-3afb775501a6)
- [An introduction to redux](https://www.smashingmagazine.com/2016/06/an-introduction-to-redux/)
