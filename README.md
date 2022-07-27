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
# Insert data into databse

```python
  insert_data = "INSERT INTO FlightLeg (id,tailNumber,sourceAirportCode,sourceCountryCode,destinationAirportCode,destinationCountryCode,departureTimeUtc,landingTimeUtc) \
      VALUES (?,?,?,?,?,?,?,? )"
  for i in range(len(flightlegs.index)):
      string = flightlegs.iloc[i][0]
      data = string.split(";")
      data_tuple = (i,data[0],data[1],data[2],data[3],data[4],data[5],data[6])
      cursor.execute(insert_data, data_tuple)
      connection.commit()
```
