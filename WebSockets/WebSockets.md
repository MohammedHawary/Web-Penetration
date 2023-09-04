# WebSockets

WebSockets, commonly used in modern web applications, establish long-lived connections over HTTP, enabling asynchronous communication in both directions. They serve various purposes, such as facilitating user actions and transmitting sensitive information. It's important to note that almost any web security vulnerability that applies to regular HTTP can also apply to WebSockets communications.

<img title="" src="https://github.com/MohammedHawary/Web-Penetration/assets/94152045/b600a22d-e678-4a0a-8838-4ec7fd1ecf4d" alt="websockets" data-align="inline">

Finding WebSockets security vulnerabilities generally involves manipulating them in ways that the application doesn't expect. 

You can use Burp Suite to: 

- Intercept and modify WebSocket messages.

- Replay and generate new WebSocket messages.

- Manipulate WebSocket connections.

#### Intercepting and modifying WebSocket messages

You can use Burp Proxy to intercept and modify WebSocket messages, as follows:

![Screenshot from 2023-08-15 11-48-25](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/d0500a98-6263-499c-8701-fc848a250695)

> **NOTE**
> 
> You can configure whether **client-to-server or server-to-client** messages are intercepted in Burp Proxy. Do this in the Settings dialog, in the WebSocket interception rules settings.

#### Replaying and generating new WebSocket messages

You can use Burp Repeater to intercept, modify, replay, and generate WebSocket messages on the fly.

![Screenshot from 2023-08-15 13-58-31](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/24621e9c-e84f-4332-b177-14fabf66d981)

#### Manipulating WebSocket connections

Manipulating the WebSocket handshake is sometimes required in different scenarios.

- It can enable you to reach more attack surface.

- Some attacks might cause your connection to drop so you need to establish a new one.

- Tokens or other data in the original handshake request might be stale and need updating.
  
  You can manipulate the WebSocket handshake using Burp Repeater: 

- Send a WebSocket message to Burp Repeater.

- In Burp Repeater, click the pencil icon next to the WebSocket URL to access a wizard for attaching to, **cloning**, or **reconnecting** to WebSocket **connections**. 

- If you choose to **clone** a **connected** WebSocket or **reconnect** to a disconnected WebSocket, then the wizard will show full details of the WebSocket handshake request, which you can edit as required before the handshake is performed. 

- Clicking "Connect" in Burp initiates the configured handshake and presents the result. If a new WebSocket connection is established successfully, you can utilize it in Burp Repeater to send new messages.

## WebSockets security vulnerabilities

In principle, practically any web security vulnerability might arise in relation to WebSockets: 

- User input transmitted to the server can be processed unsafely, resulting in vulnerabilities like SQL injection or XML external entity injection.

- Some blind vulnerabilities reached via WebSockets might only be detectable using **out-of-band** (**OAST**) techniques. 

- Transmitting attacker-controlled data through WebSockets to other application users can result in client-side vulnerabilities such as `XSS` or **other similar issues**.

### Manipulating WebSocket messages to exploit vulnerabilities

Most input-based vulnerabilities in WebSockets can be discovered and exploited by manipulating the contents of WebSocket messages.

For instance, consider a chat application that utilizes WebSockets to transmit chat messages between the browser and the server. When a user enters a chat message, it is sent to the server as a WebSocket message, typically in the following format:

```json
{"message":"Hello User"}
```

The contents of the message are transmitted (again via WebSockets) to another chat user, and rendered in the user's browser as follows: 

```html
<td>Hello User</td>
```

In this situation, provided no other input processing or defenses are in play, an attacker can perform a proof-of-concept XSS attack by submitting the following WebSocket message: 

```json
{"message":"<img src=1 onerror='alert(1)'>"}
```

#### EX: Manipulating WebSocket messages to exploit vulnerabilities

![Screenshot from 2023-08-15 14-16-51](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/24a6e7d0-eff6-4f1b-afca-d5c034ec1667)

There are a live chat, then let's send any message and capture this request

