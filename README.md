# NodeJSIoT

This example shows how measurement data can be displayed in a web application using Node.js (Express) and Google Chart so that the data is updated in real time.

The program datagenerator.py generates simulated measurement data and sends it to the server program. The server program index.js, implemented with Express.js, receives the measurements and passes them to an html page where they are displayed as a Google Chart.

## Files

### Datagenerator.py

The Python program datageneratorclient.py generates simulated measurement data using trigonometric functions and a random number generator:

```python
    # let's play that we will read data from sensors
    # simulate the sensor values
    pressure = 100 * math.sin(i/10)
    temperature = 50 * math.cos(i / 3)
    humidity = random.random() * 20 - 10

    # create an object
    measurements = { 
        "pressure" : pressure,
        "temperature" : temperature,
        "humidity" : humidity
    }
```

The generated measurements are sent to the server via HTTP Post:

```python
    # serialize the object to json and POST it the thingspeak
    response = requests.post('http://localhost:3001/api/measurements/', json=measurements)
```

### index.js

The express program index.js receives the measurements and passes them to an html page using socket.io.

The start of the program is shown below. Socket.io requires CORS to be taken into account. The json middleware must also be enabled. Incoming measurements are stored in the measurementsArray list.

```javascript
const express = require('express')
const app = express()
const socket = require("socket.io");

app.use(express.json())

const cors = require('cors')

app.use(cors())

// View engine setup
app.set('view engine', 'ejs');

// list for the measurements
let measurementsArray = []
```

The function (route) app.get('/', (request, response) handles a page request to the root. The function opens the measurements.ejs page in the browser. This program is based on the use of ejs templates.

```javascript
app.get('/', (request, response) => {
  response.render('measurements')
})
```

The function (route) app.post('/api/measurements', (request, response) receives a message sent via HTTP POST to /api/measurements. First, it checks that the message contains data. The data originally in the dictionary (pressure, temperature, humidity) is converted to list format, since Google Charts requires the data in list format.

The whole list is serialized and sent to the browser application via an io.emit message.

```javascript
app.post('/api/measurements', (request, response) => {
  const body = request.body

  if (!body.pressure) {
    return response.status(400).json({ 
      error: 'content missing' 
    })
  }

  let id = measurementsArray.length + 1
  const arrayrow = [id.toString(), body.pressure, body.temperature, body.humidity]

  measurementsArray.push(arrayrow)

  // lähetetään lista listoja [[,,,], [,,,]...]
  var s = JSON.stringify(measurementsArray);

  // lähetetään kaikki mittaukset
  io.emit('measdata', s);
  console.log(arrayrow)

  response.json(arrayrow)
})  
```
