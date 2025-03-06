# A simple IoT application using Node.js and Google Charts

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

  // send all measurements
  io.emit('measdata', s);
  console.log(arrayrow)

  response.json(arrayrow)
})  
```

### measurements.ejs

The measurements.ejs page is in the views folder. The program uses the ejs template.

The browser program receives socket.io messages and displays the measurements in them as a line graph.

The code of the page is shown below:

```html
<html>
    <head>
      <script type="text/javascript" src="https://www.gstatic.com/charts/loader.js"></script>
      <script type="text/javascript" src="//code.jquery.com/jquery-1.4.2.min.js"></script>
      <script type="text/javascript" src="//cdnjs.cloudflare.com/ajax/libs/socket.io/4.4.0/socket.io.min.js"></script>
      <script type="text/javascript" charset="utf-8"></script>    

      <script type="text/javascript">
        ...
      </script>
    </head>
    <body>
      <div id="curve_chart" style="width: 900px; height: 500px"></div>
    </body>
  </html>
```

The line chart is displayed in the div element curve_chart.

The initialization of Google Chart and the reception of the socketio message is shown below:

```javascript
        google.charts.load('current', {'packages':['corechart', 'table']});
        google.charts.setOnLoadCallback(init);

        function init() {
          var socket = io();
          console.log("init")
          socket.on('measdata', function (data) {
            var s = JSON.parse(data);
            drawChart(s);
          })
        }
```

The received socketio message contains a list of measurements. One row of measurements is also in the form of a list (id, pressure, temperature, humidity). When the socketio message is received, it is deserialized and the transformed information is passed to drawChart().

The drawChart function draws a line chart in the div element curve_chart:

```javascript
        function drawChart(s) {
            var data = new google.visualization.DataTable();
            data.addColumn('string', 'id');
            data.addColumn('number', 'pressure');
            data.addColumn('number', 'temperature');
            data.addColumn('number', 'humidity');
            console.log("draw row", s)
            data.addRows(s);
  
          var options = {
            title: 'Measurement data',
            curveType: 'function',
            legend: { position: 'bottom' }
          };
  
          var chart = new google.visualization.LineChart(document.getElementById('curve_chart'));
  
          chart.draw(data, options);
```

## Installing libraries and running programs

The application dependencies are defined in the package.json file. 

```
{
  "name": "nodejsmeasurementserver",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "start": "node index.js",
    "dev": "nodemon index.js",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC",
  "dependencies": {
    "cors": "^2.8.5",
    "ejs": "^3.1.9",
    "express": "^4.18.2",
    "socket.io": "^4.7.2",
    "socketio": "^1.0.0"
  },
  "devDependencies": {
    "nodemon": "^3.0.1"
  }
}
```

The libraries specified in package.json are installed with the command
```
npm install
```

The server program is started with the command
```
npm run dev
```

Next, open a browser at localhost:3001

Then run the Python program that generates the measurement data
```
py datagenerator
```

New measurements are updated on the chart in the browser in real time.