![Screenshot from 2023-08-15 11-48-33](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/379648d6-3e12-45f2-b72b-7c8e54658c5c)

I saw JSON data sent, then I opened websockets history to show if there are any websockets connectionsI saw JSON data sent, then I opened websockets history to show if there are any websockets connections![Screenshot from 2023-08-15 11-48-25](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/d0500a98-6263-499c-8701-fc848a250695)
Yes there are websockets connections then let's send the connection that's contain our message and send it to repeater and write XSS payload

![Screenshot from 2023-08-15 13-58-31](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/268f9163-934c-45c2-bc3c-1726e01c6f04)
![Screenshot from 2023-08-15 12-39-16](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/5d7ce5d5-9c1b-4122-b468-1b35d0bd42af)

![Screenshot from 2023-08-15 12-50-01](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/4c98bf5a-0fa7-4eb7-8ff6-8521979705d2)

### Manipulating the WebSocket handshake to exploit vulnerabilities

Certain WebSocket vulnerabilities can solely be discovered and taken advantage of through manipulation of the WebSocket handshake. These vulnerabilities typically revolve around design issues, including:

- Overreliance on HTTP headers for security determinations, like the `X-Forwarded-For` header.

- Weaknesses in session handling mechanisms, as WebSocket message processing context often aligns with the context of the handshake.

- Potential attack opportunities arising from custom application-specific HTTP headers.

#### EX: Manipulating the WebSocket handshake to exploit vulnerabilities

![Screenshot from 2023-08-15 14-16-51](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/24a6e7d0-eff6-4f1b-afca-d5c034ec1667)

There are a live chat, then let's send any message and capture this request

![Screenshot from 2023-08-15 13-58-31](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/268f9163-934c-45c2-bc3c-1726e01c6f04)

I saw JSON data sent, then I opened websockets history to show if there are any websockets connections I saw JSON data sent, then I opened websockets history to show if there are any websockets connections

![Screenshot from 2023-08-15 11-48-25](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/d0500a98-6263-499c-8701-fc848a250695)

Yes there are websockets connections then let's send the connection that's contain our message and send it to repeater and write XSS payload

![Screenshot from 2023-08-20 19-54-12](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/bc10c9fb-b28d-486c-8403-23aaf0a7f81c)

The server response with error message that the attack detected and my ip blocked

![Screenshot from 2023-08-20 19-42-59](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/df102c09-8bad-41f1-9204-83a6fcbde907)
Let's Bypass this using this header `X-Forwarder-For: 1` and connect
![Screenshot from 2023-08-20 19-55-02](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/7972d9bc-0a8c-4bc0-9589-6954c615e63b)
![Screenshot from 2023-08-20 19-55-14](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/9c87583e-0445-4d3d-a0a9-bf47f6b0ba31)
Let's Try another xss payload

![Screenshot from 2023-08-20 20-00-54](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/a67c13d4-7171-4131-9478-d94b571b8f6a)

### Using cross-site WebSockets to exploit vulnerabilities

Certain WebSocket security flaws occur when attackers initiate cross-domain WebSocket connections from a site they control. This is termed a cross-site WebSocket hijacking attack, which exploits a `CSRF` vulnerability during WebSocket handshake. This attack can lead to severe consequences, granting attackers the ability to execute privileged actions on the victim's behalf or obtain sensitive data within the victim's reach.

### What is cross-site WebSocket hijacking?

Cross-site WebSocket hijacking, or **cross-origin WebSocket hijacking**, exploits a WebSocket handshake's `CSRF` vulnerability. This arises when the handshake solely depends on HTTP cookies for session management, lacking `CSRF` tokens or unpredictable values.

Attackers craft a harmful webpage on their domain, establishing a cross-site WebSocket link with the vulnerable app. The app processes this within the victim's session.

This lets the attacker dispatch arbitrary messages to the server through the connection and access received server messages. Unlike typical `CSRF`, this attack offers two-way interaction with the compromised app.

