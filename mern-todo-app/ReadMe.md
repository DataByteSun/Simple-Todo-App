# ToDo Application

## Setup Your Environment

### Prerequisites
1. Install Node.js from [nodejs.org](https://nodejs.org/). This will also install npm (Node Package Manager).
2. Install [MongoDB](https://www.mongodb.com/try/download/community) Community Databaase.

### Development Tools

- Install a text editor like [Visual Studio Code](https://code.visualstudio.com/).
- Use [Postman](https://www.postman.com/downloads/) to test API routes.

### Dependencies

To install the necessary Node.js packages, run the following command:
```bash
npm install express mongoose cors dotenv
```
- *Express:* Web framework for Node.js.
- *Mongoose:* MongoDB object modeling tool.
- *CORS:* Middleware to allow cross-origin requests (important when connecting React to Express).
- *dotenv:* For environment variables (e.g., database credentials).

Install nodemon for development purposes (optional but helpful):
```bash
npm install -g nodemon
```
Install axios to handle HTTP requests:
```bash
npm install axios
```
## 1. Backend Development (Node.js, Express, MongoDB)
1.1 Create a folder for project:
```bash
mkdir mern-todo-app
cd mern-todo-app
```
1.2 Initialize a Node.js project:
```bash
npm init -y
```
### Set up Express and MongoDB
1.3 Create the basic structure:
```bash
└── backend/
    ├── models/
    ├── routes/
    ├── .env
    ├── server.js
```
1.4 server.js (Entry point of the server):
```Javascript
const express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');
const dotenv = require('dotenv');

dotenv.config();
const app = express();

// Middleware
app.use(cors());
app.use(express.json()); // to parse JSON bodies

// MongoDB connection
mongoose.connect(process.env.MONGO_URI, {
    useNewUrlParser: true,
    useUnifiedTopology: true
}).then(() => console.log('MongoDB connected'))
  .catch(err => console.log(err));

// Start the server
const PORT = process.env.PORT || 5000;
app.listen(PORT, () => console.log(`Server running on port ${PORT}`));

const taskRoutes = require('./routes/tasks');
app.use('/api', taskRoutes);
```
1.5 Environment Variables (`.env` file):
Create a `.env` file in your backend root directory with the following content:
```bash
MONGO_URI=<Your MongoDB Connection String>
```
**Note:** Mostly the address may be: `MONGO_URI=mongodb://localhost:27017/todoapp`

1.6 Task Model (MongoDB Schema):
Create a `models` folder and add `Task.js`:
```javascript
const mongoose = require('mongoose');

const TaskSchema = new mongoose.Schema({
    title: {
        type: String,
        required: true
    },
    completed: {
        type: Boolean,
        default: false
    }
});

module.exports = mongoose.model('Task', TaskSchema);
```
1.7 Create Routes (API)
1.7.1 Task Routes:
Create a `routes` folder and add `tasks.js`:
```javascript
const express = require('express');
const Task = require('../models/Task');
const router = express.Router();

// Create a new task
router.post('/tasks', async (req, res) => {
    const { title } = req.body;
    try {
        const newTask = new Task({ title });
        const savedTask = await newTask.save();
        res.status(201).json(savedTask);
    } catch (error) {
        res.status(500).json({ message: 'Server Error' });
    }
});

// Get all tasks
router.get('/tasks', async (req, res) => {
    try {
        const tasks = await Task.find();
        res.status(200).json(tasks);
    } catch (error) {
        res.status(500).json({ message: 'Server Error' });
    }
});

// Update a task (mark as completed)
router.put('/tasks/:id', async (req, res) => {
    const { id } = req.params;
    const { completed } = req.body;
    try {
        const updatedTask = await Task.findByIdAndUpdate(id, { completed }, { new: true });
        res.status(200).json(updatedTask);
    } catch (error) {
        res.status(500).json({ message: 'Server Error' });
    }
});

// Delete a task
router.delete('/tasks/:id', async (req, res) => {
    const { id } = req.params;
    try {
        await Task.findByIdAndDelete(id);
        res.status(204).send();
    } catch (error) {
        res.status(500).json({ message: 'Server Error' });
    }
});

module.exports = router;
```
1.7.2 Hook Routes into Server:
In `server.js`, import the routes and use them:
```javascript
const taskRoutes = require('./routes/tasks');
app.use('/api', taskRoutes);
```

## 2. Frontend Development (React)
2.1 Set up React:
In the root folder (where you have `backend`), create a `frontend` folder:
```bash
npx create-react-app frontend
cd frontend
```
2.2 Create React Components:
Inside the `src` folder of your React app, create the following structure:
```bash
└── src/
    ├── components/
        ├── TaskList.js
        ├── AddTask.js
        └── Task.js
    ├── App.js
```
2.3 TaskList.js (Fetching and displaying tasks):
```javascript
import React, { useEffect, useState } from 'react';
import axios from 'axios';
import Task from './Task';
import AddTask from './AddTask';

const TaskList = () => {
    const [tasks, setTasks] = useState([]);

    useEffect(() => {
        fetchTasks();
    }, []);

    const fetchTasks = async () => {
        const res = await axios.get('/api/tasks');
        setTasks(res.data);
    };

    const handleDelete = async (id) => {
        await axios.delete(`/api/tasks/${id}`);
        fetchTasks();
    };

    const handleComplete = async (id) => {
        const task = tasks.find(t => t._id === id);
        await axios.put(`/api/tasks/${id}`, { completed: !task.completed });
        fetchTasks();
    };

    return (
        <div>
            <AddTask fetchTasks={fetchTasks} />
            <ul>
                {tasks.map(task => (
                    <Task key={task._id} task={task} onDelete={handleDelete} onComplete={handleComplete} />
                ))}
            </ul>
        </div>
    );
};

export default TaskList;
```
2.4 Task.js (Individual Task):
```javascript
import React from 'react';

const Task = ({ task, onDelete, onComplete }) => {
    return (
        <li style={{ textDecoration: task.completed ? 'line-through' : '' }}>
            {task.title}
            <button onClick={() => onComplete(task._id)}>{task.completed ? 'Undo' : 'Complete'}</button>
            <button onClick={() => onDelete(task._id)}>Delete</button>
        </li>
    );
};

export default Task;
```
2.5 AddTask.js (Add new tasks):
```javascript
import React, { useState } from 'react';
import axios from 'axios';

const AddTask = ({ fetchTasks }) => {
    const [title, setTitle] = useState('');

    const handleSubmit = async (e) => {
    e.preventDefault();
    try {
        // The URL should be /api/tasks
        await axios.post('/api/tasks', { title });
        setTitle('');
        fetchTasks();
    } catch (err) {
        console.error('Error adding task:', err);
    }
};

    return (
        <form onSubmit={handleSubmit}>
            <input
                type="text"
                value={title}
                onChange={(e) => setTitle(e.target.value)}
                placeholder="New Task"
                required
            />
            <button type="submit">Add Task</button>
        </form>
    );
};

export default AddTask;
```
2.6 Render the TaskList component in your main App.js file:
- Open `src/App.js`.
Replace the default code with the following:
```javascript
import React from 'react';
import TaskList from './components/TaskList';

function App() {
    return (
        <div className="App">
            <h1>Todo List</h1>
            <TaskList />
        </div>
    );
}

export default App;
```
## 3. Connecting Frontend with Backend
3.1 Proxy Setup in React:
In `frontend/package.json`, add a proxy to point to the backend:
```json
"proxy": "http://localhost:5000"
```
This will ensure that requests from React (running on `localhost:3000`) are directed to the backend (`localhost:5000`).
3.2 Run Backend and Frontend:
- In one terminal, navigate to the `backend` folder and run:
```bash
nodemon server.js
```
- In another terminal, navigate to the `frontend` folder and run:
```bash
npm start
```
## 4. Any Difficulty
- Ensure Backend is Running
  - Double-check that your backend server is running on http://localhost:5000. To confirm, you can check the console logs from the terminal where the backend is running. It should show something like Server running on port 5000.
  - Use Postman or curl to manually send a POST request to http://localhost:5000/api/tasks and see if it returns the expected response.
  - For example, you can send this POST request via Postman:
  -   URL: http://localhost:5000/api/tasks
  -   Method: POST
  -   Body: JSON:
```json
{
    "title": "Sample Task"
}
```
- Check Proxy Configuration
  - Make sure your React app's proxy is set up properly to forward requests to the backend. In the frontend's `package.json`, the `proxy` setting should be:
```json
"proxy": "http://localhost:5000"
```
  - This allows requests like `/api/tasks` from React to be forwarded to `http://localhost:5000/api/tasks`.
- Restart Both Servers
  - Restart both the *backend* and *frontend* servers to ensure all changes are applied.
  - For the `backend`, run:
```bash
nodemon server.js
```
  - For the `frontend`, navigate to frontend and run:
```bash
npm start
```
## Contributions and Feedback

We welcome any contributions to this project! If you have ideas for new features or improvements, please feel free to send a request or open an issue.

If you encounter any problems or have questions, don’t hesitate to add them in the **Discussion** section. Your feedback is greatly appreciated!

Thank you for being part of this project!
