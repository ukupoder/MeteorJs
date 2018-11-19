# Installimine

## OSX/Linux
`curl https://install.meteor.com/ | sh`



## Windows
```
choco install meteor

@"%SystemRoot%\System32\WindowsPowerShell\v1.0\powershell.exe" -NoProfile -InputFormat None -ExecutionPolicy Bypass -Command "iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))" && SET "PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin"

choco upgrade chocolatey
```

# Rakenduse tegemine

`meteor create simple-todos`

# Rakenduse käivitamine

`meteor`


# Todo tegemine

Kuvamiseks kasutame Reacti, et seda teha, siis installimiseks
```
meteor npm install --save react react-dom
```


**client/main.html**
```
<body>
  <div id="render-target"></div>
</body>
```


**client/main.js**
```
import React from 'react';
import { Meteor } from 'meteor/meteor';
import { render } from 'react-dom';

import './main.html';

import App from '../imports/ui/App.js';
 
Meteor.startup(() => {
  render(<App />, document.getElementById('render-target'));
});
```


Loo kaust imports/ui ja sinna sisse failid App.js ja Task.js

**imports/ui/App.js**
```
import React, { Component } from 'react';
 
import Task from './Task.js';
 
export default class App extends Component {
  getTasks() {
    return [
      { _id: 1, text: 'This is task 1' },
      { _id: 2, text: 'This is task 2' },
      { _id: 3, text: 'This is task 3' },
    ];
  }
 
  renderTasks() {
    return this.getTasks().map((task) => (
      <Task key={task._id} task={task} />
    ));
  }
 
  render() {
    return (
      <div className="container">
        <header>
          <h1>Todo List</h1>
        </header>
 
        <ul>
          {this.renderTasks()}
        </ul>
      </div>
    );
  }
}

```


**imports/ui/Task.js**
```
import React, { Component } from 'react';
 
// Task component - represents a single todo item
export default class Task extends Component {
  render() {
    return (
      <li>{this.props.task.text}</li>
    );
  }
}
```


Et vahepeal asja ilusamaks teha, võib lisada CSSi, **client/main.css** faili, näiteks
```
body {
  font-family: sans-serif;
  background-color: #315481;
  background-image: linear-gradient(to bottom, #315481, #918e82 100%);
  background-attachment: fixed;
 
  position: absolute;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
 
  padding: 0;
  margin: 0;
 
  font-size: 14px;
}
 
.container {
  max-width: 600px;
  margin: 0 auto;
  min-height: 100%;
  background: white;
}
 
header {
  background: #d2edf4;
  background-image: linear-gradient(to bottom, #d0edf5, #e1e5f0 100%);
  padding: 20px 15px 15px 15px;
  position: relative;
}
 
#login-buttons {
  display: block;
}
 
h1 {
  font-size: 1.5em;
  margin: 0;
  margin-bottom: 10px;
  display: inline-block;
  margin-right: 1em;
}
 
form {
  margin-top: 10px;
  margin-bottom: -10px;
  position: relative;
}
 
.new-task input {
  box-sizing: border-box;
  padding: 10px 0;
  background: transparent;
  border: none;
  width: 100%;
  padding-right: 80px;
  font-size: 1em;
}
 
.new-task input:focus{
  outline: 0;
}
 
ul {
  margin: 0;
  padding: 0;
  background: white;
}
 
.delete {
  float: right;
  font-weight: bold;
  background: none;
  font-size: 1em;
  border: none;
  position: relative;
}
 
li {
  position: relative;
  list-style: none;
  padding: 15px;
  border-bottom: #eee solid 1px;
}
 
li .text {
  margin-left: 10px;
}
 
li.checked {
  color: #888;
}
 
li.checked .text {
  text-decoration: line-through;
}
 
li.private {
  background: #eee;
  border-color: #ddd;
}
 
header .hide-completed {
  float: right;
}
 
.toggle-private {
  margin-left: 5px;
}
 
@media (max-width: 600px) {
  li {
    padding: 12px 15px;
  }
 
  .search {
    width: 150px;
    clear: both;
  }
 
  .new-task input {
    padding-bottom: 5px;
  }
}
```

## Taskid andmebaasist

Hetkel taskid tulevad ette antud nimekirjast, mida muuta ei saa, et asju lisada ja kuskil hoida, kasutame mongot. Selleks loome faili **imports/api/task.js**, mille sisu on
```
import { Mongo } from 'meteor/mongo';
 
export const Tasks = new Mongo.Collection('tasks');
```


Selle impordime failis **server/main.js**
```
import '../imports/api/tasks.js';
```


Et Reacti komponendi sees seda datat mida salvestame kasutada siis põhikaustas
```
meteor add react-meteor-data
```

Lisame **imports/ui/App.js** faili veel mõned read, algusesse 2 importi
```
import { withTracker } from 'meteor/react-meteor-data';
import { Tasks } from '../api/tasks.js';
```

App komponent ei tohi olla enam
```
export default class App extends Component
```
vaid  peab olema
```
export class App extends Component
```

renderTasks komponendi App sisse ja sisu ära vahetada järgnevaks
```
renderTasks() {
  return this.props.tasks.map((task) => (
    <Task key={task._id} task={task} />
  ));
}
```

kõige lõppu
```
export default withTracker(() => {
  return {
    tasks: Tasks.find({}).fetch(),
  };
})(App);
```

Lisame mõned kirjed siis, et veebist näha oleks
`meteor mongo` konsooli ja siis 
```
db.tasks.insert({ text: "Task1", createdAt: new Date() });
```
lisab uue kirje


##Vormi abil veebist lisamine

**imports/ui/App.js** header osasse renderi sees läheb
```
<form className="new-task" onSubmit={this.handleSubmit.bind(this)} >
  <input
    type="text"
    ref="textInput"
    placeholder="Type to add new tasks"
  />
</form>
```

ja App komponent saab uue meetodi
```
handleSubmit(event) {
  event.preventDefault();

  // Leiab tekstivälja Reacti *refs* abil
  const text = ReactDOM.findDOMNode(this.refs.textInput).value.trim();

  Tasks.insert({
    text,
    createdAt: new Date(),
  });

  // Tühjendab vormi
  ReactDOM.findDOMNode(this.refs.textInput).value = '';
}
```

viimasena uuendame expord'i return statement'i, uus näeb välja selline
```
export default withTracker(() => {
  return {
    tasks: Tasks.find({}, { sort: { createdAt: -1 } }).fetch(),
  };
})(App);
```

## Taskide tehtuks märkimine ja kustutamine
Et oleks võimalik veebis taske maha tõmmata ja kustutada, on vaja muuta **imports/ui/Task.js** faili, alustame sellest, et lisame impordi
```
import { Tasks } from '../api/tasks.js';
```

Komponent Task saab omale uued meetodid `toggleChecked()` ja `deleteThisTask()`
```
toggleChecked() {
  Tasks.update(this.props.task._id, {
    $set: { checked: !this.props.task.checked },
  });
}

deleteThisTask() {
  Tasks.remove(this.props.task._id);
}
```

Samuti uuendame Task'i render meetodi sisu
```
render() {
  const taskClassName = this.props.task.checked ? 'checked' : '';

  return (
    <li className={taskClassName}>
    <button className="delete" onClick={this.deleteThisTask.bind(this)}>
      &times;
    </button>

    <input
      type="checkbox"
      readOnly
      checked={!!this.props.task.checked}
      onClick={this.toggleChecked.bind(this)}
    />

    <span className="text">{this.props.task.text}</span>
    </li>
  );
}
```