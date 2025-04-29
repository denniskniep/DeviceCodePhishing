# DeviceCodePhishing

## TL;DR;
This is a novel technique that leverages the well-known Device Code phishing approach. 
It dynamically initiates the flow as soon as the victim opens the phishing link and instantly redirects them to the authentication page.
A headless browser automates this by directly entering the generated Device Code into the webpage behind the scenes. 
This defeats the 10-minute token validity limitation and eliminates the need for the victim to manually perform these steps, elevating the efficiency of the attack to a new level.  
What makes Device Code phishing especially dangerous is that no authentication method, not even FIDO, is able to protect against this type of attack. 
Additionally, the victim interacts with the original website they expect, making it impossible to detect the attack based on a suspicious URL.

## Demo
https://gist.github.com/user-attachments/assets/bf6d1c2d-7199-4394-824d-e6f57e8136a2

## Description 
DeviceCodePhishing is an advanced phishing tool, which leverages the Device Code Flow.
It can be used for phishing access-tokens, which in turn allows to bypass two-factor authentication protection, including accounts that exclusively use FIDO for authentication.

While other tools exist to automate device code phishing attacks, they often come with certain limitations, 
such as requiring the attacker to convince the victim to open the URL and enter the code within a strict 10-minute time frame.
The goal of this tool is to overcomes those limitations by automating the process with a headless browser, which initiates the attack 
as soon as the victim clicks on the phishing link.

This attack technique is even more dangerous than attacker-in-the-middle (AitM) proxies, because the
user **enters their credentials on the original webpage**, making it nearly impossible to detect the phishing attempt based on a suspicious URL.
Additionally, the victim might not need to authenticate interactively because a session is still active. 
Therefore, the victim has almost no time to realize that this is not legitimate. 
And not to forget that Device Code Flow is undermining FIDO's phishing resistance!

Currently, this tool is limited to targeting Microsoft Azure Entra users, but the underlying technique is not restricted to any specific vendor.

For more details, check out the blog post: [Phishing despite FIDO, leveraging a novel technique based on the Device Code Flow](https://denniskniep.github.io/posts/09-device-code-phishing)

## How it works
1. The attacker sends a URL to the victim
2. The victim opens that URL
3. When the URL is opened, a headless browser is started, performing the following automated steps:
   - Starts the Device Code Flow with `<tenant>` and `<clientId>`
   - Opens the device-code webpage and enters the corresponding user-code
   - The device-code webpage forwards to the URL for interactive authentication (By clicking on "Can't access your account" and immediately navigating back by clicking the cancel button, see [here](https://github.com/denniskniep/DeviceCodePhishing/blob/main/pkg/entra/devicecode.go#L101))
   - Returns the URL for interactive authentication as a redirect to the victim
4. The victim is redirected to the authentication URL
5. The victim completes the authentication
6. The attacker is authenticated

A demo video of the flow can be seen [here](#demo)  

## Install
Download appropriate binary from [Releases](https://github.com/denniskniep/DeviceCodePhishing/releases)
or install via go using following command:
```shell
go install github.com/denniskniep/DeviceCodePhishing@v1.0.1
```

## Start the phishing server

By default, it runs with tenant set to `common` and with the AuthenticationBroker ClientId `29d9ed98-a469-4536-ade2-f981bc1d605e`
```shell
DeviceCodePhishing server
```
Use the args if one want to define a specific tenant, a different clientId or a custom userAgent
```shell
DeviceCodePhishing server --tenant <tenantId> --client-id <clientId> --user-agent <userAgent> 
```
For further help on syntax or how to use arguments execute:
```shell
DeviceCodePhishing server --help
```

## Use
Open Url:
http://localhost:8080/lure


## Azure Entra ClientIds

| ClientId                             | Description                     |
|--------------------------------------|---------------------------------|
| 29d9ed98-a469-4536-ade2-f981bc1d605e | Microsoft Authentication Broker |
| 9ba1a5c7-f17a-4de9-a1f1-6178c8d51223 | Microsoft Intune Company Portal |

Hint: Use Microsoft Intune Company Portal for bypassing Intune compliant device Conditional Access Policy ([More Details](https://i.blackhat.com/EU-24/Presentations/EU-24-Chudo-Unveiling-the-Power-of-Intune-Leveraging-Intune-for-Breaking-Into-Your-Cloud-and-On-Premise.pdf))

## Next steps with obtained tokens
Once you have successfully obtained tokens, you can use them with other attack tools, such as:
* https://github.com/dafthack/GraphRunner
* https://github.com/f-bader/TokenTacticsV2?tab=readme-ov-file#azure-json-web-token-jwt-manipulation-toolset
* https://github.com/secureworks/family-of-client-ids-research


## Build it yourself 
```shell
go build main.go
```

```shell
./main server
```

## Run with Docker
```shell
docker run -p 8080:8080 ghcr.io/denniskniep/device-code-phishing:v1.0.1
```


## Build & Run it yourself with Docker
```shell
docker build . -t device-code-phishing
```

```shell
docker run -p 8080:8080 device-code-phishing
```

## Detection
The following three fields and their values identify DeviceCodePhishing events:

* originalTransferMethod: `deviceCodeFlow`
* resourceDisplayName: `Microsoft Graph`
* appDisplayName: `Microsoft Authentication Broker`

## Disclaimer
Provided as educational content only!
