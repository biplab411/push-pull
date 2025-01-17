const express = require('express');
const bodyParser = require('body-parser');
const mongoose = require('mongoose');
const cors = require('cors');

const app = express();
const PORT = process.env.PORT || 3000;

app.use(cors());
app.use(bodyParser.json());

mongoose.connect('mongodb://localhost:27017/github-webhook', {
    useNewUrlParser: true,
    useUnifiedTopology: true,
});

const eventSchema = new mongoose.Schema({
    type: String,
    author: String,
    from_branch: String,
    to_branch: String,
    timestamp: Date,
});

const Event = mongoose.model('Event', eventSchema);

app.post('/webhook', async (req, res) => {
    const eventType = req.headers['x-github-event'];
    const author = req.body.pusher?.name || req.body.pull_request?.user?.login;
    const timestamp = new Date().toISOString();

    if (eventType === 'push') {
        const toBranch = req.body.ref.split('/').pop();
        const newEvent = new Event({ type: 'push', author, to_branch: toBranch, timestamp });
        await newEvent.save();
    } else if (eventType === 'pull_request') {
        const fromBranch = req.body.pull_request?.head?.ref;
        const toBranch = req.body.pull_request?.base?.ref;
        const newEvent = new Event({ type: 'pull_request', author, from_branch: fromBranch, to_branch: toBranch, timestamp });
        await newEvent.save();
    } else if (eventType === 'merge') {
        const fromBranch = req.body.pull_request?.head?.ref;
        const toBranch = req.body.pull_request?.base?.ref;
        const newEvent = new Event({ type: 'merge', author, from_branch: fromBranch, to_branch: toBranch, timestamp });
        await newEvent.save();
    }

    res.status(200).send('Event received');
});

app.get('/events', async (req, res) => {
    const events = await Event.find().sort({ timestamp: -1 }).limit(10);
    res.json(events);
});

app.listen(PORT, () => {
    console.log(`Server running on port ${PORT}`);
});
