const express = require('express');
const bodyParser = require('body-parser');
const cors = require('cors');
const { Pool } = require('pg');

const app = express();
const port = 5000;

app.use(cors());
app.use(bodyParser.json());

const pool = new Pool({
    user: 'your_user',
    host: 'localhost',
    database: 'task_management',
    password: 'your_password',
});

// Create Task
app.post('/tasks', async (req, res) => {
    const { name, description, due_date, status, priority } = req.body;
    const result = await pool.query(
        'INSERT INTO tasks (name, description, due_date, status, priority) VALUES ($1, $2, $3, $4, $5) RETURNING *',
        [name, description, due_date, status, priority]
    );
    res.status(201).json(result.rows[0]);
});
app.put('/tasks/:id', async (req, res) => {
    const { id } = req.params;
    const { status } = req.body;
    const result = await pool.query(
        'UPDATE tasks SET status = $1 WHERE id = $2 RETURNING *',
        [status, id]
    );
    res.json(result.rows[0]);
});
app.get('/tasks', async (req, res) => {
    const result = await pool.query('SELECT * FROM tasks');
    res.json(result.rows);
});
app.get('/tasks/search', async (req, res) => {
    const { name } = req.query;
    const result = await pool.query('SELECT * FROM tasks WHERE name ILIKE $1', [`%${name}%`]);
    res.json(result.rows);
});

app.listen(port, () => {
    console.log(`Server running at http://localhost:${port}`);
});
