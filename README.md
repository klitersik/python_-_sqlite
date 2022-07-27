# 1. Create a databse from csv file

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
# 2. Insert data into databse

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

# 3. Add new colums to database
  - flightDuration (duration of flight in minutes)
  - flightType (value 'D' = domestic) (value 'I' = international)
```python
  addColumn_flightDuration = "ALTER TABLE FlightLeg ADD COLUMN flightDuration int"
  cursor.execute(addColumn_flightDuration)
  connection.commit()
  
  addColumn_flightType = "ALTER TABLE FlightLeg ADD COLUMN flightType CHAR(1)"
  cursor.execute(addColumn_flightType)
  connection.commit()
```
# 4 Inserting data in new columns

```python
  update_data = """Update FlightLeg set flightDuration = ? where id = ?"""
  for i in range(len(flightlegs.index)):
      string = flightlegs.iloc[i][0]
      data = string.split(";")
      date_start = datetime.strptime(data[5], '%Y-%m-%d %H:%M:%S')
      date_stop = datetime.strptime(data[6], '%Y-%m-%d %H:%M:%S')
      date_diff = round((date_stop - date_start).total_seconds() / 60)
      time = (date_diff,i)
      cursor.execute(update_data, time)
  connection.commit()
```
```python
  update_data = """Update FlightLeg set flightType = ? where id = ?"""
  for i in range(len(flightlegs.index)):
      string = flightlegs.iloc[i][0]
      data = string.split(";")
      if data[2] == data[4]:
          code = "D"
      else:
          code = "I"
      codes = (code,i)
      cursor.execute(update_data, codes)
  connection.commit()
```
# 5 Implement the logic that answers the following questions: 
  1. Which aircraft made the most flights? 
  ```python
    cursor.execute('SELECT tailNumber FROM FlightLeg GROUP BY tailNumber ORDER BY COUNT(tailNumber) DESC LIMIT 1')
    for row in cursor.fetchall():
         print("Samolot który wykonał najwięcej lotów",row[0])
  ```
  2. Which aircraft flew the most minutes? 
  ```python
    cursor.execute('SELECT tailNumber FROM FlightLeg GROUP BY tailNumber ORDER BY SUM(flightDuration) DESC LIMIT 1')
    for row in cursor.fetchall():
         print("Samolot który przeleciał najwięcej minut",row[0])
  ```
  3. Which flight, split between domestic and international, was the shortest and which was the longest, and how many minutes did it take? 
  ```python
    cursor.execute('SELECT tailNumber,departureTimeUtc FROM FlightLeg WHERE flightType = "D" ORDER BY flightDuration DESC LIMIT 1')
    for row in cursor.fetchall():
         print("Najdłuższy lot krajowy",row)
    cursor.execute('SELECT tailNumber,departureTimeUtc FROM FlightLeg WHERE flightType = "D" ORDER BY flightDuration ASC LIMIT 1')
    for row in cursor.fetchall():
         print("Najkrótszy lot krajowy",row)
  ```
  ```python
    cursor.execute('SELECT tailNumber,departureTimeUtc FROM FlightLeg WHERE flightType = "I" ORDER BY flightDuration DESC LIMIT 1')
    for row in cursor.fetchall():
         print("Najdłuższy lot międzykrajowy",row)
    cursor.execute('SELECT tailNumber,departureTimeUtc FROM FlightLeg WHERE flightType = "I" ORDER BY flightDuration ASC LIMIT 1')
    for row in cursor.fetchall():
         print("Najkrótszy lot międzykrajowy",row)
  ```
