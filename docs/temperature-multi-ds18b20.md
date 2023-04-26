<!--remove-start-->

# Thermometer - Multi DS18B20

<!--remove-end-->




In this example that works with ConfigurableFirmata (with its high baudrate), we wanted to keep automatic port detection for the board. 
This is why we are using CustomIO in the Board initialization.
Then we call the sendOneWireSearch on Firmata to retreive all the connected DS18B20 ti finaly initialize each sensor with its address.


```javascript


const { Board, Thermometer } = require('johnny-five');
const SerialPort = require('serialport');
const Firmata = require('firmata');
const debug = require('debug')('this')


const ONE_WIRE_PIN = 12;
const CONFIGURABLE_FIRMATA_BAUD_RATE = 115200;

/**
 * Class permettant de surcharger l'initialisation par défaut avec une baudrate adapté au Firmware installé sur l'arduino
 */
class CustomIO extends Firmata.Board {
  constructor(port, options = {}, callback) {
    const customBaudRate = CONFIGURABLE_FIRMATA_BAUD_RATE; // Remplacez par le baudrate souhaité
    const serialPort = new SerialPort(port, { baudRate: customBaudRate });

    super(serialPort, options, callback);
  }
}

function getAddress(device) {
  // 64-bit device code
  // device[0]    => Family Code
  // device[1..6] => Serial Number (device[1] is LSB)
  // device[7]    => CRC
  let i;

  let result = 0;
  for (i = 6; i > 0; i--) {
    result = result * 256 + device[i];
  }
  return result;
}

function printData(address, celsius, fahrenheit, kelvin) {
  console.log(`Thermometer at address: 0x${address.toString(16)}`);
  console.log("  celsius      : ", celsius);
  console.log("  fahrenheit   : ", fahrenheit);
  console.log("  kelvin       : ", kelvin);
  console.log("--------------------------------------");
}




// Fonction pour détecter le port de connexion de la carte
function detectPort(portDetectedCallback) {
  SerialPort.list().then((ports) => {
    const port = ports.find((port) => /usb|acm|^com/i.test(port.path));
    if (port) {
      portDetectedCallback(null, port.path);
    } else {
      portDetectedCallback(new Error('No port detected'));
    }
  });
}

detectPort((error, port) => {
  if (error) {
    console.error(error);
    return;
  }

  const board = new Board({
    io: new CustomIO(port),
  });

  board.on('ready', () => {

    debug("Board is ready");

    board.io.sendOneWireConfig(ONE_WIRE_PIN, true);
    board.io.sendOneWireSearch(ONE_WIRE_PIN, (err, devices) => {
      if (err) {
        debug("DS18B20:initialize", `error occured (${err}`);
        return;
      }

      if (devices.length === 0) {
        debug("DS18B20: FAILED TO FIND TEMPERATURE DEVICE");
        return;
      }

      devices.forEach(device => {
        const address = getAddress(device);
        debug("DS18B20:initialize", `found sensor (${address})`);
        const t = new Thermometer({
          controller: "DS18B20",
          pin: ONE_WIRE_PIN,
          address: address
        }).on("change", () => {
          const { address, celsius, fahrenheit, kelvin } = t;
          printData(address, celsius, fahrenheit, kelvin);
        });

      });
    });
  });
});

```


## Additional Notes
- [DS18B20 - Thermometer Sensor](http://www.maximintegrated.com/en/products/analog/sensors-and-sensor-interface/DS18S20.html)

&nbsp;

<!--remove-start-->

## License
Copyright (c) 2012-2014 Rick Waldron <waldron.rick@gmail.com>
Licensed under the MIT license.
Copyright (c) 2015-2023 The Johnny-Five Contributors
Licensed under the MIT license.

<!--remove-end-->
