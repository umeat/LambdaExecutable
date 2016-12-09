# Executable
Python class to prepare and run 64 bit Linux executable's for use in AWS Lambda functions

This Python class is designed for use with AWS Lambda scripts, where the execution of a third party Linux 64 bit executable is required.

### Usage

```
from executable import Executable

exe = Executable('relative/path/to/exe')
result = exe.run('-sw1 -sw2')
print('OUT: {}\nERR: {}\nRET: {}'.format(
    exe.stdout, exe.stderr, exe.returncode))
    
# run() will return self.stdout
# result == exe.stdout
```

The following example is a full Lambda implementation, where the funciton is triggered by the creation of an S3 object - in this example the input file is assumed to be a RINEX file. The file is Hatanaka compressed then decompressed using the Linux 64 bit executables CRX2RNX, and RNX2CRX respectively. 

A zip file 'function package' is uploaded to Lambda with the following contents:

```
.
├───executable.py
├───executables
│   ├───CRX2RNX
│   └───RNX2CRX
└───lambda_function.py
```

The Lambda handler is "lambda_funciton.lambda_handler", and lambda_function.py contains the following code:

```
from __future__ import print_function

import json
import urllib
import boto3
import os

from executable import Executable

s3 = boto3.client('s3')

def lambda_handler(event, context):
  # Get the file object and bucket names from the event
  bucket = event['Records'][0]['s3']['bucket']['name']
  key = urllib.unquote_plus(event['Records'][0]['s3']['object']['key']).decode('utf8')

  # Get the submitted RINEX file object from S3 bucket
  try:
    response = s3.get_object(Bucket=bucket, Key=key)
  except Exception as err:
    print('Error: Failed to get {} from {} bucket'.format(key, bucket))
    raise err
	
  # Write S3 object contents to local file
  RNX = '/tmp/{}'.format(os.path.basename(key))
  with open(in_file, 'wb') as f:
  	f.write(response['Body'].read())
  
  # Run RNX2CRX
  rnx2crx = Executable('executables/RNX2CRX')
  CRX = rnx2crx.run('{} -'.format(RNX))
  
  # Run CRX2RNX
  crx2rnx = Executable('executables/CRX2RNX')
  new_RNX = crx2rnx.run('{} -'.format(CRX))
    
  return
```
