# solaredge_modbus

solaredge_modbus is a python library that collects data from SolarEdge power inverters over Modbus or ModbusTCP.

## Installation

To install, either clone this project and install using `setuptools`:

```python3 setup.py install```

or install the package from PyPi:

```pip3 install solaredge_modbus```

## Usage

The script `example.py` provides a minimal example of connecting to and displaying all registers from a SolarEdge power inverter over ModbusTCP.

```
usage: example.py [-h] [--timeout TIMEOUT] [--unit UNIT] [--json] host port

positional arguments:
  host               ModbusTCP address
  port               ModbusTCP port

optional arguments:
  -h, --help         show this help message and exit
  --timeout TIMEOUT  Connection timeout
  --unit UNIT        Modbus unit
  --json             Output as JSON
```

Output:

```
Inverter(10.0.0.123:1502, connectionType.TCP: timeout=1, retries=3, unit=0x1):

Registers:
    Model: SE3500H-RW000BNN4
    Type: Single Phase Inverter
    Version: 0004.0009.0030
    Serial: 123ABC12
    Status: Producing
    Temperature: 49.79°C
    Current: 8.93A
    Voltage: 240.20V
    Frequency: 50.00Hz
    Power: 2141.80W
    Power (Apparent): 2149.60VA
    Power (Reactive): 183.20VAr
    Power Factor: 99.69%
    Total Energy: 3466757Wh
    DC Current: 5.68A
    DC Voltage: 382.50V
    DC Power: 2173.50W
```

Passing `--json` returns:

```
{
    'c_model': 'SE3500H-RW000BNN4',
    'c_version': '0004.0009.0030',
    'c_serialnumber': '123ABC12',
    'c_deviceaddress': 1,
    'c_sunspec_did': 101,
    'current': 895,
    'p1_current': 895,
    'p2_current': False,
    'p3_current': False,
    'current_scale': -2,
    'p1_voltage': 2403,
    'p2_voltage': False,
    'p3_voltage': False,
    'p1n_voltage': False,
    'p2n_voltage': False,
    'p3n_voltage': False,
    'voltage_scale': -1,
    'frequency': 50003,
    'frequency_scale': -3,
    'power_ac': 21413,
    'power_ac_scale': -1, 
    'power_apparent': 21479,
    'power_apparent_scale': -1,
    'power_reactive': 16859,
    'power_reactive_scale': -2,
    'power_factor': 9969,
    'power_factor_scale': -2,
    'energy_total': 3466757,
    'energy_total_scale': 0,
    'current_dc': 5678,
    'current_dc_scale': -3,
    'voltage_dc': 3826,
    'voltage_dc_scale': -1,
    'power_dc': 21726,
    'power_dc_scale': -1,
    'temperature': 4979,
    'temperature_scale': -2,
    'status': 4,
    'vendor_status': 0
}
```

## Examples

If you wish to use ModbusTCP the following parameters are relevant:

`host = IP or DNS name of your ModbusTCP device, required`  
`port = listening port of the ModbusTCP device, required`  
`unit = Modbus device id, default=1, optional`

While if you are using a serial Modbus connection you can specify:

`device = path to serial device, e.g. /dev/ttyUSB0, required`  
`baud = baud rate of your device, defaults to product default, optional`  
`unit = Modbus unit id, defaults to 1, optional`

Connecting to the inverter:

```
    >>> import solaredge_modbus

    # Inverter over ModbusTCP
    >>> inverter = solaredge_modbus.Inverter(host="10.0.0.123", port=1502)
    
    # Inverter over Modbus RTU
    >>> inverter = solaredge_modbus.Inverter(device="/dev/ttyUSB0", baud=115200)
```

Test the connection, remember that only a single connection at a time is allowed:

```
    >>> inverter.connected()
    True
```

Printing the class yields basic device parameters:

```
    >>> inverter
    Inverter(10.0.0.123:1502, connectionType.TCP: timeout=1, retries=3, unit=0x1)
```

Reading a single input register by name:

```
    >>> inverter.read("current")
    {
        'current': 895
    }
```

Read all input registers using `read_all()`:

```
    >>> inverter.read_all()
    {
        'c_model': 'SE3500H-RW000BNN4',
        'c_version': '0004.0009.0030',
        'c_serialnumber': '123ABC12',
        'c_deviceaddress': 1,
        'c_sunspec_did': 101,
        'current': 895,
        'p1_current': 895,
        'p2_current': False,
        'p3_current': False,
        'current_scale': -2,
        'p1_voltage': 2403,
        'p2_voltage': False,
        'p3_voltage': False,
        'p1n_voltage': False,
        'p2n_voltage': False,
        'p3n_voltage': False,
        'voltage_scale': -1,
        'power_ac': 21413,
        'power_ac_scale': -1, 
        'frequency': 50003,
        'frequency_scale': -3,
        'power_apparent': 21479,
        'power_apparent_scale': -1,
        'power_reactive': 16859,
        'power_reactive_scale': -2,
        'power_factor': 9969,
        'power_factor_scale': -2,
        'energy_total': 3466757,
        'energy_total_scale': 0,
        'current_dc': 5678,
        'current_dc_scale': -3,
        'voltage_dc': 3826,
        'voltage_dc_scale': -1,
        'power_dc': 21726,
        'power_dc_scale': -1,
        'temperature': 4979,
        'temperature_scale': -2,
        'status': 4,
        'vendor_status': 0
    }
```

If you need more information about a particular register, to look up the units or enumerations, for example:

```
    >>> inverter.registers["current"]
        # address, length, type, datatype, valuetype, name, unit, batching
        (40071, 1, <registerType.HOLDING: 2>, <registerDataType.UINT16: 3>, <class 'int'>, 'Current', 'A', 2)

    >>> inverter.registers["status"]
        # address, length, type, datatype, valuetype, name, unit, batching
        (40107, 1, <registerType.HOLDING: 2>, <registerDataType.UINT16: 3>, <class 'int'>, 'Status', ['Undefined', 'Off', 'Sleeping', 'Grid Monitoring', 'Producing', 'Producing (Throttled)', 'Shutting Down', 'Fault', 'Standby'], 2)
```

### Meters

SolarEdge supports various kWh meters and exposes their registers through a set of pre-defined registers on the inverter. The number of supported registers is hard-coded, per the SolarEdge SunSpec implementation, to three. It is possible to query the meter registers:

```
    >>> inverter.meters()
    {
        'Meter1': Meter1(10.0.0.123:1502, connectionType.TCP: timeout=1, retries=3, unit=0x1)
    }

    >>> meter1 = inverter.meters()["Meter1"]
    >>> meter1
    Meter1(10.0.0.123:1502, connectionType.TCP: timeout=1, retries=3, unit=0x1)

    >>> meter1.read_all()
    {
        'c_model': 'PRO380-Mod',
        'c_option': 'Export+Import',
        'c_version': '2.19',
        'c_serialnumber': '12312332',
        'c_deviceaddress': 1,
        'c_sunspec_did': 203,
        'current': -13,
        ...
    }
```

**Note:** as I do not have access to a compatible kWh meter, the meter implementation is not thoroughly tested. If you have issues with this functionality, please open a GitHub issue.

## Contributing

Contributions are more than welcome.