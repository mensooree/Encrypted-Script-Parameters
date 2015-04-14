# Encrypted Script Parameters

## What were we trying to solve?

A standard practice when running a script in a policy that needs to interact with an API is to pass the username and password for the service as parameters. All staff at JAMF Software have read access to the majority of objects in the production JSS (we refer to the main IT managed JSS as the "production JSS" since over half the company runs their own for testing and demos!). This presents the challenge of maintaining security around these API accounts in a very transparent environment.

Our Information Security department provided a solution to IT that was adapted into standard functions used for our Bash and Python scripts. Using the openssl binary, we encrypt the strings for these username and password parameters, generating unique salt and passphrase values that we hardcode into the uploaded script. Without access to both the policy and the script, the strings cannot be decrypted and used.

## What does it do?

There are two functions to this: the encryption function (called GenerateEncryptedString) that generates the encrypted string with the salt and passphrase values, and the decryption function (called DecryptString) which is embedded into a script that has the salt and passphrase values hardcoded to decrypt the string that is passed as a parameter. Because staff can access policy data in the JSS, but not the script data, they don't have all the pieces to decrypt the account credentials.

## How to use these scripts with policies

Here is an example of using these functions to generate the encrypted strings and then using the resulting values with a policy and script.

1) Use GenerateEncryptedString to encrypt the username and/or password values

```
~$ GenerateEncryptedString "Captain Hammer"
Encrypted String: U2FsdGVkX18/iRQ6O7Hr+pouW8TAl0RcrUByBUzavuY=
Salt: 3f89143a3bb1ebfa | Passphrase: 67a61589eb6fb3874052333b
```
2) Embed the DecryptString function with the "salt" and "passphrase" values into the script that will take the encrypted string above as a parameter

```
#!/bin/bash

function DecryptString() {
    echo "${1}" | /usr/bin/openssl enc -aes256 -d -a -A -S "${2}" -k "${3}"
}
username=$(DecryptString $4 '3f89143a3bb1ebfa' '67a61589eb6fb3874052333b') 
...
```

3) Add the script to a policy and paste the encrypted string into the corresponding parameter field

![Screenshot](/images/policy.png)

The Python functions are wrappers to openssl using the subprocess module and are used the same way:

```
>>> import subprocess
>>> GenerateEncryptedString("Doctor Horrible")
Encrypted String: U2FsdGVkX1/+1bcze4/E7R3wCfEru9qnHWG5da7p+bg=
Salt: fed5b7337b8fc4ed | Passphrase: bbf59ee05d84e8c8d5190b31
>>> DecryptString('U2FsdGVkX1/+1bcze4/E7R3wCfEru9qnHWG5da7p+bg=', 'fed5b7337b8fc4ed', 'bbf59ee05d84e8c8d5190b31')
'Doctor Horrible'
``` 

## License

```
Copyright (c) 2015, JAMF Software, LLC. All rights reserved.

Redistribution and use in source and binary forms, with or without modification, are
permitted provided that the following conditions are met:

    * Redistributions of source code must retain the above copyright notice, this
      list of conditions and the following disclaimer.
    * Redistributions in binary form must reproduce the above copyright notice, this
      list of conditions and the following disclaimer in the documentation and/or
      other materials provided with the distribution.
    * Neither the name of the JAMF Software, LLC nor the names of its contributors
      may be used to endorse or promote products derived from this software without
      specific prior written permission.
      
THIS SOFTWARE IS PROVIDED BY JAMF SOFTWARE, LLC "AS IS" AND ANY EXPRESS OR IMPLIED
WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY
AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL JAMF SOFTWARE,
LLC BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS
OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED
AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
(INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE,
EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
```