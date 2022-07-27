# Create a databse from csv file

```python
  connection = sqlite3.connect('flights.db')
  cursor = connection.cursor()
  
  cursor.execute('''CREATE TABLE FlightLeg (
    id INT PRIMARY KEY NOT NULL, 
    tailNumber text NOT NULL,
    sourceAirportCode CHAR(3),
    sourceCountryCode CHAR(3),
    destinationAirportCode CHAR(3),
    destinationCountryCode CHAR(3),
    departureTimeUtc TEXT,
    landingTimeUtc TEXT);''')
   
  url = "https://bitpeak.pl/datasets/flightlegs.csv"
  flightlegs = pd.read_csv(url)
```
