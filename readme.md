# pytebis Python Connector for TeBIS from Steinhaus

pytebis is a connector for interacting with a TeBIS Server.

The connector can return structured data in a defined timespan with defined measuring points.
There are function to get the data as structured NumPy Array, Pandas or as json.
For further interaction it is possible to load the measuring points, the groups and the tree.
Alarms are currently not supported.

## Install the package

```python
pip install pytebis
```

## Usage

### Import the package

```python
from pytebis import tebis
```

### Basic configuration

With the basic configuration it is possible to read data and to load the measuring point names and ids.
The advanced Configuration is needed to additional load the groups and tree config.

```python
configuration = {
    'host': '192.168.1.10', # The tebis host IP Adr
    'configfile': 'd:/tebis/Anlage/Config.txt' # Tebis config file loaction on the server -> ask your admin
}
teb = tebis.Tebis(configuration=configuration)
```

### Advanced configuration

```python
configuration = {
            'host': None, #Your Server IP-Adress
            'port': 4712, #Communication port [4712]
            'configfile': 'd:/tebis/Anlage/Config.txt', #Your tebis Instance to connect to.
            'useOracle': None,  # use Oracle true / false, Opt. if not defined a defined OracleDbConn.Host will set it to true. Use false to deactive Oracle usage
            'OracleDbConn': {
                'host': None,  # Host IP-Adr
                'port': 1521,  # host port [1521]
                'schema': None,  # schema name opt. if not set user is used as schema name
                'user': None,  # db user
                'psw': None,  # db pwd
                'service': 'XE'  # Oracle service name
            },
            'liveValues': {
                'enable': False,    # Use LiveValue Feature - This is used to compensate possible timedrifts between the Tebis Server and the Client.
                'recalcTimeOffsetEvery': 600,  # When using LiveValues recalc TimeOffset every x Seconds
                'offsetMstId': 100025,  # This is the Mst which is used to calculate the last available Timestamp. Use a always available mst.
            }
        }
teb = tebis.Tebis(configuration=configuration)
```

### read Data from TeBIS

There are different functions to read data from the TeBIS Server. All functions have the some parameters. Only the return is specific to the function.
Parameters:

`result = teb.getDataAsJson(names, start, end, rate=1)`

- names = Array of all mst-names to read. You can pass a array of IDs, names, TebisMst-Objects or Group-Objects (even mixed).
- start = Unix-Timestamp where to start the read 
          - or -
          DateTimeObject
          - or -
          String in Format '%Y-%m-%d %H:%M:%S.%f'
          (always the same timezone as the server is)
- end = Unix-Timestamp where to end the read
          - or -
          DateTimeObject
          - or -
          String in Format '%Y-%m-%d %H:%M:%S.%f'
          (always the same timezone as the server is)
- rate = What reduction should be used for the read. If your System supports Values smaller than 1second use Fractions of Seconds. eg: 0.1 for 100ms

The Data which is returned by the TeBIS-Server is vectorized into a structured numpy array. Which is working super fast and is totally comparable with the performance of the TeBIS A Client. You can use different functions to get the data in std. Python formats for further analysis.

#### as Numpy structured array

```python
resNP = teb.getDataAsNP(['My_mst_1','My_mst_2'], 1581324153, 1581325153, 10)
```

A structured Numpy Array is returned. There is a Column per mst-name, additional a column with the timestamp is added with index 0.
You can directly access the elements e.g. by indexing them by name `resNP["timestamp"]`

#### as Pandas

```python
df = teb.getDataAsPD(['My_mst_1','My_mst_2'], 1581324153, 1581325153, 10)
```

The Pandas DataFrame will not return a column with the timestamp. But a DateTimeIndex. So you can directly use this for TimeSeries Operations. The creation of the Pandas Dataframe is a bit slower than the generic NumPy function, as the DataFrame and the DateTimeIndex is generated afterwards.

#### as Json

```python
resJSON = teb.getDataAsJson(['My_mst_1','My_mst_2'], 1581324153, 1581325153, 10)
```

#### Example

This will show a plot containing the last hour of data of the point-ids 1 and 2. The reduction is 10 seconds.

```python
import time
import matplotlib.pyplot as plt
from pytebis import tebis

def example():
    configuration = {
        'host': '192.168.1.10',  # The tebis host IP Adr
        'configfile': 'd:/tebis/Anlage/Config.txt' # Tebis config file loaction on the server -> ask your admin
    }
    teb = tebis.Tebis(configuration=configuration)
    df = teb.getDataAsPD([1,2], time.time() - 3600, time.time(), 10)  # adjust which points you want to load pass id, name, mst- or group-Object
    df.plot()
    plt.show()

if __name__ == "__main__":
    example()
    pass
```

### Working with measuring points, groups and the tree

The measuring points and the virtual measuring points are loaded once at startup. This is always possible so you don't need to specify a db Connection.

If you want to load the Groups, Group Members and the Tree as it is configured in the TeBIS A client you must have a working db Connection.

If you have a long running service it is a good idea to reload the information in a regular interval. (e.g. all 10min)

Just call ```teb.refreshMsts()``` to reload the data.


### Logging

The package is implementing a logger using the std. logging framework of Python. The loggername is: ```pytebis```. There is no handler configured. To setup a specific log-level for the package use a config like this after ```logging.basicConfig()``` e.g. ```logging.getLogger('pytebis').setLevel(logging.INFO)``` 
