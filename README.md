# eWeLink mobile Application - Incorrect Access Control Vulnerability (CVE-2021-27941)
Unconstrained Web access to the device's private encryption key in the QR code pairing mode in the eWeLink mobile application (through 4.9.2 on Android and through 4.9.1 on iOS) allows a physically proximate attacker to eavesdrop on Wi-Fi credentials and other sensitive information by monitoring the Wi-Fi spectrum during a device pairing process.

# Pairing protocol and vulnerability details
From the user perspective, the QR code-based pairing process between the IoT device and the user’s WiFi, through the eWeLink application, acts as follows:
- the user boots the IoT device into paring mode.
- the user scans the QR-code attached to the IoT device with the eWeLink application.
- the application connects to a remote service to retrieve some device information, among which the device secret-key.
- the application connects to the device soft AP WiFi.
- the application uses the device secret key to encrypt a message with the target WiFi credential and send it to the device.
- the device uses the credential to access the home WiFi network,
- the eWeLink application adds the IoT device to the user’s device list (using a remote Cloud service).

The QR code label supplied with a supported device (a Sonoff Micro from ITEAD) encodes the following URL https://api.coolkit.cn:8080/add-ap-device?brand=ewelink\&ssid=ITEAD-device_id, where:
- the SSID parameter is the name of the WiFi network that is broadcasted by the device during the pairing process.
- the device_id is an alphanumeric string that represents the unique device id, which is used by the eWeLink application to query the Coolkit remote API and retrieve the device private encryption key (previously referenced as secret key).


During the first step, the application connects to eu-api.coolkit.cc:8080/api/user/fdbaseinfo?deviceid=device_id&other_parameters endpoint, from which it retrieves the device secret-key, along with other device information. According to the above-described flow, the service invoked by the application returns a key that is then used to encrypt (using the AES algorithm) the user’s WiFi credentials before sending them to the IoT device through the dedicated softAP WiFi network.
An encrypted JSON message is sent through an http post request to the open device http server.
The original payload follows the structure reported below:
```{"ssid" : "Test-Network", "password" : "Test-Password", "serverName" : "eu-disp.coolkit.cc", "port" : 443}```
where the "Test-Network" and "Test-Password" parameters are the network credentials we chose for our testbed configuration, and the eu-disp.coolkit.cc is the cloud platform serving the European region.


We ran different tests, and we noticed that during the pairing process everyone near the WiFi soft AP can retrieve the device_id parameter, since it is part of the device softAP WiFi SSID.
To retrieve a particular device secret-key you just need:
- a valid user account registered on the eWeLink platform. 
- the device_id parameter.

As a consequence, just by knowing a device id, the corresponding device secret key can be easily retrieved.
This enables a nearby malicious user to grab the user’s WiFi credentials by eavesdropping on the configuration message transmitted by the user’s smartphone to the IoT device. Note that, the device softAP Wifi is open and unprotected by default, so eavesdropping is possible.

After retrieving the secret key from the cloud services (together with the already known device_id parameter), the malicious user could decrypt the configuration message and extract the user’s WiFi credentials.
As a consequence, the customer's full home network is compromised.


# Tested Versions
 This vulnerability has been tested with the Android (v4.11.0 and earlier) and iOS (v4.9.1 and earlier) versions of the *eWeLink* mobile application.

# Disclosure Timeline
- Feb 25, 2021: Report submitted to Coolkit, the company behind eWeLink.
- Mar 02, 2021: No acknowledgement received, second report submitted to Coolkit.
- Mar 04, 2021: Received acknowledgement from Coolkit CTO, with a temporary mitigation before API re-design.
- Apr 08, 2021: Disclosing the vulnerability @ITASEC2021 Conference [1]
- May 03, 2021: Publishing the CVE-2021-27941 ref.

# References
[1] Daniele Granata, and Massimiliano Rak, and Giovanni Salzillo, and Umberto Barbato. "Security in IoT Pairing & Authentication protocols, a Threat Model and a Case Study Analysis", ITASEC 2021.

License
----

MIT
