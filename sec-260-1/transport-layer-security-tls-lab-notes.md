---
description: 2/4/2026
---

# Transport Layer Security (TLS) Lab Notes

<figure><img src="../.gitbook/assets/unknown.png" alt=""><figcaption></figcaption></figure>

Use Wireshark to capture a HTTPS session - make sure to start the capture before you load the page.

* Identify the packets that make up the TLS negotiation and certificate exchange:\
  **The Packets that make up the TLS exchange the request, TCP ACK, the Reponse and the following ACK from the source PC to the server. The 4 in green are the certificate packets.**&#x20;

<figure><img src="../.gitbook/assets/{01FC129C-5E42-4688-BD17-D6E570CEDCBF}.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/{E16A29BD-A875-49A0-9FDD-17F1F93E6825}.png" alt=""><figcaption></figcaption></figure>

The Following packets demonstrate the client Hello, and the Server Hello that take place beforehand.&#x20;

* validity date:

<figure><img src="../.gitbook/assets/{87720723-2F82-435F-BF8E-D73D34775A1A}.png" alt=""><figcaption></figcaption></figure>

* certificate authority (issuer): **Under wireshark found the issuer name hash, found issuer name in the certificate.**

<figure><img src="../.gitbook/assets/{76074D7D-9582-4B45-88D1-F5E38A745BCA}.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/{18DC8804-98BB-48EE-B152-AC999EBC2EA8}.png" alt=""><figcaption></figcaption></figure>

* public key

<figure><img src="../.gitbook/assets/{F054F456-B9A3-4C54-8E88-D58B45CC4ACD}.png" alt=""><figcaption></figcaption></figure>

Below is a successful response from the OCSP Request.

<figure><img src="../.gitbook/assets/{D1E77B52-B7A6-4416-9923-98EE2F91D6C3}.png" alt=""><figcaption></figcaption></figure>

