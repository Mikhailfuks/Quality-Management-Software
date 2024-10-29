// index.js
const express = require('express');
const sqlite3 = require('sqlite3').verbose();
const bodyParser = require('body-parser');
require('dotenv').config();

const app = express();
const PORT = process.env.PORT || 3000;

// Middleware
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: true }));

// Create SQLite database
const db = new sqlite3.Database(':memory:');

// Create tables
db.serialize(() => {
    db.run(`CREATE TABLE IF NOT EXISTS documents (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        title TEXT NOT NULL,
        description TEXT,
        created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
    )`);

    db.run(`CREATE TABLE IF NOT EXISTS issues (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        title TEXT NOT NULL,
        description TEXT,
        status TEXT NOT NULL DEFAULT 'open',
        created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
    )`);
});

// Route to add a new document
app.post('/documents', (req, res) => {
    const { title, description } = req.body;
    db.run(`INSERT INTO documents (title, description) VALUES (?, ?)`, [title, description], function(err) {
        if (err) {
            return res.status(500).json({ message: 'Database error', error: err.message });
        }
        res.status(201).json({ id: this.lastID, title, description });
    });
});

// Route to list all documents
app.get('/documents', (req, res) => {
    db.all(`SELECT * FROM documents ORDER BY created_at DESC`, [], (err, rows) => {
        if (err) {
            return res.status(500).json({ message: 'Database error', error: err.message });
        }
        res.status(200).json(rows);
    });
});

// Route to add a new issue
app.post('/issues', (req, res) => {
    const { title, description } = req.body;
    db.run(`INSERT INTO issues (title, description) VALUES (?, ?)`, [title, description], function(err) {
        if (err) {
            return res.status(500).json({ message: 'Database error', error: err.message });
        }
        res.status(201).json({ id: this.lastID, title, description });
    });
});

// Route to list all issues
app.get('/issues', (req, res) => {
    db.all(`SELECT * FROM issues ORDER BY created_at DESC`, [], (err, rows) => {
        if (err) {
            return res.status(500).json({ message: 'Database error', error: err.message });
        }
        res.status(200).json(rows);
    });
});

// Route to update an issue status
app.patch('/issues/:id', (req, res) => {
    const { id } = req.params;
    const { status } = req.body;
    db.run(`UPDATE issues SET status = ? WHERE id = ?`, [status, id], function(err) {
        if (err) {


            return res.status(500).json({ message: 'Database error', error: err.message });
        }
        res.status(200).json({ message: `Issue ${id} updated successfully!` });
    });
});

// Start the server
app.listen(PORT, () => {
    console.log(`Server is running on http://localhost:${PORT}`);
});