### What is the impact of cross-site WebSocket hijacking?

 A successful cross-site WebSocket hijacking attack will often enable an attacker to: 

- **Execute unauthorized actions as the victim user**. Like standard `CSRF`, attackers can send arbitrary messages to the server-side app. If the app uses client-generated WebSocket messages for sensitive tasks, attackers can generate fitting cross-domain messages to initiate those actions.

- Access user-sensitive data. Unlike standard `CSRF`, cross-site WebSocket hijacking offers attackers bidirectional interaction via the hijacked WebSocket. If the app sends server-generated WebSocket messages containing sensitive data to the user, attackers can intercept these messages and seize the victim's information.

### Performing a cross-site WebSocket hijacking attack

As cross-site WebSocket hijacking is a WebSocket handshake's `CSRF` vulnerability, the initial step is assessing the app's handshakes to ascertain their `CSRF` protection.

In line with typical `CSRF` conditions, identifying a handshake that solely relies on HTTP cookies for session management (without tokens or unpredictable values in request parameters) is crucial.

Consider this WebSocket handshake request as an example of likely `CSRF` vulnerability, since the sole session token is in a cookie: 

```log
GET /chat HTTP/1.1
Host: normal-website.com
Sec-WebSocket-Version: 13
Sec-WebSocket-Key: wDqumtseNBJdhkihL6PW7w==
Connection: keep-alive, Upgrade
Cookie: session=KOsEJNuflw4Rd9BDNrVmvwBF9rEijeE2
Upgrade: websocket
```

> **NOTE**
> 
> The `Sec-WebSocket-Key` header **contains a random value to prevent errors from caching proxies**, and is **not used for authentication or session handling purposes**.

If the WebSocket handshake request is `CSRF` susceptible, an attacker's webpage can trigger a cross-site request to initiate a WebSocket on the vulnerable site. Subsequent steps hinge on the app's logic and WebSocket usage. Possibilities include:

- Dispatching WebSocket messages for unauthorized actions as the victim user.

- Sending WebSocket messages to access sensitive data.

- In some cases, awaiting incoming messages containing sensitive information.

#### EX: Cross-site WebSocket hijacking

![Screenshot from 2023-08-15 14-16-51](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/24a6e7d0-eff6-4f1b-afca-d5c034ec1667)

There are a live chat, then let's send any message and capture this request

![Screenshot from 2023-08-22 08-00-05](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/432baa77-719f-44f4-acf8-fa605cfeaf30)
The First Message sended to server is `READY` 

![Screenshot from 2023-08-22 08-00-14](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/450fb9ab-e3d5-4ff4-884d-44b6a00ffea6)
If you send it to Reapeter and resend it to server it's load all old messages

![Screenshot from 2023-08-22 08-02-28](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/3e11953a-0502-4f8c-b721-06e004b41762)

And there are no token or headers to prevent from csrf then let's ctreate csrf payload and send it to victim

![Screenshot from 2023-08-22 08-32-11](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/9befa415-8d3a-4246-9291-4f803c465c6f)

![Screenshot from 2023-08-22 08-31-56](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/d93fd81e-a836-448b-ad10-d24a34b1688f)![Screenshot from 2023-08-22 08-31-49](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/6082ec20-a2fb-4ea5-bf8d-528b1565a946)
After decode the query after message we get the calos password

![Screenshot from 2023-08-22 09-40-57](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/2afd9544-a6ec-412c-aa86-0b6952dc5ae5)

## How to secure a WebSocket connection

To minimize the risk of security vulnerabilities arising with WebSockets, use the following guidelines:

- Use the wss:// protocol (WebSockets over TLS).

- Hard code the URL of the WebSockets endpoint, and certainly don't incorporate user-controllable data into this URL.

- Protect the WebSocket handshake message against CSRF, to avoid cross-site WebSockets hijacking vulnerabilities.

- Treat data received via the WebSocket as untrusted in both directions. Handle data safely on both the server and client ends, to prevent input-based vulnerabilities such as SQL injection and cross-site scripting.


