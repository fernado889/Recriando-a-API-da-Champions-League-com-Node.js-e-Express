# Recriando-a-API-da-Champions-League-com-Node.js-e-Express
mkdir champions-league-api
cd champions-league-api
npm init -y
npm install express mongoose dotenv
const mongoose = require('mongoose');

const teamSchema = new mongoose.Schema({
  name: String,
  country: String,
  founded: Number,
  stadium: String,
});

module.exports = mongoose.model('Team', teamSchema);
const mongoose = require('mongoose');

const matchSchema = new mongoose.Schema({
  date: Date,
  homeTeam: { type: mongoose.Schema.Types.ObjectId, ref: 'Team' },
  awayTeam: { type: mongoose.Schema.Types.ObjectId, ref: 'Team' },
  homeScore: Number,
  awayScore: Number,
});

module.exports = mongoose.model('Match', matchSchema);
require('dotenv').config();
const express = require('express');
const mongoose = require('mongoose');
const app = express();

mongoose.connect(process.env.MONGODB_URI, { useNewUrlParser: true, useUnifiedTopology: true })
  .then(() => console.log('Conectado ao MongoDB'))
  .catch((err) => console.error('Erro ao conectar ao MongoDB:', err));

app.use(express.json());

// Definindo as rotas
const teamsRouter = require('./routes/teams');
const matchesRouter = require('./routes/matches');
app.use('/api/teams', teamsRouter);
app.use('/api/matches', matchesRouter);

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`Servidor rodando na porta ${PORT}`);
});
const express = require('express');
const Team = require('../models/Team');
const router = express.Router();

// Adicionar um novo time
router.post('/', async (req, res) => {
  try {
    const team = new Team(req.body);
    await team.save();
    res.status(201).json(team);
  } catch (error) {
    res.status(400).json({ message: error.message });
  }
});

// Obter todos os times
router.get('/', async (req, res) => {
  try {
    const teams = await Team.find();
    res.json(teams);
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
});

module.exports = router;
const express = require('express');
const Match = require('../models/Match');
const router = express.Router();

// Adicionar uma nova partida
router.post('/', async (req, res) => {
  try {
    const match = new Match(req.body);
    await match.save();
    res.status(201).json(match);
  } catch (error) {
    res.status(400).json({ message: error.message });
  }
});

// Obter todas as partidas
router.get('/', async (req, res) => {
  try {
    const matches = await Match.find().populate('homeTeam').populate('awayTeam');
    res.json(matches);
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
});

module.exports = router;
MONGODB_URI=mongodb://localhost:27017/champions-league
node index.js
