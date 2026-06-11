# vault-door-5

## Challenge Overview

The challenge provides **Java source code** with a hardcoded encoded string.
The goal is to reverse the encoding pipeline to recover the flag.

---

## Vulnerability

The `checkPassword` function encodes the input through **two layers** before comparing:

```java
public boolean checkPassword(String password) {
    String urlEncoded   = urlEncode(password.getBytes());    // Step 1: URL encode
    String base64Encoded = base64Encode(urlEncoded.getBytes()); // Step 2: Base64 encode
    String expected = "JTYzJTMwJTZlJTc2JTMzJTcyJTc0JTMxJTZlJTY3JTVm"
                    + "JTY2JTcyJTMwJTZkJTVmJTYyJTYxJTM1JTY1JTVmJTM2"
                    + "JTM0JTVmJTM0JTMxJTM4JTM1JTM1JTM1JTMxJTY1";
    return base64Encoded.equals(expected);
}
```

The encoding pipeline is:

```
password  →  URL encode (%xx)  →  Base64 encode  →  stored string
```

Since both operations are fully reversible, we simply run the pipeline **backwards**.

---

## Exploitation

### Step 1 — Identify the Reverse Pipeline

```
stored string  →  Base64 decode  →  URL decode  →  password
```

### Step 2 — Write the Decoder

```java
import java.util.Base64;
import java.net.URLDecoder;
import java.nio.charset.StandardCharsets;

public class VaultDoor5Solver {
    public static void main(String[] args) throws Exception {
        String expected =
                "JTYzJTMwJTZlJTc2JTMzJTcyJTc0JTMxJTZlJTY3JTVm"
              + "JTY2JTcyJTMwJTZkJTVmJTYyJTYxJTM1JTY1JTVmJTM2"
              + "JTM0JTVmJTM0JTMxJTM4JTM1JTM1JTM1JTMxJTY1";

        // Step 1: Base64 decode
        byte[] urlEncodedBytes = Base64.getDecoder().decode(expected);
        String urlEncoded = new String(urlEncodedBytes, StandardCharsets.UTF_8);

        // Step 2: URL decode
        String password = URLDecoder.decode(urlEncoded, "UTF-8");
        System.out.println("Password: " + password);
    }
}
```

Running the solver prints the flag.

🏁 Flag retrieved.

---

## Key Takeaway

> **Encoding is not encryption.** Base64 and URL encoding are reversible transformations with no secret key — anyone with the source code can decode the stored string in seconds. The comment in the source even jokes about Base64 being "eight times stronger" than base 8 or base 16, which highlights the misconception. Always use a proper cryptographic hash (e.g. bcrypt, Argon2) to store passwords.
```
