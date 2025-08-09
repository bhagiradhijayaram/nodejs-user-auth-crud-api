# nodejs-user-auth-crud-api

### Authentication APIs

```
const express = require('express')
const path = require('path')
const sqlite = require('sqlite3')
const {open} = require('sqlite')
const bcrypt = require('bcrypt')
const app = express()

app.use(express.json())

let db = null
const dbPath = path.join(__dirname, 'userData.db')

// Backend + Database Setup
const initializeDBandServer = async () => {
  try {
    db = await open({
      filename: dbPath,
      driver: sqlite.Database,
    })
    app.listen(3000, () => {
      console.log('Server Running at http://localhost:3000/')
    })
  } catch (error) {
    console.log(error)
    process.exit(1)
  }
}

// Register API
app.post('/register/', async (req, res) => {
  const {username, name, password, gender, location} = req.body
  const hashPassword = await bcrypt.hash(password, 10)
  const selectUserQuery = `SELECT * FROM user WHERE username = '${username}'`
  const dbUser = await db.get(selectUserQuery)

  // Password Checking
  if (password.length <= 5) {
    res.status(400).send('Password is too short')
  }

  // User Checking
  if (dbUser === undefined) {
    const addUserQuery = `
    INSERT INTO user(username, name, password, gender, location) VALUES('${username}','${name}','${hashPassword}','${gender}','${location}')`
    await db.run(addUserQuery)
    res.status(200).send('User created successfully')
  } else {
    res.status(400)
    res.send('User already exists')
  }
})

// Login API
app.post('/login/', async (req, res) => {
  const {username, password} = req.body
  const selectUserQuery = `SELECT * FROM user WHERE username = '${username}'`
  const dbUser = await db.get(selectUserQuery)
  if (dbUser === undefined) {
    // Invaild User
    res.status(400).send('Invalid user')
  } else {
    // Password Checking
    const isPasswordMatched = await bcrypt.compare(password, dbUser.password)
    if (isPasswordMatched) {
      res.status(200).send('Login success!')
    } else {
      res.status(400).send('Invalid password')
    }
  }
})

const validatePassword = password => {
  return password.length >= 5
}
// Change password API
app.put('/change-password/', async (req, res) => {
  const {username, oldPassword, newPassword} = req.body
  const selectUserQuery = `SELECT * FROM user WHERE username = '${username}'`
  const dbUser = await db.get(selectUserQuery)
  if (dbUser === undefined) {
    res.status(400).send('Invaild User')
  } else {
    const isPasswordMatched = await bcrypt.compare(oldPassword, dbUser.password)
    if (isPasswordMatched) {
      if (validatePassword(newPassword)) {
        const hashPassword = await bcrypt.hash(newPassword, 10)
        // Update Password
        const updatePasswordQuery = `
        UPDATE
          user
        SET
        password = '${hashPassword}'
        WHERE 
        username = '${username}'`
        await db.run(updatePasswordQuery)
        res.status(200).send('Password updated')
      } else {
        res.status(400).send('Password is too short')
      }
    } else {
      res.status(400).send('Invalid current password')
    }
  }
})

initializeDBandServer()

module.exports = app
```
```
const express = require('express')
const {open} = require('sqlite')
const sqlite = require('sqlite3')
const path = require('path')
const app = express()
app.use(express.json())

let db = null
const dbPath = path.join(__dirname, 'cricketTeam.db')

// Backend + Database Connection
const initializeDBAndServer = async () => {
  try {
    db = await open({
      filename: dbPath,
      driver: sqlite.Database,
    })
    app.listen(3000, () => {
      console.log('Server Running at http://localhost/3000')
    })
  } catch (error) {
    console.log(error)
  }
}

initializeDBAndServer()

const convertDataFun = dbObject => {
  return {
    playerId: dbObject.player_id,
    playerName: dbObject.player_name,
    jerseyNumber: dbObject.jersey_number,
    role: dbObject.role,
  }
}

// API 1
app.get('/players/', async (req, res) => {
  const getPlayersQuery = 'SELECT * FROM cricket_team'
  const playersData = await db.all(getPlayersQuery)
  const convertData = playersData.map(i => convertDataFun(i))
  res.send(convertData)
})

// API 2
app.post('/players/', async (req, res) => {
  const playerDetails = req.body
  const {playerName, jerseyNumber, role} = playerDetails
  const insertDataQuery = `INSERT INTO cricket_team(player_name,jersey_number,role) VALUES('${playerName}','${jerseyNumber}', '${role}')`
  const insertData = await db.run(insertDataQuery)
  console.log(playerDetails)
  res.send('Player Added to Team')
})

// API 3
app.get('/players/:playerId/', async (req, res) => {
  const {playerId} = req.params
  const getplayerQuery = `SELECT * FROM cricket_team WHERE player_id = ${playerId}`
  const playerData = await db.get(getplayerQuery)
  if (playerData) {
    res.send(playerData)
  } else {
    res.send('player not found')
  }
})

// API 4
app.put('/players/:playerId/', async (req, res) => {
  const {playerId} = req.params
  const playerDetails = req.body
  const {playerName, jerseyNumber, role} = playerDetails
  const updataPlayerQuery = `
  UPDATE cricket_team 
  SET
  player_name =  '${playerName}',
  jersey_number = '${jerseyNumber}',
  role = '${role}'
  WHERE 
  player_id = ${playerId}
  `

  await db.run(updataPlayerQuery)
  res.send('Player Details Updated')
})

// API 5
app.delete('/players/:playerId/', async (req, res) => {
  const {playerId} = req.params
  const deletePlayerQuery = `DELETE FROM cricket_team WHERE player_id = ${playerId}`
  await db.run(deletePlayerQuery)
  res.send('Player Removed')
})

module.exports = app
```
