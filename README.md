# Nodejs-user-auth-crud-api

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

### CRUD Operations APIs
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

// Getting Players Data From Database
app.get('/players/', async (req, res) => {
  const getPlayersQuery = 'SELECT * FROM cricket_team'
  const playersData = await db.all(getPlayersQuery)
  const convertData = playersData.map(i => convertDataFun(i))
  res.send(convertData)
})

// Add Players Data into Database
app.post('/players/', async (req, res) => {
  const playerDetails = req.body
  const {playerName, jerseyNumber, role} = playerDetails
  const insertDataQuery = `INSERT INTO cricket_team(player_name,jersey_number,role) VALUES('${playerName}','${jerseyNumber}', '${role}')`
  const insertData = await db.run(insertDataQuery)
  console.log(playerDetails)
  res.send('Player Added to Team')
})

// Getting Specific Player Data From Database
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

// Updated Player Data
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

// Delete Player Data From Database
app.delete('/players/:playerId/', async (req, res) => {
  const {playerId} = req.params
  const deletePlayerQuery = `DELETE FROM cricket_team WHERE player_id = ${playerId}`
  await db.run(deletePlayerQuery)
  res.send('Player Removed')
})

module.exports = app
```

### Task 

```
const express = require('express')
const path = require('path')
const {open} = require('sqlite')
const sqlite = require('sqlite3')
const bcrypt = require('bcrypt')
const jwt = require('jsonwebtoken')
const app = express()

app.use(express.json())

let db = null
const dbPath = path.join(__dirname, 'covid19IndiaPortal.db')

const initializeDBandServer = async () => {
  try {
    db = await open({
      filename: dbPath,
      driver: sqlite.Database,
    })
    app.listen(3000, () => {
      console.log('Server Runnin at http://localhost:3000/')
    })
  } catch (error) {
    console.log(error)
  }
}

initializeDBandServer()

//  Data Case Conversion
const statesDataCaseConversionFun = dbData => {
  return {
    stateId: dbData.state_id,
    stateName: dbData.state_name,
    population: dbData.population,
  }
}

const districtDataCaseConversionFun = dbData => {
  return {
    districtId: dbData.district_id,
    districtName: dbData.district_name,
    stateId: dbData.state_id,
    cases: dbData.cases,
    cured: dbData.cured,
    active: dbData.active,
    deaths: dbData.deaths,
  }
}

// Middleware Function
const authenticateToken = async (req, res, next) => {
  let jwtToken
  const authHeader = req.headers['authorization']
  if (authHeader !== undefined) {
    jwtToken = authHeader.split(' ')[1]
  }
  if (jwtToken === undefined) {
    res.status(401).send('Invalid JWT Token')
  } else {
    jwt.verify(jwtToken, 'xyz234', async (error, payload) => {
      if (error) {
        res.status(400).send('Invalid JWT Token')
      } else {
        next()
      }
    })
  }
}

// Login API
app.post('/login/', async (req, res) => {
  const {username, password} = req.body
  const getUserQuery = `SELECT * FROM user WHERE username = '${username}'`
  const dbUser = await db.get(getUserQuery)
  if (dbUser === undefined) {
    res.status(400).send('Invalid user')
  } else {
    const isPasswordMatched = await bcrypt.compare(password, dbUser.password)
    if (isPasswordMatched) {
      const payload = {username: username}
      const jwtToken = jwt.sign(payload, 'xyz234')
      res.send({jwtToken})
    } else {
      res.status(400).send('Invalid password')
    }
  }
})

// Getting Data From Database (Only Authenticated User)
app.get('/states/', authenticateToken, async (req, res) => {
  const getStatesQuery = `SELECT * FROM state`
  const statesData = await db.all(getStatesQuery)
  const caseConverstionData = statesData.map(item =>
    statesDataCaseConversionFun(item),
  )
  res.send(caseConverstionData)
})

// Getting Specific State Data (Only Authenticated User)
app.get('/states/:stateId/', authenticateToken, async (req, res) => {
  const {stateId} = req.params
  const getStateQuery = `SELECT * FROM state WHERE state_id = '${stateId}'`
  const stateData = await db.get(getStateQuery)
  res.send(statesDataCaseConversionFun(stateData))
})

app.post('/districts/', authenticateToken, async (req, res) => {
  const {stateId, districtName, cases, cured, active, deaths} = req.body
  const addDistrictQuery = `
  INSERT INTO district(state_id, district_name, cases, cured, active, deaths) 
  VALUES('${stateId}','${districtName}','${cases}','${cured}','${active}','${deaths}')
  `
  await db.run(addDistrictQuery)
  res.send('District Successfully Added')
})

app.get('/districts/:districtId/', authenticateToken, async (req, res) => {
  const {districtId} = req.params
  const getDistrictQuery = `
  SELECT * FROM district WHERE district_id = '${districtId}'`
  const districtData = await db.get(getDistrictQuery)
  res.send(districtDataCaseConversionFun(districtData))
})

