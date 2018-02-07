## Raspberry Pi and Google Docs

This guide will cover the basic steps to record sensor data from a Raspberry Pi in a Google Spreadsheet using the Google Sheets API.

### What you need
- A Raspberry Pi running raspbian
- A sensor properly wired to the raspi via GPIO pins (this sample assumes a PIR Motion sensor is connected to GPIO4)
- A Google account
- A Google Spreadsheet for recording the pi data


### Basic steps
1. Follow steps 1 and 2 (3 is optional) from [this guide](https://developers.google.com/sheets/api/quickstart/python) to enable the Google Sheets API for your project.
2. Create a new directory for your project on your raspberry pi.
ex. `mkdir /home/pi/yourPiProject`
3. Move the `client_secret.json` from step 1 (step 1) to your project directory.
4. Use your method of choice to create a new python script in your project directory. 
ex. `post_pi_status.py`
5. Copy this sample code into your python file (this is a sample from [Google Sheets API docs](https://developers.google.com/sheets/api/quickstart/python) with minor adjustments for the sensor data and request payload).

```
import httplib2
import os

from apiclient import discovery
from oauth2client import client
from oauth2client import tools
from oauth2client.file import Storage

from datetime import datetime
from gpiozero import MotionSensor

pir = MotionSensor(4)

try:
    import argparse
    flags = argparse.ArgumentParser(parents=[tools.argparser]).parse_args()
except ImportError:
    flags = None

# If modifying these scopes, delete your previously saved credentials
# at ~/.credentials/sheets.googleapis.com-python-quickstart.json
SCOPES = 'https://www.googleapis.com/auth/spreadsheets'
CLIENT_SECRET_FILE = 'client_secret.json'
APPLICATION_NAME = 'raspi-sensor-sample'

def get_credentials():
    """Gets valid user credentials from storage.

    If nothing has been stored, or if the stored credentials are invalid,
    the OAuth2 flow is completed to obtain the new credentials.

    Returns:
        Credentials, the obtained credential.
    """
    home_dir = os.path.expanduser('~')
    credential_dir = os.path.join(home_dir, '.credentials')
    if not os.path.exists(credential_dir):
        os.makedirs(credential_dir)
    credential_path = os.path.join(credential_dir,
                                   'sheets.googleapis.raspi-sensor-sample.json')

    store = Storage(credential_path)
    credentials = store.get()
    if not credentials or credentials.invalid:
        flow = client.flow_from_clientsecrets(CLIENT_SECRET_FILE, SCOPES)
        flow.user_agent = APPLICATION_NAME
        if flags:
            credentials = tools.run_flow(flow, store, flags)
        else: # Needed only for compatibility with Python 2.6
            credentials = tools.run(flow, store)
        print('Storing credentials to ' + credential_path)
    return credentials


def main():
    credentials = get_credentials()
    http = credentials.authorize(httplib2.Http())
    discoveryUrl = ('https://sheets.googleapis.com/$discovery/rest?'
                    'version=v4')
    service = discovery.build('sheets', 'v4', http=http, discoveryServiceUrl=discoveryUrl)


    status = pir.is_active
    datestring = str(datetime.now())
    raspiId = "RasPi.0"
    
    values = [
        [status, datestring, raspiId]
    ]
        
    
    spreadsheetId = 'YOUR_SHEET_ID_HERE'
    rangeName = 'Sheet1!A1:C1'
    inputOption = 'RAW'
    body = { 'values' : values }

    request = service.spreadsheets().values().append(
        spreadsheetId=spreadsheetId, 
        range=rangeName,
        valueInputOption=inputOption,
        body=body
    )
    
    response = request.execute()
    print(response)


if __name__ == '__main__':
    main()

```

6. Get the ID of your Google Spreadsheet 
ex. `https://docs.google.com/spreadsheets/d/THIS_PART_IS_YOUR_SHEET_ID/edit`
7. Replace `YOUR_SHEET_ID_HERE` with your sheet id.
8. Make sure your are signed in to Google on your Raspberry Pi browser.
9. Save and run your python script.
ex. `python3 /home/pi/yourPiProject/post_pi_status.py`
note: The first run will trigger the permission request in the browser. 
10. After granting access, run the script again and verify a row has been added to your sheet. 


## Other info

### [Google Sheets API](https://developers.google.com/sheets/api/)

### [Sheets API Python Quickstart](https://developers.google.com/sheets/api/quickstart/python)

### [raspberry pi projects](https://projects.raspberrypi.org)

### [gpiozero](https://gpiozero.readthedocs.io/en/stable/)

### [gpiozero MotionSensor](https://gpiozero.readthedocs.io/en/stable/api_input.html#motion-sensor-d-sun-pir)


## Scheduling with `cron`
Once everything is working as desired you can schedule your script to run automatically.

`crontab -e`

- To run your script every hour add this line

` 0 * * * * python3 /home/pi/yourPiProject/post_pi_status.py`

- To edit your cron settings run 

`crontab -l`
- To run your script every hour minute change the line to

` * * * * * python3 /home/pi/yourPiProject/post_pi_status.py`

For more resources on `cron` and `crontab`:
https://www.raspberrypi.org/documentation/linux/usage/cron.md
https://tecadmin.net/crontab-in-linux-with-20-examples-of-cron-schedule/#
