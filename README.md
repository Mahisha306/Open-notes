const express = require('express');
const Note = require('../models/Note');
const router = express.Router();


// Add note
router.post('/', async (req, res) => {
const note = new Note(req.body);
await note.save();
res.json(note);
});


// Get all notes (with search)
router.get('/', async (req, res) => {
const { q } = req.query;
let filter = {};
if (q) {
filter = { $or: [
{ title: new RegExp(q, 'i') },
{ content: new RegExp(q, 'i') },
{ keywords: { $in: [q] } }
] };
}
const notes = await Note.find(filter);
res.json(notes);
});


// Update rating
router.patch('/:id/rate', async (req, res) => {
const { rating } = req.body;
const note = await Note.findByIdAndUpdate(req.params.id, { rating }, { new: true });
res.json(note);
});


module.exports = router;


// frontend/src/App.js
import React, { useEffect, useState } from 'react';
import NoteCard from './components/NoteCard';


function App() {
const [notes, setNotes] = useState([]);
const [query, setQuery] = useState('');


useEffect(() => {
fetch(`http://localhost:5000/api/notes?q=${query}`)
.then(res => res.json())
.then(data => setNotes(data));
}, [query]);


return (
<div className="p-4">
<h1 className="text-2xl font-bold mb-4">OpenNotes</h1>
<input
type="text"
placeholder="Search notes..."
value={query}
onChange={e => setQuery(e.target.value)}
className="border p-2 mb-4 w-full"
/>
<div className="grid gap-4">
{notes.map(note => <NoteCard key={note._id} note={note} />)}
</div>
</div>
);
}


export default App;


// frontend/src/components/NoteCard.js
import React from 'react';


function NoteCard({ note }) {
return (
<div className="border rounded p-4 shadow">
<h2 className="font-semibold">{note.title}</h2>
<p>{note.content}</p>
<p className="text-sm text-gray-500">{note.subject} | {note.grade}</p>
<p>⭐ {note.rating}</p>
</div>
);
}


export default NoteCard;