app.delete('/districts/:districtId/', authenticateToken, async (req, res) => {
  const {districtId} = req.params
  const getDistrictQuery = `
  DELETE FROM district WHERE district_id = '${districtId}'`
  await db.run(getDistrictQuery)
  res.send('District Removed')
})


// Update API
app.put('/districts/:districtId/', authenticateToken, async (req, res) => {
  const {districtId} = req.params
  const {districtName, stateId, cases, cured, active, deaths} = req.body
  const updateDistrictQuery = `
  UPDATE district
  SET 
  district_name = '${districtName}',
  state_id = '${stateId}',
  cases = '${cases}',
  cured = '${cured}',
  active = '${active}',
  deaths = '${deaths}'
  WHERE
  district_id = '${districtId}'
  `
  await db.run(updateDistrictQuery)
  res.send('District Details Updated')
})


module.exports = app
```

### Testing
```
###
POST http://localhost:3000/login/
Content-Type: application/json

{
  "username": "christopher_phillips",
  "password": "christy@123"
}

###
GET http://localhost:3000/states/
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImNocmlzdG9waGVyX3BoaWxsaXBzIiwiaWF0IjoxNzU0ODA3ODYzfQ.D13nnXVBVly0fwwJQHWfA9P5C8YMBHDmEdPPsS0ZmlY

###
GET http://localhost:3000/states/8/
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImNocmlzdG9waGVyX3BoaWxsaXBzIiwiaWF0IjoxNzU0ODA3ODYzfQ.D13nnXVBVly0fwwJQHWfA9P5C8YMBHDmEdPPsS0ZmlY 

###
POST http://localhost:3000/districts/
Content-Type: application/json

{
  "districtName": "Bagalkot",
  "stateId": 3,
  "cases": 2323,
  "cured": 2000,
  "active": 315,
  "deaths": 8
}

###
GET http://localhost:3000/districts/3/
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImNocmlzdG9waGVyX3BoaWxsaXBzIiwiaWF0IjoxNzU0ODA3ODYzfQ.D13nnXVBVly0fwwJQHWfA9P5C8YMBHDmEdPPsS0ZmlY 


###
DELETE http://localhost:3000/districts/2/
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImNocmlzdG9waGVyX3BoaWxsaXBzIiwiaWF0IjoxNzU0ODA3ODYzfQ.D13nnXVBVly0fwwJQHWfA9P5C8YMBHDmEdPPsS0ZmlY 

###
PUT http://localhost:3000/districts/3/
Content-Type: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImNocmlzdG9waGVyX3BoaWxsaXBzIiwiaWF0IjoxNzU0ODA3ODYzfQ.D13nnXVBVly0fwwJQHWfA9P5C8YMBHDmEdPPsS0ZmlY 

{
  "districtName": "Nadia",
  "stateId": 3,
  "cases": 9628,
  "cured": 6524,
  "active": 3000,
  "deaths": 104
}
```
### Response

```
API 1
"Method": "POST" 
"Status-Code": "200 - OK" 
"content-length": "166"
"content-type": "application/json; charset=utf-8"
 

{
    "jwtToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImNocmlzdG9waGVyX3BoaWxsaXBzIiwiaWF0IjoxNzU0ODgzMDE1fQ.qsVDJ3anz9J5Yrf7xNK5nY-25ivJPOrgJjlUW1PaauQ"
}

API 2

"Method": "GET" 
"Status-Code": "200 - OK" 
"content-length": "2226"
"content-type": "application/json; charset=utf-8"
 

[
    {
        "stateId": 1,
        "stateName": "Andaman and Nicobar Islands",
        "population": 380581
    },
    {
        "stateId": 2,
        "stateName": "Andhra Pradesh",
        "population": 49386799
    },
    {
        "stateId": 3,
        "stateName": "Arunachal Pradesh",
        "population": 1382611
    },
    ....
    ....
    ....
]

API 3
"Method": "GET" 
"Status-Code": "200 - OK" 
"content-length": "55"
"content-type": "application/json; charset=utf-8"
 

{
    "stateId": 8,
    "stateName": "Delhi",
    "population": 16787941
}

API 4
"Method": "POST" 
"Status-Code": "200 - OK" 
"content-length": "27"
"content-type": "text/html; charset=utf-8"
 

District Successfully Added
API 5
"Method": "GET" 
"Status-Code": "200 - OK" 
"content-length": "104"
"content-type": "application/json; charset=utf-8"
 

{
    "districtId": 3,
    "districtName": "Nadia",
    "stateId": 3,
    "cases": 9628,
    "cured": 6524,
    "active": 3000,
    "deaths": 104
}
API 6
"Method": "DELETE" 
"Status-Code": "200 - OK" 
"content-length": "16"
"content-type": "text/html; charset=utf-8"
 

District Removed

API 7
"Method": "PUT" 
"Status-Code": "200 - OK" 
"content-length": "24"
"content-type": "text/html; charset=utf-8"
 

District Details Updated
