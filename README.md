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
